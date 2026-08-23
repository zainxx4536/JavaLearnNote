## 版本边界

本文档按 ==Spring AI 2.0== 系列 API 整理。

```text
JDK 17
Spring Boot 4.1.0
Spring AI 2.0.0
MyBatis-Plus 3.5.17
DeepSeek Chat Model
OpenAI Starter，用来接入阿里云百炼 OpenAI-compatible API
SimpleVectorStore
PDF Document Reader
```

版本参考：[Spring AI 2.0.0 GA 官方发布说明](https://spring.io/blog/2026/06/12/spring-ai-2-0-0-GA-available-now/)。不同大模型厂商的模型名、计费方式和兼容能力可能变化，接入时还应核对对应平台的当前文档。

## 1. 模型部署

==大模型应用开发是通过**访问各厂商提供的大模型的对外暴露的API接口，实现与大模型的交互**。==

按照项目需求，可以访问不同功能的大模型，例如 ==ChatModel（对话大模型）==、==支持工具调用的大模型（ToolCalling）==、==支持多模态的大模型==等。

若想使大模型的对话符合设定的身份或语气，可以不断雕琢提示词，使大模型能给出最理想的回答，这个过程就叫做==**提示词工程**（**Prompt Engineering**）==。若想使大模型具备某一专业领域的知识，可使用 ==**RAG**== 技术来扩展或自定义大模型的知识库。

使用可访问的大模型有三种方式，三种方式各有优缺点：

1. ==使用开放的大模型 API==
2. 在云平台部署私有大模型
3. 在本地服务器部署私有大模型

### 开放大模型服务

通常发布大模型的官方，大多数的云平台都会提供开放的、公共的大模型服务，同时也提供 API 使用帮助文档，借助帮助文档可快速了解如何访问大模型 API。这些开放平台并不是免费，而是按照调用时消耗的 ==`token`== 来计费。一般必须要申请 ==API_KEY== 才能访问大模型。

### 本地部署大模型

很多云平台都提供了一键在本地部署大模型的功能，参考官方文档即可。

若想进行手动部署，可使用==帮助部署和运行大模型的工具————Ollama==，参考 Ollama 官方文档即可快速完成本地大模型部署。

## 2. 调用大模型

要去访问大模型 API 接口，必须掌握模型的 API 接口规范。目前==大多数大模型都遵循 OpenAI 的接口规范==，是基于 Http 协议的接口。因此请求路径、参数、返回值信息都是类似的，可能部分地方会有一些小的差别，具体需要查看大模型的官方 API 文档。

### 大模型接口规范

以 DeepSeek 官方给出的文档为例：

```Python
# Please install OpenAI SDK first: `pip3 install openai`

from openai import OpenAI

# 1.初始化OpenAI客户端，要指定两个参数：api_key、base_url
client = OpenAI(api_key="<DeepSeek API Key>", base_url="https://api.deepseek.com")

# 2.发送http请求到大模型，参数比较多
response = client.chat.completions.create(
    model="deepseek-chat", # 2.1.选择要访问的模型
    messages=[ # 2.2.发送给大模型的消息
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello"},
    ],
    stream=False # 2.3.是否以流式返回结果
)

print(response.choices[0].message.content)
```

### 接口说明
- ==请求方式：通常是 POST==，因为要传递 JSON 风格的参数
- ==请求路径（base_url）==：与平台有关
    - DeepSeek官方平台：https://api.deepseek.com
    - 阿里云百炼平台：https://dashscope.aliyuncs.com/compatible-mode/v1
- ==安全校验==：开放平台都需要提供 API_KEY 来校验权限，本地 ollama 则不需要
- ==请求参数==：参数很多，比较常见的有：
    - model：要访问的模型名称
    - messages：发送给大模型的消息，是一个数组
    - stream：true，代表响应结果流式返回；false，代表响应结果一次性返回，但需要等待
    - temperature：取值范围\[0:2)，代表大模型生成结果的随机性，越小随机性越低。DeepSeek-R1不支持

messages 是一个消息数组，其中的消息要包含两个属性：

- role：消息对应的角色
- content：消息内容

其中消息的内容，也被称为==**提示词**（**Prompt**）==，也就是发送给大模型的**指令**。

### 提示词角色

| 角色            | 描述                                                       | 示例               |
| ------------- | -------------------------------------------------------- | ---------------- |
| **system**    | 优先于 user 指令之前的指令，也就是给大模型设定角色和任务背景的系统指令                   | 你是一个乐于助人的编程助手。   |
| **user**      | 终端用户输入的指令（类似于你在 ChatGPT 聊天框输入的内容）                        | 写一首关于 Java 编程的诗。 |
| **assistant** | 由大模型生成的消息，可能是上一轮对话生成的结果。注意，用户可能与模型产生多轮对话，每轮对话模型都会生成不同结果。 |                  |

### 回话记忆

==**大模型本身是没有记忆的**。==因此在调用 API 接口与大模型对话时，每一次对话信息都不会保留，多次对话之间都是独立的。

要想使大模型有记忆，只需要每一次发送请求时，==把历史对话中每一轮的System prompt、User prompt、Assistant prompt 都封装到 Messages 数组中==，一起发送给大模型，这样大模型就会根据这些历史对话信息进一步回答，就像是拥有了记忆一样。

## 3. 大模型应用

==**大模型应用**是基于大模型的推理、分析、生成能力，**结合传统编程**能力，开发出的各种应用。==

传统应用开发和大模型有着各自擅长的领域：

- 传统编程：**确定性、规则化、高性能**，适合数学计算、流程控制等场景。
- AI大模型：**概率性、非结构化、泛化性**，适合语言、图像、创造性任务。

两者之间恰好是互补的关系，两者结合则能解决以前难以实现的一些问题：

- **混合系统（Hybrid AI）**
    - 用传统程序处理结构化逻辑（如支付校验），AI处理非结构化任务（如用户意图识别）。
    - **示例**：智能客服中，AI理解用户问题，传统代码调用数据库返回结果。
- **增强可解释性**
    - 结合规则引擎约束AI输出（如法律文档生成时强制符合条款格式）。
- **低代码/无代码平台**
    - 通过AI自动生成部分代码（如GitHub Copilot），降低传统开发门槛。

在传统应用开发中引入 AI 大模型，充分利用两者的优势，既能利用 AI 实现更加便捷的人机交互，更好的理解用户意图，又能利用传统编程保证安全性和准确性。

综上所述，大模型应用就是整合传统程序和大模型的能力和优势来开发的一种应用。

==模型本身只具备生成后文、基本推理的能力，并不具备会话记忆功能、联网功能等等，要想让大模型具有这些功能，需要通过额外的算法程序来实现的，也就是**基于大模型开发应用**。所以，**我们现在接触的 AI 对话产品其实都是基于大模型开发的应用，并不是大模型本身**。==

## 4. 大模型应用开发技术架构

基于大模型开发应用有多种方式：
- ==**提示词工程**（**Prompt Engineering**）==：通过不断雕琢提示词，使大模型能给出最理想的答案。
- ==**工具调用（Tool Calling）**==：大模型虽然可以理解自然语言，更清晰弄懂用户意图，但是无法直接操作数据库、执行严格的业务规则。这个时候我们可以把传统应用中的部分功能封装成一个个Tool，然后在提示词中描述用户的需求，并且描述清楚每个函数的作用，要求 AI 理解用户意图，判断什么时候需要调用哪个函数，并且将任务拆解为多个步骤（Agent），当 AI 执行到某一步，需要调用某个函数时，会返回要调用的函数名称、函数需要的参数信息。传统应用接收到这些数据以后，就可以调用本地函数。再把函数执行结果封装为提示词，再次发送给 AI。
- ==**RAG（Retrieval-Augmented Generation）**==：检索增强生成，简单来说就是把**信息检索技术**和**大模型**结合的方案。RAG 分为两个模块：
	- **检索模块（Retrieval）**：负责存储和检索拓展的知识库
	    - 文本拆分：将文本按照某种规则拆分为很多片段
	    - 文本嵌入（==Embedding==)：根据文本片段内容，将文本片段归类存储
	    - 文本检索：根据用户提问的问题，找出最相关的文本片段
	- **生成模块（Generation）**：
	    - 组合提示词：将检索到的片段与用户提问组织成提示词，形成更丰富的上下文信息
	    - 生成结果：调用生成式模型（例如DeepSeek）根据提示词，生成更准确的回答
- ==**模型微调（Fine-tuning）**==：在预训练大模型（比如DeepSeek、Qwen）的基础上，通过企业自己的数据做进一步的训练，使大模型的回答更符合自己企业的业务需求。这个过程通常需要在模型的参数上进行细微的修改，以达到最佳的性能表现。

## 5. SpringAI 调用大模型

### 引入依赖

SpringAI 完全适配了 SpringBoot 的自动装配功能，而且给不同的大模型提供了不同的 starter：

| 模型/平台 | starter                                                      |
| --------- | ------------------------------------------------------------ |
| Anthropic | \<dependency><br>    \<groupId>org.springframework.ai\</groupId><br>    \<artifactId>spring-ai-starter-model-anthropic\</artifactId><br>\</dependency> |
| DeepSeek  | \<dependency>  <br>    \<groupId>org.springframework.ai\</groupId><br>    \<artifactId>spring-ai-starter-model-deepseek\</artifactId><br>\</dependency> |
| Ollama    | \<dependency>   <br>   \<groupId>org.springframework.ai\</groupId><br>   \<artifactId>spring-ai-starter-model-ollama\</artifactId><br>\</dependency> |
| OpenAI    | \<dependency>    <br>    \<groupId>org.springframework.ai\</groupId><br>    \<artifactId>spring-ai-starter-model-openai\</artifactId><br>\</dependency> |

添加 spring-ai 的依赖管理项，将 spring-ai 的依赖统一管理起来：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 配置模型信息

在配置文件中配置模型的参数信息：

```yaml
spring:  
  ai:  
    deepseek:  
      base-url: https://api.deepseek.com  
      api-key: ${Deepseek_API_KEY}  
      chat:  
        model: deepseek-v4-flash  #对话模型，支持ToolCalling，不支持多模态、向量检索
```

### ChatClient

==**`ChatClient`** 是 **Spring AI 中用于调用大语言模型（LLM）的客户端**==。`ChatClient` 封装了与 AI 大模型对话的各种 API，同时支持同步式或响应式交互。主要负责发送 prompt，接收模型回答，配置系统提示词、用户消息，支持结构化、流式输出等。

==在使用之前，首先需要声明一个 `ChatClient` 并同时进行各种配置==：

```java
@Configuration  
public class CommonConfiguration {
	@Bean  
	public ChatClient chatClient(DeepSeekChatModel model) {  
	    return ChatClient  
	            .builder(model) //创建 ChatClient 工厂实例  
	            .defaultSystem("你是一个智能助手") //系统提示词 system propmt 
	            .build(); //构建 ChatClient 实例
	}
}
```

- `ChatClient.builder`：会得到一个`ChatClient.Builder`工厂对象，利用它可以自由选择模型、添加各种自定义配置
- `DeepSeekChatModel`：引入了 Deepseek 的 starter ，已经自动注入了对象。

==声明 `ChatClient` 后，在需要调用大模型的地方注入 `ChatClient` ，链式使用其中的方法==即可：

```java
@RequestMapping("/chat")  
public Flux<String> chat(String prompt) {  
    return chatClient.prompt()  //创建一个 Prompt 请求构建器，后面可配置.user(prompt)或.system(p)
            .user(prompt)  //设置用户发送给 AI 的消息
            .stream()  //采用流式调用
            .content();  //表示从模型返回的结果中，获取文本内容
}
```

==.stream()== 表示采用流式调用，即让模型生成一点返回一点。也可以换成 ==.call()== 方法进行同步调用，需要所有响应结果全部返回后才能返回给前端。

### Advisor

==Spring AI 通过 `ChatClient` 的 Advisor 调用链实现对请求、响应和上下文的增强。它与 Spring AOP 的概念相似，但这里使用的是 Spring AI 自己的 Advisor API，并不是给业务方法创建 AOP 代理。==

常用 Advisor：

- **SimpleLoggerAdvisor** ：日志记录的 Advisor，对外提供空参构造方法
- **MessageChatMemoryAdvisor** ：会话记忆的 Advisor，SpringAI 2.0 中不对外提供空参构造，常使用 builder(ChatMemory c) 来创建对象
- **QuestionAnswerAdvisor** ：实现 RAG 的 Advisor，SpringAI 2.0 中不对外提供空参构造，常使用 builder(VectorStore v) 来创建对象

### 日志 Advisor

给 `ChatClient` 添加日志Advisor：

```java
@Bean  
public ChatClient chatClient(DeepSeekChatModel model, ChatMemory chatMemory) {  
    return ChatClient  
            .builder(model) //创建 ChatClient 工厂实例  
            .defaultSystem("你是一个多模态的智能助手") //系统提示词  
            .defaultAdvisors(new SimpleLoggerAdvisor()) //配置日志 Advisor 
            .build(); //构建 ChatClient 实例  
}
```

==`SimpleLoggerAdvisor` 默认主要记录在 **DEBUG 级别**==，所以需要修改文件的日志级别：

```yaml
logging:
  level:
    org.springframework.ai: debug # AI对话的日志级别
    com.itzainxx.ai: debug # 本项目的日志级别
```

日志可能包含提示词、模型响应或工具参数。生产环境不要长期打开 DEBUG，也不要记录 API Key、个人信息和业务敏感数据。

### CORS 跨域问题

==**CORS（Cross-Origin Resource Sharing，跨域资源共享）**，本质上是**浏览器为了安全，限制一个网页的 JavaScript 去访问不同源的后端接口**。==服务器之间的 HTTP 请求不受浏览器 CORS 限制。所以 CORS 只需处理“浏览器前端访问不同源后端接口”的情况；后端调用 DeepSeek、百炼等模型 API 不需要 CORS。

可以==在 MVC 中配置全局 CORS==：

```java
@Configuration
public class MvcConfiguration implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") //对哪些路径生效
                .allowedOrigins("*") //允许哪些来源的请求，即允许那些域名进行跨域访问
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS") //允许哪些 HTTP 方法
                .allowedHeaders("*"); //允许请求中携带哪些请求头
    }
}
```

正式项目不建议 `allowedOrigins("*")`，应该改成明确的前端域名：

```java
.allowedOrigins("http://localhost:5173", "https://你的域名")
```

### 会话记忆功能

Spring AI 自带会话记忆抽象，可以保存历史消息并在后续调用时拼接。==`MessageChatMemoryAdvisor` 是 ChatClient Advisor 链中的一个组件，需要指定 `ChatMemory` 实例；将它添加到 `ChatClient` 后，它会按会话 ID 读写历史消息。==

声明 `ChatMemory` 示例：

```java
@Bean  
public ChatMemory chatMemory() {  
    return MessageWindowChatMemory.builder().build();  
}
```

`ChatMemory` 接口中提供了存储和读取会话记忆的方法：

- add(String conversationId, Message message)；
- add(String conversationId, List\<Message> messages）；
- get(String conversationId)；
- clear(String conversationId)；

可以看到，==所有的会话记忆都是与 `conversationId` 有关联的，也就是**会话ID**==，不同会话ID的记忆是分开管理的，这样才能保证不同对话、不同用户的回话保持独立。

添加回话记忆 `Advisor`：

```java
@Bean
public ChatClient chatClient(DeepSeekChatModel model, ChatMemory chatMemory) {
	return ChatClient
		.builder(model) //创建ChatClient工厂实例
        .defaultSystem("你是一个多模态的智能助手") //系统提示词
        .defaultAdvisors(new SimpleLoggerAdvisor()) //日志Advisor
        .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) //会话记忆Advisor
        .build(); //构建ChatClient实例
    }
```

==`ChatMemory` 负责按照 `conversationId` 保存和读取历史 `Message`，`MessageChatMemoryAdvisor` 则负责在 `ChatClient` 调用过程中自动获取对应会话的历史消息，并在模型调用后保存新的消息。==

==`MessageWindowChatMemory` 默认将会话记忆保存在内存中，因此后端服务重启后，之前的会话记忆会丢失。如果需要持久化保存，则需要使用 Redis、数据库等持久化方案。==

==每个会话通过 `conversationId` 进行区分，只要请求时正确传递不同的 `conversationId`，不同会话之间的记忆就是相互隔离的。==

### 会话历史功能

由于会话记忆是以 `conversationId` 来管理的，也就是**会话ID**。为了实现查询会话历史记录，必须记录所有的 ChatId，这样去查询会话历史，其实就是查询历史中有哪些 ChatId。最好将 ChatId 持久化保存到 Redis、MongoDB、MySQL 等数据库中，这里保存到 Map 集合中作为示例。

定义一个 `repository` 包，然后新建一个 `ChatHistoryRepository` 接口：

```Java
public interface ChatHistoryRepository {

    /**
     * 保存会话记录
     */
    void save(String type, String chatId);

    /**
     * 获取会话ID列表
     */
    List<String> getChatIds(String type);
}
```

定义一个实现类 `InMemoryChatHistoryRepository` ：

```Java
@Component
public class InMemoryChatHistoryRepository implements ChatHistoryRepository {

    private final Map<String, List<String>> chatHistory = new ConcurrentHashMap<>();

    @Override
    public void save(String type, String chatId) {
        List<String> chatIds = chatHistory.computeIfAbsent(type, k -> new CopyOnWriteArrayList<>());
        if (chatIds.contains(chatId)) {
            return;
        }
        chatIds.add(chatId);
    }

    @Override
    public List<String> getChatIds(String type) {
        return chatHistory.getOrDefault(type, List.of());
    }
}
```

这样保存 ChatId 和获取 ChatId 的方法就有了，接下来：

- 每次前端请求 AI 时都需要传递 ChatId 
- 每次处理请求时，将 ChatId 存储到 ChatRepository
- 每次发请求到 AI 大模型时，都传递自定义的 ChatId

控制层接口（也要配置 advisors）：

```Java
private final ChatHistoryRepository chatHistoryRepository;

@GetMapping(value = "/chat", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> chat(@RequestParam(defaultValue = "讲个笑话") String prompt, String chatId) {
    //保存前端传递的 ChatId 
	chatHistoryRepository.save("chat", chatId);
	return chatClient
    	.prompt(prompt)
        //将 ChatId 传递给 MessageChatMemoryAdvisor
    	.advisors(spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId))
    	.stream()
    	.content();
}
```

`spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId)` 把业务中的 `chatId` 和 Spring AI 的 `conversationId` 对应起来。这里传递 ChatId 给 `MessageChatMemoryAdvisor` 的方式是通过 **AdvisorContext**，也就是以 key-value 形式存入上下文，后面 `MessageChatMemoryAdvisor` 执行的过程中就可以拿到这个 ChatId 了。

到这里实际上就完成了根据不同的 ChatId 去获取不同的会话记忆了。

根据 ChatId 查询会话历史消息，也就是查询 **Message** 集合，这个集合就在 ChatMemory 中，可以通过 get 方法获得：

```java
@GetMapping("/{type}/{chatId}")
public List<MessageVO> getChatHistory(@PathVariable("type") String type, @PathVariable("chatId") String chatId) {
        List<Message> messages = chatMemory.get(chatId);
        if(messages == null) {
            return List.of();
        }
        return messages.stream().map(MessageVO::new).toList();
    }
```

这里的 MessageVO 是为了让响应数据符合接口要求：

```java
@NoArgsConstructor
@Data
public class MessageVO {
    private String role;
    private String content;

    public MessageVO(Message message) {
        this.role = switch (message.getMessageType()) {
            case USER -> "user";
            case ASSISTANT -> "assistant";
            case SYSTEM -> "system";
            default -> "";
        };
        this.content = message.getText();
    }
}
```

==完整流程的简述：==

用户第一次创建会话：

```
前端
 ↓
生成 chatId = 1001
 ↓
发送给后端
```

后端：

```
chatHistoryRepository.save(type, chatId);
```

于是：

```
ChatHistoryRepository
       ↓
记录 1001
```

然后：

```
chatClient
    .prompt(prompt)
    .advisors(spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId))
```

告诉 Advisor：

```
这次对话属于 1001
```

然后 `MessageChatMemoryAdvisor`：

```
1001
 ↓
ChatMemory
 ↓
读取 1001 的历史消息
 ↓
加入当前 Prompt
 ↓
调用大模型
 ↓
把新的 Message 保存回 1001
```

## 6. 提示词工程（Prompt Engineering）

==通过优化提示词，让大模型生成出尽可能理想的内容，这一过程就称为**提示词工程（Prompt Engineering）**。==

### 核心策略

1. **清晰明确的指令**
2. **使用分隔符标记输入内容**
3. **分步骤拆解复杂任务**
4. **提供示例（Few-shot Learning）** 
5. **指定输出格式**  
6. **给模型设定一个角色** 

### 减少模型“幻觉”的技巧

- **引用原文**：要求答案基于提供的数据（如“根据以下文章...”）。  
- **限制编造**：添加指令如“若不确定，不允许回答不相关信息“。

### 提示词攻击防范

- **提示注入（Prompt Injection）**
- **越狱攻击（Jailbreaking）**
- **数据泄露攻击（Data Extraction）**
- **模型欺骗（Model Manipulation）**
- **拒绝服务攻击（DoS via Prompt）**

## 7. Tool Calling

==首先把数据库的操作都定义成 Tool，也就是工具，然后在提示词中告诉大模型在什么情况下需要调用什么工具，即 Tool Calling。==

流程：

1. 提前把这些操作定义为 Tool
2. 然后将 Tool 的名称、作用、需要的参数等信息都封装为 Prompt 与用户的 Prompt 一起发送给大模型
3. 大模型在与用户交互的过程中，根据用户交流的内容判断是否需要调用 Tool
4. 如果需要则返回 Tool 名称、参数等信息
5. Java 解析结果，判断要调用哪个 Tool，代码执行 Tool，把结果再次封装到 Prompt 中发送给AI
6. AI 继续与用户交互，直到完成任务

SpringAI 提供了 ==@ToolParam==、==@Tool== 注解：

- ==@Tool：加在方法上，告诉 AI“这个方法可以被调用”。可加 description 参数给 AI 模型描述这个工具是干什么的。==
- ==@ToolParam：加在实体类参数或方法参数上，告诉 AI“这个实体类或方法的参数是什么”。可加 description 参数给 AI 说明这个参数应该填什么。==

定义 Tool 前需要定义一个实体类，封装可能传入的查询条件：

```Java
@Data
public class CourseQuery {
    @ToolParam(required = false, description = "课程类型：编程、设计、自媒体、其它")
    private String type;
    @ToolParam(required = false, description = "学历要求：0-无、1-初中、2-高中、3-大专、4-本科及本科以上")
    private Integer edu;
    @ToolParam(required = false, description = "排序方式")
    private List<Sort> sorts;

    @Data
    public static class Sort {
        @ToolParam(required = false, description = "排序字段: price或duration")
        private String field;
        @ToolParam(required = false, description = "是否是升序: true/false")
        private Boolean asc;
    }
}
```

Tool 工具书写示例：

```Java
@RequiredArgsConstructor
@Component
public class CourseTools {
    private final ICourseService courseService;
    private static final Set<String> ALLOWED_SORT_FIELDS = Set.of("price", "duration");

    @Tool(description = "根据条件查询课程")
    public List<Course> queryCourse(@ToolParam(required = false, description = "课程查询条件") CourseQuery query) {
        QueryChainWrapper<Course> wrapper = courseService.query();
        if (query == null) {
            return wrapper.list();
        }
        wrapper.eq(query.getType() != null, "type", query.getType())
                .le(query.getEdu() != null, "edu", query.getEdu());
        if(query.getSorts() != null) {
            for (CourseQuery.Sort sort : query.getSorts()) {
                if (sort != null && ALLOWED_SORT_FIELDS.contains(sort.getField())) {
                    wrapper.orderBy(true, Boolean.TRUE.equals(sort.getAsc()), sort.getField());
                }
            }
        }
        return wrapper.list();
    }
```

最后给 AI 设定一个 System prompt，告诉 AI 需要调用工具来实现复杂功能。

### 配置 ChatClient

```java
@Bean
public ChatClient serviceChatClient(DeepSeekChatModel model, ChatMemory chatMemory, CourseTools courseTools) {
	return ChatClient
		.builder(model) //创建ChatClient工厂实例
        .defaultSystem(SERVICE_SYSTEM_PROMPT) //系统提示词
        .defaultAdvisors(new SimpleLoggerAdvisor()) //配置日志 Advisor
        .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) //会话记忆 Advisor
        .defaultTools(courseTools) //工具调用 ToolsCalling
        .build(); //构建ChatClient实例
}
```

控制层接口无需设置：

```Java
@RequiredArgsConstructor
@RestController
@RequestMapping("/ai")
public class CustomerServiceController {
    private final ChatClient serviceChatClient;
    private final ChatHistoryRepository chatHistoryRepository;

    @GetMapping(value = "/service", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> service(String prompt, String chatId) {
        // 1.保存会话id
        chatHistoryRepository.save("service", chatId);
        // 2.请求模型
        return serviceChatClient.prompt()
                .user(prompt)
                .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, chatId))
                .stream()
                .content();
    }
}
```

## 8. RAG

==**RAG（Retrieval-Augmented Generation）**：检索增强生成，简单来说就是把**信息检索技术**和**大模型**结合的方案。RAG 分为两个模块：==

- ==**检索模块（Retrieval）**==：负责存储和检索拓展的知识库
  - 文本拆分：将文本按照某种规则拆分为很多片段
  - 文本嵌入（==Embedding==)：根据文本片段内容，将文本片段归类存储
  - 文本检索：根据用户提问的问题，找出最相关的文本片段
- ==**生成模块（Generation）**==：
  - 组合提示词：将检索到的片段与用户提问组织成提示词，形成更丰富的上下文信息
  - 生成结果：调用生成式模型（例如DeepSeek）根据提示词，生成更准确的回答

### RAG 原理

要解决大模型的知识限制问题的思路就是给大模型外挂一个**知识库**。不过，知识库不能简单的直接拼接在提示词中。因为通常知识库数据量都是非常大的，而大模型的上下文是有大小限制的。所以，我们需要==**从庞大的知识库中找到与用户问题相关的一小部分，组装成提示词**==，再发送给大模型。

而要从内容相似度来判断，就需要了解==**向量模型**==。

### 向量模型

==通常，两个向量的欧式距离或余弦距离越小，表示越相似；如果使用余弦相似度，则数值越大表示越相似。具体返回“距离”还是“相似度”，必须以所用向量库及 Spring AI 适配器的定义为准。==

要将文本转换为向量，就需要使用==向量模型（EmbeddingModel/VectorModel）==，一个好的向量模型，就是要**尽可能让文本含义相似的向量，在空间中距离更近**。

这里使用阿里云百炼平台的 `text-embedding-v4` 向量模型进行知识库检索。该接口提供 OpenAI-compatible 访问方式，可使用 Spring AI OpenAI 模块配置：

```yaml
spring:
  ai:
    openai:  
      base-url: https://dashscope.aliyuncs.com/compatible-mode/v1  
      api-key: ${OPENAI_API_KEY}  
      embedding:
        model: text-embedding-v4  # 文本嵌入模型
        dimensions: 1024
```

`dimensions: 1024` 指定生成的**向量维度为 1024**，即每段文本最终转换成一个 1024 维的向量。

Embedding 模型只支持将文本转换成向量，要进行数据比较和检索，需要使用**向量数据库**。

### 向量数据库

==向量数据库（VectorStore/VectorDatabase）的主要作用有两个：==

- ==存储向量数据==
- ==基于相似度检索数据==

SpringAI支持很多向量数据库，并且都进行了封装，可以用统一的 API 去访问：

- [Pinecone Vector Store](https://docs.spring.io/spring-ai/reference/api/vectordbs/pinecone.html) - [PineCone](https://www.pinecone.io/) vector store.
- [Qdrant Vector Store](https://docs.spring.io/spring-ai/reference/api/vectordbs/qdrant.html) - [Qdrant](https://www.qdrant.tech/) vector store.
- [Redis Vector Store](https://docs.spring.io/spring-ai/reference/api/vectordbs/redis.html) - The [Redis](https://redis.io/) vector store.
- ......

这些库都实现了统一的接口：`VectorStore`，因此操作方式基本相同。

SpringAI 提供了一个基于内存实现的 `SimpleVectorStore` 向量库，是一个专门用来测试、教学用的库，这里就使用 `SimpleVectorStore` 向量库，这个向量库就在 spring-ai-vector-store 中。

引入 VectorStore 依赖：

```xml
<dependency>
	<groupId>org.springframework.ai</groupId>
	<artifactId>spring-ai-vector-store</artifactId>
</dependency>
```

声明 VectorStore：

```Java
@Bean
public VectorStore vectorStore(OpenAiEmbeddingModel embeddingModel) {
	return SimpleVectorStore.builder(embeddingModel).build();
}
```

Vector Store中声明的部分方法：

- void add(List\<Document> documents); 保存文档到向量库，其中封装了文本的向量化
- void delete(List\<String> idList); 根据文档id删除文档
- List\<Document> similaritySearch(String query); 根据条件检索文档
- List\<Document> similaritySearch(SearchRequest request); 根据条件检索文档

==可以发现 `VectorStore` 操作向量化的基本单位是 `Document`，所以在使用时需要将知识库分割转换为一个个的 `Document`，然后利用 add() 方法将文本向量化后写入到 `VectorStore` 中。==

这里的 **`Document` 可以理解成 Spring AI 用来表示“一段知识内容”的标准对象**。它不是我们平时说的 Word/PDF 文件，而是一个**承载文本 + 元数据的对象**。`Document` 中封装了ID、Score、Text等，可通过 get 方法获取。

### 文件读取和转换

当知识库过大时，需要拆分成文档片段，然后再做向量化。而且 SpringAI 中 `VectorStore` 接收的是 `Document` 类型的文档，所以处理文档时还要将其转成 `Document` 格式。

在 SpringAI 中提供了各种文档读取的工具：

- ==`PagePdfDocumentReader` ：按页拆分，read() 方法可以读取 PDF 文档，并拆分为 Document，推荐使用==
- `ParagraphPdfDocumentReader` ：按 PDF 的目录拆分，不推荐，因为很多 PDF 不规范，没有章节标签

这里选择使用 `PagePdfDocumentReader`。引入依赖：

```XML
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pdf-document-reader</artifactId>
</dependency>
```

使用示例：

```java
Resource resource = new FileSystemResource("中二知识笔记.pdf");
// 1.创建PDF的读取器
PagePdfDocumentReader reader = new PagePdfDocumentReader(
    resource, // 文件源
    PdfDocumentReaderConfig.builder()
    	.withPageExtractedTextFormatter(ExtractedTextFormatter.defaults())
    	.withPagesPerDocument(1) // 每1页PDF作为一个Document
    	.build()
);
// 2.读取PDF文档，拆分为Document
List<Document> documents = reader.read();
// 3.写入向量库
vectorStore.add(documents);
// 4.搜索
SearchRequest request = SearchRequest.builder()
    .query("论语中教育的目的是什么") //搜索条件，也即prompt
    .topK(1) //取最相似的 1 个
    .similarityThreshold(0.6) //相似阈值
    .filterExpression("file_name == '中二知识笔记.pdf'") //文件过滤，即检索指定的文件
    .build();
List<Document> docs = vectorStore.similaritySearch(request);
if (docs == null || docs.isEmpty()) {
    System.out.println("没有搜索到任何内容");
    return;
}
for (Document doc : docs) {
    System.out.println(doc.getId());
    System.out.println(doc.getScore());
    System.out.println(doc.getText());
}
```

总结：RAG 要做的事情就是将知识库分割，然后利用向量模型做向量化，存入向量数据库，然后查询的时候去检索

**第一阶段（存储知识库）**：

- 将知识库内容切片，分为一个个片段 document（`PagePdfDocumentReader` 完成）
- 将每个片段利用向量模型向量化，向量化后的片段写入向量数据库（VectorStore 类的 add 方法完成）

**第二阶段（检索知识库）**：

- 每当用户询问 AI 时，将用户问题向量化
- 拿着问题向量去向量数据库检索最相关的片段

**第三阶段（对话大模型）**：

- 将检索到的片段、用户的问题一起拼接为提示词
- 发送提示词给大模型，得到响应

### 文件上传下载、向量化

这里构建的知识库是 PDF 形式的，由用户提交 PDF。所以需要先实现上传 PDF 的接口，在接口中实现下列功能：

- 校验文件格式是否为 PDF
- 保存文件信息
  - 保存文件（可以是 oss 或本地保存）
  - 保存==会话 ID 和文件路径的映射关系==（方便查询会话历史的时候再次读取文件）
- 文档拆分和向量化（文档太大，需要拆分为一个个片段，分别向量化）

另外，将来用户查询会话历史，还需要返回 PDF 文件给前端用于预览，所以需要实现下载 PDF 接口，包含功能：

- 读取文件
- 返回文件给前端

先定义两个方法：

```java
public interface FileRepository {
    /**
     * 保存文件,还要记录 chatId 与文件的映射关系
     */
    boolean save(String chatId, Resource resource);

    /**
     * 根据 chatId 获取文件
     */
    Resource getFile(String chatId);
}
```

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class LocalPdfFileRepository implements FileRepository {

    private final SimpleVectorStore vectorStore;
    private final Path storageDir = Paths.get("data", "pdf").toAbsolutePath().normalize();
    private final Path mappingFile = Paths.get("data", "chat-pdf.properties").toAbsolutePath().normalize();
    private final File vectorFile = Paths.get("data", "chat-pdf.json").toAbsolutePath().normalize().toFile();

    // 教学示例：生产环境应把映射关系存入数据库，并把文件存入对象存储。
    private final Properties chatFiles = new Properties();

    @Override
    public boolean save(String chatId, Resource resource) {
        if (chatId == null || chatId.isBlank()) {
            return false;
        }
        // 不直接使用客户端文件名，避免重名覆盖和路径穿越。
        String storedName = UUID.randomUUID() + ".pdf";
        Path target = storageDir.resolve(storedName).normalize();
        if (!target.startsWith(storageDir)) {
            return false;
        }
        try {
            Files.createDirectories(storageDir);
            Files.createDirectories(mappingFile.getParent());
            try (InputStream input = resource.getInputStream()) {
                Files.copy(input, target);
            }
            synchronized (chatFiles) {
                chatFiles.setProperty(chatId, storedName);
                persistMappings();
            }
            return true;
        } catch (IOException e) {
            log.error("Failed to save PDF resource.", e);
            return false;
        }
    }

    @Override
    public Resource getFile(String chatId) {
        String storedName;
        synchronized (chatFiles) {
            storedName = chatFiles.getProperty(chatId);
        }
        if (storedName == null) {
            return new FileSystemResource(storageDir.resolve("__missing__.pdf"));
        }
        Path target = storageDir.resolve(storedName).normalize();
        if (!target.startsWith(storageDir)) {
            return new FileSystemResource(storageDir.resolve("__invalid__.pdf"));
        }
        return new FileSystemResource(target);
    }

    /**
     * 项目启动时加载之前保存的数据
     */
    @PostConstruct
    private void init() {
        try {
            Files.createDirectories(storageDir);
            Files.createDirectories(mappingFile.getParent());
            if (Files.exists(mappingFile)) {
                try (Reader reader = Files.newBufferedReader(mappingFile, StandardCharsets.UTF_8)) {
                    chatFiles.load(reader);
                }
            }
            FileSystemResource vectorResource = new FileSystemResource(vectorFile);
            if (vectorResource.exists()) {
                vectorStore.load(vectorResource);
            }
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }

    private void persistMappings() throws IOException {
        try (Writer writer = Files.newBufferedWriter(
                mappingFile,
                StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.TRUNCATE_EXISTING)) {
            chatFiles.store(writer, "chatId to generated PDF filename");
        }
    }

    /** 项目正常关闭时再保存一次；写入向量后也应立即保存，不能只依赖此回调。 */
    @PreDestroy
    private void persistent() {
        try {
            synchronized (chatFiles) {
                persistMappings();
            }
            vectorStore.save(vectorFile);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }
}
```

由于使用基于内存的 `SimpleVectorStore`，重启会丢失内存中的向量数据，所以示例把映射关系和向量数据保存到磁盘。上传并写入向量后还要立即调用 `vectorStore.save(...)`；`@PreDestroy` 只能作为补充，进程异常退出时不一定执行。实际项目通常使用持久化向量库、数据库和对象存储，并补充事务补偿、权限校验、配额和文件清理策略。

文件上传和下载接口：

```java
@Slf4j
@RequiredArgsConstructor
@RestController
@RequestMapping("/ai/pdf")
public class PdfController {

    private final FileRepository fileRepository;

    private final SimpleVectorStore vectorStore;
    /**
     * 文件上传
     */
    @PostMapping(value = "/upload/{chatId}", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public Result uploadPdf(@PathVariable String chatId, @RequestParam("file") MultipartFile file) {
        try {
            // MIME 类型来自客户端，不能单独作为安全校验；至少再检查 PDF 文件头。
            if (file.isEmpty() || !Objects.equals(file.getContentType(), MediaType.APPLICATION_PDF_VALUE)
                    || !hasPdfHeader(file)) {
                return Result.fail("只能上传PDF文件！");
            }
            // 2.保存文件
            boolean success = fileRepository.save(chatId, file.getResource());
            if(! success) {
                return Result.fail("保存文件失败！");
            }
            // 使用服务器生成的安全文件名写入向量库，保证元数据和下载文件一致。
            this.writeToVectorStore(fileRepository.getFile(chatId));
            return Result.ok();
        } catch (Exception e) {
            log.error("Failed to upload PDF.", e);
            return Result.fail("上传文件失败！");
        }
    }

    private boolean hasPdfHeader(MultipartFile file) throws IOException {
        byte[] header = new byte[5];
        try (InputStream input = file.getInputStream()) {
            return input.read(header) == header.length
                    && Arrays.equals(header, "%PDF-".getBytes(StandardCharsets.US_ASCII));
        }
    }

    /**
     * 文件下载
     */
    @GetMapping("/file/{chatId}")
    public ResponseEntity<Resource> download(@PathVariable("chatId") String chatId) throws IOException {
        // 1.读取文件
        Resource resource = fileRepository.getFile(chatId);
        if (!resource.exists()) {
            return ResponseEntity.notFound().build();
        }
        String filename = Objects.requireNonNullElse(resource.getFilename(), "document.pdf");
        // 3.返回文件
        return ResponseEntity.ok()
                .contentType(MediaType.APPLICATION_PDF)
                .header(HttpHeaders.CONTENT_DISPOSITION,
                        ContentDisposition.attachment()
                                .filename(filename, StandardCharsets.UTF_8)
                                .build().toString())
                .body(resource);
    }

    private void writeToVectorStore(Resource resource) {
        // 1.创建PDF的读取器
        PagePdfDocumentReader reader = new PagePdfDocumentReader(
                resource, // 文件源
                PdfDocumentReaderConfig.builder()
                        .withPageExtractedTextFormatter(ExtractedTextFormatter.defaults())
                        .withPagesPerDocument(1) // 每1页PDF作为一个Document
                        .build()
        );
        // 2.读取PDF文档，拆分为Document
        List<Document> documents = reader.read();
        // 3.写入向量库
        vectorStore.add(documents);
        // SimpleVectorStore 是内存实现，添加后立即落盘；生产环境应改用持久化向量库。
        vectorStore.save(Paths.get("data", "chat-pdf.json").toFile());
    }
}
```

Spring Boot 默认单个文件上限通常为 1MB、单次 multipart 请求上限为 10MB。知识库文件可能超过默认值，因此按业务容量、网关限制和内存预算显式配置：

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 30MB
```

==默认情况下跨域请求的响应头是不暴露的，这样前端就拿不到下载的文件名，需要修改 CORS 配置，暴露响应头：==

```java
@Configuration
public class MvcConfiguration implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("Content-Disposition"); //暴露响应头
    }
}
```

### 配置 ChatClient

理论上来说，每次与 AI 对话的完整流程是这样的：

- 将用户上传的文件利用向量大模型做向量化后存储到 `VectorStore` 中
- 去 `VectorStore` 中检索相关的 `document` 片段，`getText()` 获取到文本信息
- 将获取的文本信息拼接上提示词，发送给大模型
- 解析响应结果

Spring AI 可以通过 `QuestionAnswerAdvisor` 把检索步骤接入 ChatClient Advisor 链；将 `VectorStore` 和检索参数配置给该 Advisor 即可：

```Java
@Bean
public ChatClient pdfChatClient(DeepSeekChatModel model, ChatMemory chatMemory, VectorStore vectorStore) {
    return ChatClient
        .builder(model) //创建ChatClient工厂实例
        .defaultSystem("根据从文件检索出来的内容回答问题") //系统提示词
        .defaultAdvisors(new SimpleLoggerAdvisor()) //配置日志 Advisor
        .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) //会话记忆 Advisor
        .defaultAdvisors(
        	QuestionAnswerAdvisor.builder(vectorStore) //配置向量库
        		.searchRequest(SearchRequest.builder() //设置检索条件
                       .topK(5) //最相关的前 5 个
                       .similarityThreshold(0.5) //相似阈值0.5，即相似度超0.5的信息才有效
                       .build())
        	.build())
        .build(); //构建ChatClient实例
}
```

控制层接口（也要配置 advisors）：

```java
@RestController
@Slf4j
@RequestMapping("/ai/pdf")
public class PdfController {
    private final FileRepository fileRepository;
    private final VectorStore vectorStore;
    private final ChatClient pdfChatClient;
    private final ChatHistoryRepository chatHistoryRepository;

    public PdfController(
            FileRepository fileRepository,
            VectorStore vectorStore,
            @Qualifier("pdfChatClient") ChatClient pdfChatClient,
            ChatHistoryRepository chatHistoryRepository) {
        this.fileRepository = fileRepository;
        this.vectorStore = vectorStore;
        this.pdfChatClient = pdfChatClient;
        this.chatHistoryRepository = chatHistoryRepository;
    }

    @GetMapping(value = "/chat", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chat(String prompt, String chatId) {
        //找到会话文件
        Resource file = fileRepository.getFile(chatId);
        //保存会话ID
        chatHistoryRepository.save("pdf", chatId);
        //请求模型
        String documentKey = Objects.requireNonNull(file.getFilename());
        if (!documentKey.matches("[0-9a-fA-F-]{36}\\.pdf")) {
            return Flux.error(new IllegalStateException("非法文档标识"));
        }
        return pdfChatClient.prompt()
            .user(prompt)
            .advisors(spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId))
            //给 QuestionAnswerAdvisor 设置一个过滤条件，让 RAG 检索时只搜索指定 PDF 文件对应的 Document
            .advisors(spec -> spec.param(
                    QuestionAnswerAdvisor.FILTER_EXPRESSION,
                    "file_name == '" + documentKey + "'"))
            .stream()
            .content();
    }
```

## 9. 多模态

==所谓多模态就是 AI 能够同时理解、处理或生成多种不同类型的信息，这里的“模态”可以理解成**信息的类型**。==

有些大模型支持多模态，有些则不支持。上面使用的 `deepseek-v4-flash` 对话模型支持 `ToolCalling`，但并不支持多模态，切换模型为阿里云的 `qwen-omni-turbo`。切换模型可以在 `yaml` 配置文件中直接更换模型配置，但这种改动是全局的，上面实现 `ChatModel` 和 `ToolCalling` 都是使用的 `deepseek-v4-flash`，不建议使用这种方式；另一种方式是==在创建 ChatClient 指定使用的模型==（需要在 `yaml` 配置文件配置 `base_url` 和 `API_KEY`）：

```java
@Bean
public ChatClient mulChatClient(OpenAiChatModel model, ChatMemory chatMemory) {
    return ChatClient
        .builder(model) //创建 ChatClient 工厂实例
        //指定支持多模态的模型
        .defaultOptions(ChatOptions.builder().model("qwen-omni-turbo"))
        .defaultSystem("你是一个多模态的智能助手") //系统提示词
        .defaultAdvisors(new SimpleLoggerAdvisor()) //配置日志 Advisor
        .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) //会话记忆 Advisor
        .build(); //构建 ChatClient 实例
}
```

修改控制层代码，实现无上传文件时使用 `deepseek-v4-flash` 模型进行对话，用户上传文件时使用 `qwen-omni-turbo` 模型进行支持多模态的对话：

```java
@PostMapping(
    value = "/chat",
    consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
    produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamChat(
    @RequestParam("prompt") String prompt,
    @RequestParam("chatId") String chatId,
    @RequestParam(value = "files", required = false) List<MultipartFile> files) {
    //保存会话ID
    chatHistoryRepository.save("chat", chatId);
    //请求大模型API
    if (files == null || files.isEmpty()) {
        //无附件，文本聊天
        return chatClient.prompt()
            .user(prompt)
            .advisors(spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId))
            .stream()
            .content();
    } else {
        //有附件，多模态聊天
        Set<String> allowedTypes = Set.of(
                MediaType.IMAGE_JPEG_VALUE,
                MediaType.IMAGE_PNG_VALUE,
                MediaType.APPLICATION_PDF_VALUE,
                "audio/mpeg");
        List<Media> medias = files.stream()
            .map(file -> {
                String contentType = file.getContentType();
                if (file.isEmpty() || !allowedTypes.contains(contentType)) {
                    throw new IllegalArgumentException("不支持的附件类型");
                }
                return new Media(MimeType.valueOf(contentType), file.getResource());
            })
            .toList();

        return mulChatClient.prompt()
            .user(spec -> spec.text(prompt).media(medias.toArray(Media[]::new)))
            .advisors(spec -> spec.param(ChatMemory.CONVERSATION_ID, chatId))
            .stream()
            .content();
    }
}
```

注意：==需要把附件转换成 Spring AI 能识别的 `Media` 类型，再和文字 Prompt 一起发送。图片、音频或 PDF 能否被处理，取决于具体模型和提供商接口是否支持对应 MIME 类型；`MultipartFile.getContentType()` 来自客户端，生产环境还必须校验文件头、大小和实际解码结果。==

```
new Media(文件类型,文件资源);创建一个 Media 对象，指定 MIME 类型和文件内容

file.getContentType();获取客户端声明的 MIME 类型（不是文件后缀，也不能作为唯一安全依据）

MimeType.valueOf(file.getContentType());就是将文件类型转换为 Spring 使用的 MimeType，主要用来告诉 AI 这个文件是什么类型的

file.getResource();获取文件内容

.user(spec -> spec.text(prompt).media(medias.toArray(Media[]::new)));发送文字+附件（多模态消息）medias.toArray(Media[]::new) 将 list 转为要求的数组
```

## 10.兼容性问题

Spring AI 的 `OpenAiChatModel` 按 OpenAI API 语义实现；阿里云百炼虽然提供 OpenAI-compatible 接口，但部分模型或能力仍可能存在差异，包括但不限于：

- `ToolCalling` 的 `stream` 模式，阿里云百炼返回的 `tool-arguments` 是不完整的，需要拼接，而 `OpenAI` 则是完整的，无需拼接。
- 音频识别中的数据格式，阿里云百炼的 `qwen-omni` 模型要求的参数格式为 `data:;base64,${media-data}`，而 `OpenAI` 是直接 `{media-data}`

兼容性结论具有版本边界，不能断言后续版本永远不会适配。遇到问题时应先用最小请求对照百炼当前文档与 Spring AI 当前版本，再选择：

- 使用与当前 Spring AI 版本兼容的 Spring AI Alibaba 适配层；
- 在 OpenAI-compatible 客户端外增加针对差异字段的适配，必要时再自定义 `ChatModel` 实现。

相关问题解决和功能优化参见：https://my.feishu.cn/wiki/space/7473055531994120220?ccm_open_type=lark_wiki_spaceLink&open_tab_from=wiki_home
