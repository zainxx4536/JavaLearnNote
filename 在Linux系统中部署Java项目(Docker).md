
本文记录一个典型 Java Web 项目在 Linux 上的部署方式。项目包含：

- 前端静态资源，通过 Nginx 对外访问
- 后端 Spring Boot `jar` 服务
- MySQL 数据库
- 可选 Docker / Docker Compose 容器化部署

推荐优先使用 **Docker Compose 部署**。直接部署适合理解环境配置和手动排错。

> 版本与安全边界：示例使用 Java 17、MySQL 8.4 和 Nginx 1.28 系列。镜像发布后应在测试环境验证并锁定 digest。密码、OSS 密钥不得写进 Dockerfile、Compose 文件、命令行历史或 Git；生产部署还要配置 HTTPS、备份、监控和最小权限。

## 一、部署前准备

### 1. 确认 Linux 服务器 IP

浏览器访问项目时，使用的是 Linux 服务器或虚拟机的 IP，不是容器 IP。

```bash
ip addr
```

例如网卡 `ens33` 显示：

```text
inet 192.168.88.130/24
```

则访问地址通常是：

```text
http://192.168.88.130
```

### 2. 常用端口

| 服务 | 容器内端口 | 宿主机端口示例 | 说明 |
| --- | ---: | ---: | --- |
| Nginx | 80 | 80 | 前端入口 |
| Spring Boot | 8080 | 不对公网发布 | 由 Nginx 通过容器网络访问；调试时可只绑定 `127.0.0.1:8080` |
| MySQL | 3306 | 不对公网发布 | 仅由后端通过容器网络访问；调试时可只绑定 `127.0.0.1:3307` |

如果使用防火墙，需要放行对应端口：

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone=public --list-ports
```

通常只开放 80/443。不要为了“方便连接”把数据库端口暴露给公网；管理访问应通过 VPN、SSH 隧道或受控安全组。

## 二、方式一：Docker Compose 部署【推荐】

### 1. 安装 Docker

以下命令适用于仍受支持的 RHEL/Rocky/AlmaLinux 类系统。CentOS Linux 7 已结束维护，不应作为新的生产部署基线；其他发行版请使用 Docker 官方对应安装说明。

卸载旧版本：

```bash
yum remove -y docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine \
  docker-selinux
```

安装 yum 工具：

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

配置 Docker CE yum 源：

```bash
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#更新 yum，建⽴缓存
yum makecache
```

安装 Docker：

```bash
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

启动并设置开机自启：

```bash
systemctl enable --now docker
docker version
docker compose version
```

### 2. 配置 Docker 镜像加速

```bash
mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://mirror.example.com"
  ]
}
EOF

systemctl daemon-reload
systemctl restart docker
docker info | grep -A 10 "Registry Mirrors"
```

`mirror.example.com` 是占位域名，不能直接使用。请替换为组织或云厂商当前分配给你的 HTTPS 加速地址；公共镜像地址可能变更或存在供应链风险。

### 3. 准备目录结构

在 Linux 中创建部署目录：

```bash
mkdir -p /usr/local/app/mysql/conf
mkdir -p /usr/local/app/mysql/data
mkdir -p /usr/local/app/mysql/init
mkdir -p /usr/local/app/nginx/conf
mkdir -p /usr/local/app/nginx/html
cd /usr/local/app
```

推荐目录结构：

```text
/usr/local/app
├── docker-compose.yml
├── Dockerfile
├── tlias.jar
├── mysql
│   ├── conf
│   │   └── my.cnf
│   ├── data
│   └── init
│       └── tlias.sql
└── nginx
    ├── conf
    │   └── nginx.conf
    └── html
        ├── index.html
        ├── favicon.ico
        └── assets
```

注意：前端打包后的 `dist` 目录内容要整体放入 `/usr/local/app/nginx/html`，不能只放 `index.html`，否则浏览器可能报：

```text
Failed to load module script: Expected a JavaScript-or-Wasm module script but the server responded with a MIME type of "text/html".
```

### 4. MySQL 配置文件

创建 `/usr/local/app/mysql/conf/my.cnf`：

```ini
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
default-time-zone=+08:00

[client]
default-character-set=utf8mb4
```

### 5. MySQL 初始化 SQL

把初始化 SQL 放到：

```text
/usr/local/app/mysql/init/tlias.sql
```

重要规则：

- `/docker-entrypoint-initdb.d` 中的 `.sql` 只会在 MySQL **第一次初始化空数据目录**时执行
- 如果 `/usr/local/app/mysql/data` 已经存在 MySQL 数据，后续再添加 SQL 文件不会自动执行
- 如果要重新执行初始化 SQL，需要先停止容器并处理旧数据目录；这会创建一个全新的数据库，必须先确认备份可恢复

重新初始化示例：

```bash
docker compose down
backup_dir="/usr/local/app/mysql/data-backup-$(date +%Y%m%d-%H%M%S)"
mv /usr/local/app/mysql/data "$backup_dir"
mkdir -p /usr/local/app/mysql/data
docker compose up -d
docker logs -f mysql
```

确认新实例和备份都正常后，再按保留策略处理 `data-backup-*`。不要直接复制执行 `rm -rf` 清空生产数据。

如果不想删除数据，可以手动导入：

```bash
docker exec -i mysql sh -c 'exec mysql -utlias_app -p"$MYSQL_PASSWORD" tlias' < /usr/local/app/mysql/init/tlias.sql
```

### 6. 后端 Dockerfile

创建 `/usr/local/app/Dockerfile`：

```dockerfile
# 使用维护中的 Java 17 JRE 基础镜像；上线时进一步锁定 digest。
FROM eclipse-temurin:17-jre

ENV TZ=Asia/Shanghai
ENV LANG=en_US.UTF-8

# 创建应用目录
RUN mkdir -p /tlias
WORKDIR /tlias
# 复制应用 JAR 文件到容器
COPY tlias.jar /tlias/tlias.jar

# 暴露端口
EXPOSE 8080

# 运行命令
ENTRYPOINT ["java", "-jar", "/tlias/tlias.jar"]
```

OSS、数据库等密钥在容器启动时通过环境变量或密钥管理服务注入，不能使用 Dockerfile 的 `ENV` 固化进镜像层。

后端配置中的 MySQL 地址建议写容器服务名：

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/tlias?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8
    username: ${MYSQL_USER:tlias_app}
    password: ${MYSQL_PASSWORD}
```

在同一个 Docker network 中，后端访问 MySQL 应使用：

```text
mysql:3306
```

不是：

```text
127.0.0.1:3306
```

### 7. Nginx 配置

创建 `/usr/local/app/nginx/conf/nginx.conf`：

```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name localhost;

        root /usr/share/nginx/html;
        index index.html;

        location /assets/ {
            try_files $uri =404;
        }

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api/ {
            proxy_pass http://tlias:8080/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

如果前端请求路径不是 `/api/`，需要按项目实际接口前缀调整 Nginx 配置或前端接口地址。

这里 `location /api/` 与 `proxy_pass http://tlias:8080/;` 都带尾部 `/`，因此转发时会去掉 `/api/` 前缀。例如 `/api/users` 会发给后端 `/users`。如果后端本身要求保留 `/api`，应改为 `proxy_pass http://tlias:8080;`。

### 8. docker-compose.yml

创建 `/usr/local/app/docker-compose.yml`：

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: mysql
    restart: unless-stopped
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:?set MYSQL_ROOT_PASSWORD in a protected .env file}
      MYSQL_DATABASE: tlias
      MYSQL_USER: tlias_app
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:?set MYSQL_PASSWORD in a protected .env file}
    volumes:
      - /usr/local/app/mysql/conf:/etc/mysql/conf.d
      - /usr/local/app/mysql/data:/var/lib/mysql
      - /usr/local/app/mysql/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -uroot -p\"$$MYSQL_ROOT_PASSWORD\" --silent"]
      interval: 10s
      timeout: 5s
      retries: 12
      start_period: 30s
    networks:
      - tlias-net

  tlias:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: tlias-server
    restart: unless-stopped
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/tlias?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8
      SPRING_DATASOURCE_USERNAME: tlias_app
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD:?set MYSQL_PASSWORD in a protected .env file}
      OSS_ACCESS_KEY_ID: ${OSS_ACCESS_KEY_ID:?set OSS_ACCESS_KEY_ID securely}
      OSS_ACCESS_KEY_SECRET: ${OSS_ACCESS_KEY_SECRET:?set OSS_ACCESS_KEY_SECRET securely}
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - tlias-net

  nginx:
    image: nginx:1.28-alpine
    container_name: nginx-tlias
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - /usr/local/app/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
      - /usr/local/app/nginx/html:/usr/share/nginx/html
    depends_on:
      - tlias
    networks:
      - tlias-net

networks:
  tlias-net:
    name: itzainxx
```

把 `.env` 放在 `/usr/local/app`、权限设为 `600`，并确保不提交到 Git：

```bash
install -m 600 /dev/null /usr/local/app/.env
# 使用编辑器写入 MYSQL_ROOT_PASSWORD、MYSQL_PASSWORD、OSS_ACCESS_KEY_ID、OSS_ACCESS_KEY_SECRET
```

此处通过 MySQL `healthcheck` 和 `depends_on.condition: service_healthy` 等待数据库可连接。它仍不能替代后端连接池的重试、数据库迁移工具和运行期故障恢复。

### 9. 启动项目

在 `/usr/local/app` 目录执行：

```bash
docker compose up -d
```

查看容器：

```bash
docker ps -a
```

查看日志：

```bash
docker logs -f mysql
docker logs -f tlias-server
docker logs -f nginx-tlias
```

访问：

```text
http://Linux服务器IP
```

例如：

```text
http://192.168.88.130
```

### 10. 停止项目

停止并删除容器、默认网络：

```bash
docker compose down
```

如果保留了 `/usr/local/app/mysql/data`，数据库数据不会丢。

如果要彻底重新初始化数据库：

```bash
docker compose down
backup_dir="/usr/local/app/mysql/data-backup-$(date +%Y%m%d-%H%M%S)"
mv /usr/local/app/mysql/data "$backup_dir"
mkdir -p /usr/local/app/mysql/data
docker compose up -d
```

## 三、方式二：不用 Docker Compose 部署

不用 Compose 时，需要手动创建 Docker 网络，并确保 MySQL、后端、Nginx 在同一个网络里。

### 1. 创建网络

```bash
docker network create itzainxx
```

如果网络已存在，会提示重复，可以忽略。

### 2. 启动 MySQL 容器

如果宿主机已经安装并运行 MySQL，且 Docker MySQL 要映射宿主机 `3306`，需要先停掉宿主机 MySQL。若映射为 `3307:3306`，一般不需要停宿主机 MySQL。

```bash
docker run -d \
  --name mysql \
  --network itzainxx \
  --env-file /usr/local/app/.env \
  -e MYSQL_DATABASE=tlias \
  -e MYSQL_USER=tlias_app \
  -e TZ=Asia/Shanghai \
  -v /usr/local/app/mysql/data:/var/lib/mysql \
  -v /usr/local/app/mysql/init:/docker-entrypoint-initdb.d \
  -v /usr/local/app/mysql/conf:/etc/mysql/conf.d \
  mysql:8.4
```

需要从宿主机临时调试数据库时，可额外添加 `-p 127.0.0.1:3307:3306`，不要绑定到所有网卡。

查看日志：

```bash
docker logs -f mysql
```

连接测试：

```bash
mysql -h127.0.0.1 -P3307 -uroot -p
```

### 3. 构建后端镜像

准备 `Dockerfile` 和 `tlias.jar` 后执行：

```bash
docker build -t tlias:1.0 .
```

启动后端：

```bash
docker run -d \
  --name tlias-server \
  --network itzainxx \
  --env-file /usr/local/app/.env \
  -e SPRING_DATASOURCE_URL='jdbc:mysql://mysql:3306/tlias?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8' \
  -e SPRING_DATASOURCE_USERNAME=tlias_app \
  tlias:1.0
```

查看日志：

```bash
docker logs -f tlias-server
```

### 4. 启动 Nginx 容器

```bash
docker run -d \
  --name nginx-tlias \
  --network itzainxx \
  -p 80:80 \
  -v /usr/local/app/nginx/html:/usr/share/nginx/html \
  -v /usr/local/app/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.28-alpine
```

查看日志：

```bash
docker logs -f nginx-tlias
```

## 四、方式三：直接部署到 Linux

直接部署适合学习环境。生产环境建议使用 systemd 管理服务，避免只靠手动命令启动。

### 1. 安装 JDK

上传 JDK：

```text
jdk-17.0.10_linux-x64_bin.tar.gz
```

解压：

```bash
tar -zxvf jdk-17.0.10_linux-x64_bin.tar.gz -C /usr/local/
```

配置全局环境变量，推荐使用 `/etc/profile.d/java.sh`：

```bash
tee /etc/profile.d/java.sh <<-'EOF'
export JAVA_HOME=/usr/local/jdk-17.0.10
export PATH=$JAVA_HOME/bin:$PATH
EOF

source /etc/profile
java -version
javac -version
```

### 2. 安装 MySQL：Generic Linux 方式

上传：

```text
mysql-8.4.6-linux-glibc2.28-x86_64.tar.xz
```

解压并移动：

```bash
tar -xJvf mysql-8.4.6-linux-glibc2.28-x86_64.tar.xz
mv mysql-8.4.6-linux-glibc2.28-x86_64 /usr/local/mysql
```

创建用户和数据目录：

```bash
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
mkdir -p /usr/local/mysql/data
chown -R mysql:mysql /usr/local/mysql
```

初始化：

```bash
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

初始化输出中会出现 root 临时密码，需要保存。

配置环境变量：

```bash
tee /etc/profile.d/mysql.sh <<-'EOF'
export MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
EOF

source /etc/profile
mysql --version
```

注册服务：

```bash
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
chkconfig --add mysql
systemctl start mysql
systemctl enable mysql
systemctl status mysql
```

如果使用 RPM 安装的 MySQL Community，服务名通常是 `mysqld`：

```bash
systemctl start mysqld
systemctl enable mysqld
systemctl status mysqld
```

登录并修改密码：

```bash
mysql -uroot -p
```

进入 MySQL 后：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Replace-With-A-Unique-Strong-Password!';
```

不要为了通过安装演示而降低全局密码策略。为应用创建最小权限账户；下面的主机模式仅为内网示例，应按真实后端来源进一步收紧：

```sql
CREATE DATABASE IF NOT EXISTS tlias CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'tlias_app'@'192.168.88.%' IDENTIFIED BY 'Replace-With-An-App-Only-Strong-Password!';
GRANT SELECT, INSERT, UPDATE, DELETE ON tlias.* TO 'tlias_app'@'192.168.88.%';
SELECT user, host FROM mysql.user;
```

开放端口：

```bash
ss -lntp | grep 3306
# 如确需跨主机访问，只允许后端所在内网网段/安全组；不要直接对公网开放 3306。
```

### 3. 安装 Nginx

安装依赖：

```bash
yum install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel gcc-c++
```

解压源码：

```bash
tar -zxvf nginx-1.28.0.tar.gz
cd nginx-1.28.0
```

编译安装：

```bash
./configure --prefix=/usr/local/nginx
make
make install
```

启动和停止：

```bash
cd /usr/local/nginx
sbin/nginx
sbin/nginx -s stop
sbin/nginx -s reload
sbin/nginx -t
```

如果启动时报：

```text
bind() to 0.0.0.0:80 failed (98: Address already in use)
```

说明 80 端口已被占用，检查：

```bash
ss -lntp | grep ':80'
```

### 4. 部署前端

将前端打包后的 `dist` 目录内容复制到：

```text
/usr/local/nginx/html
```

注意是复制 `dist` 里面的内容，不是复制 `dist` 目录本身。

修改 Nginx 配置后测试并重载：

```bash
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx -s reload
```

### 5. 部署后端

上传后端 `jar` 包。前台启动只用于快速验证：

```bash
java -jar tlias-web-management-0.0.1-SNAPSHOT.jar
```

临时后台调试可以使用 `nohup`，但它不负责开机启动、重启策略和统一日志：

```bash
nohup java -jar tlias-web-management-0.0.1-SNAPSHOT.jar > tlias.log 2>&1 &
```

查看进程：

```bash
ps -ef | grep tlias
```

停止进程：

```bash
kill 进程ID
```

如果无法正常停止，再使用：

```bash
kill -9 进程ID
```

长期运行使用 systemd。先创建仅 root 可读的环境文件：

```bash
sudo install -m 600 /dev/null /etc/tlias.env
sudoedit /etc/tlias.env
```

文件内容使用 `KEY=value`，不要写 `export`，也不要提交到 Git：

```text
OSS_ACCESS_KEY_ID=替换为真实值
OSS_ACCESS_KEY_SECRET=替换为真实值
MYSQL_PASSWORD=替换为应用账号密码
```

创建 `/etc/systemd/system/tlias.service`：

```ini
[Unit]
Description=Tlias Spring Boot Service
After=network-online.target mysql.service
Wants=network-online.target

[Service]
Type=simple
User=tlias
Group=tlias
WorkingDirectory=/opt/tlias
EnvironmentFile=/etc/tlias.env
ExecStart=/usr/local/jdk-17.0.10/bin/java -jar /opt/tlias/tlias-web-management-0.0.1-SNAPSHOT.jar
Restart=on-failure
RestartSec=5
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

先创建受限用户、准备目录并赋权，再启动：

```bash
sudo useradd --system --home /opt/tlias --shell /usr/sbin/nologin tlias
sudo install -d -o tlias -g tlias /opt/tlias
sudo chown tlias:tlias /opt/tlias/tlias-web-management-0.0.1-SNAPSHOT.jar
sudo systemctl daemon-reload
sudo systemctl enable --now tlias
sudo systemctl status tlias
journalctl -u tlias -f
```

## 五、常见问题排查

### 1. nginx 容器运行了，但浏览器访问不到

先确认端口映射：

```bash
docker ps -a
```

应看到：

```text
0.0.0.0:80->80/tcp
```

本机测试：

```bash
curl http://127.0.0.1
```

查看 Linux IP：

```bash
ip addr
```

浏览器访问：

```text
http://Linux服务器IP
```

### 2. 浏览器报 MIME type text/html

常见原因是 `index.html` 引用的 JS 文件不存在，Nginx 返回了 `index.html`。

检查静态资源：

```bash
docker exec -it nginx-tlias ls -l /usr/share/nginx/html
docker exec -it nginx-tlias ls -l /usr/share/nginx/html/assets
```

确认 `assets` 目录和 `.js`、`.css` 文件完整存在。

### 3. 后端连接 MySQL 报 Connection refused

检查 MySQL 容器：

```bash
docker ps -a
docker logs -f mysql
```

检查后端配置：

```text
jdbc:mysql://mysql:3306/数据库名
```

如果后端运行在宿主机，不在 Docker 网络内，则连接地址应使用：

```text
jdbc:mysql://127.0.0.1:3307/数据库名
```

### 4. Docker MySQL 的 init SQL 没执行

检查数据目录是否已经初始化：

```bash
ls -la /usr/local/app/mysql/data
```

如果里面已有 `mysql`、`sys`、`performance_schema`、`ibdata1` 等内容，说明已经初始化过，`init` 目录里的 SQL 不会再次自动执行。

手动导入：

```bash
docker exec -i mysql sh -c 'exec mysql -utlias_app -p"$MYSQL_PASSWORD" tlias' < /usr/local/app/mysql/init/tlias.sql
```

### 5. Docker volume 删除失败

如果提示：

```text
volume is in use
```

先找到占用容器：

```bash
docker ps -a --filter volume=卷名
```

删除容器后再删除卷：

```bash
docker rm -f 容器ID
docker volume rm 卷名
```

### 6. systemctl stop mysql 找不到服务

MySQL Community RPM 的服务名通常是：

```bash
mysqld
```

使用：

```bash
systemctl status mysqld
systemctl stop mysqld
systemctl start mysqld
```

## 六、推荐部署顺序

1. 安装 Docker 和 Docker Compose
2. 准备 `/usr/local/app` 目录结构
3. 放入 MySQL 初始化 SQL
4. 放入前端 `dist` 的完整内容
5. 准备后端 `jar` 和 `Dockerfile`
6. 编写 `docker-compose.yml`
7. 执行 `docker compose up -d`
8. 查看 `mysql`、`tlias-server`、`nginx-tlias` 日志
9. 浏览器访问 `http://Linux服务器IP`
