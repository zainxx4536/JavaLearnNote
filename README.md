# Java 后端学习笔记

这是一个面向 **Java 后端 / Java 全栈方向** 的个人学习知识库，内容主要整理自黑马程序员相关课程的学习过程，并结合个人理解、代码示例与实践经验持续补充。

仓库使用两种形式组织内容：

- **XMind 思维导图**：梳理知识体系、核心概念及技术之间的关系。
- **Markdown 专题笔记**：记录适合连续阅读的知识点、配置说明、命令和代码示例。

当前仓库包含 **9 份 XMind 笔记、5 份 Markdown 笔记和 2 张命令速查图**，覆盖 Java 基础、数据库、主流后端框架、工程化、部署、微服务、前端基础与大模型应用开发。

## 笔记导航

### 1. Java 基础与工程工具

| 笔记 | 形式 | 主要内容 |
| --- | --- | --- |
| [Java基础.xmind](Java基础.xmind) | XMind | 基础语法、数组与方法、面向对象、常用 API、集合、基础算法、Lambda、Stream、异常、File、IO、多线程、网络编程、反射和动态代理 |
| [MySQL_note.md](MySQL_note.md) | Markdown | SQL、函数、约束、多表查询、事务、MySQL 体系结构、存储引擎、索引、性能分析与 SQL 优化 |
| [Maven.xmind](Maven.xmind) | XMind | Maven 配置、基础与高级用法、常见问题以及 `settings.xml` 配置补充 |
| [POM文件帮助文档.md](POM文件帮助文档.md) | Markdown | `pom.xml` 总体配置，以及 `settings.xml` 中仓库、镜像、代理、服务器和 Profile 等配置项 |
| [Git.xmind](Git.xmind) | XMind | 版本控制基础、初始化与配置、本地仓库命令、分支管理和远程仓库 |
| [Linux_note.md](Linux_note.md) | Markdown | 目录结构、基础命令、用户与权限、软件安装、网络、进程、系统状态、环境变量及压缩解压 |

### 2. Java Web 与数据访问

| 笔记 | 形式 | 主要内容 |
| --- | --- | --- |
| [Java后端.xmind](Java后端.xmind) | XMind | HTTP、REST、Spring MVC、Spring Boot、IOC/DI、事务、MyBatis、文件上传、登录认证、AOP、缓存、任务调度、WebSocket、POI 和自定义 Starter |
| [MybatisPlus.xmind](MybatisPlus.xmind) | XMind | `BaseMapper`、常用注解、条件构造器、自定义 SQL、`IService`、逻辑删除、枚举与 JSON 类型处理、分页插件 |
| [Redis.xmind](Redis.xmind) | XMind | Redis 数据结构与命令、Java 客户端、登录状态、缓存、秒杀、点赞与关注、GEO、BitMap、HyperLogLog、高级知识和原理分析 |

### 3. 容器化、部署与微服务

| 笔记 | 形式 | 主要内容 |
| --- | --- | --- |
| [Docker.xmind](Docker.xmind) | XMind | 镜像、容器、仓库、常用命令、数据卷、自定义镜像、网络和 Docker Compose |
| [在Linux系统中部署Java项目(Docker).md](<在Linux系统中部署Java项目(Docker).md>) | Markdown | 使用 Docker Compose、普通 Docker 或直接部署三种方式，在 Linux 上部署 Java 后端、MySQL 与 Nginx，并整理常见故障排查方法 |
| [SpringCloud.xmind](SpringCloud.xmind) | XMind | 单体与微服务架构、服务拆分、注册中心、OpenFeign、Gateway、配置管理、服务保护、分布式事务、RabbitMQ 与 Spring AMQP |

### 4. 前端与 AI 应用扩展

| 笔记 | 形式 | 主要内容 |
| --- | --- | --- |
| [前端.xmind](前端.xmind) | XMind | HTML、CSS、JavaScript、Ajax、Axios、Vue、工程化、Vue Router 和 Element Plus |
| [SpringAI.md](SpringAI.md) | Markdown | 模型部署与调用、Spring AI、ChatClient、Advisor、会话记忆、提示词工程、Tool Calling、RAG、多模态和兼容性问题 |

### 5. 命令速查图

- [Git 命令速查](image/Git命令.png)
- [Docker 命令速查](image/Docker命令.png)

## 建议学习路线

```text
Java 基础
   ↓
MySQL + Maven + Git + Linux
   ↓
Java Web / Spring Boot / MyBatis
   ↓
MyBatis-Plus + Redis
   ↓
Docker + Linux 项目部署
   ↓
Spring Cloud 微服务
```

在完成后端主线后，可以根据目标选择扩展方向：

- **Java 全栈方向**：学习前端基础、Vue、Vue Router 和 Element Plus，建立完整的前后端协作能力。
- **Java + AI 应用方向**：学习模型 API、提示词工程、Tool Calling、RAG 和 Spring AI，重点提升大模型应用集成能力。

## 阅读方式

### Markdown 文件

Markdown 文件可以直接在 GitHub、IDEA、VS Code 或 Obsidian 中阅读。若需要复制命令或代码，建议使用具备语法高亮功能的编辑器打开。

### XMind 文件

`.xmind` 文件需要使用 [XMind](https://xmind.app/) 打开。代码托管平台通常无法直接预览完整思维导图，请先下载对应文件，再使用 XMind 桌面端查看。部分主题标题使用 `*` 标记重点内容。

## 使用说明

1. 根据“建议学习路线”选择当前阶段的笔记。
2. 先使用 XMind 建立整体知识框架，再通过 Markdown 专题补充细节、命令和代码。
3. 对重要知识点进行代码复现，并结合实际项目验证，而不是只记忆结论。
4. Spring Cloud、Spring AI、模型接口和第三方依赖更新较快，实际开发前应结合项目版本查阅官方文档。

## 说明

- 本仓库是个人学习过程中的知识整理，不是黑马程序员、XMind 或相关技术项目的官方文档。
- 笔记中的配置、端口、依赖版本和示例代码可能针对特定学习环境；生产环境使用前需要重新进行安全、兼容性和性能检查。
- 仓库会随学习进度持续更新，部分内容仍可能存在遗漏或理解偏差，欢迎通过 Issue 或 Pull Request 交流和纠正。

## 许可协议

本仓库采用 [Creative Commons Attribution 4.0 International（CC BY 4.0）](LICENSE) 许可协议。转载或改编时请保留原作者署名并注明来源。
