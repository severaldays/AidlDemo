# 汽车座舱AI智能体系统 - 技术选型文档

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档类型 | 技术选型 |
| 版本 | V2.0 |
| 创建日期 | 2026-01-14 |
| 更新日期 | 2026-01-14 |

---

## 1. 概述

本文档详细说明汽车座舱AI智能体系统的技术选型方案，包括各模块的技术选择、对比分析和最终决策依据。

### 1.1 选型原则

| 原则 | 说明 |
|------|------|
| **车规级适配** | 优先选择支持车规级芯片(8295)的技术方案 |
| **性能优先** | 满足实时交互的低延迟要求 |
| **稳定可靠** | 选择成熟稳定的技术方案 |
| **端云协同** | 支持离线能力，云端增强 |
| **可扩展性** | 便于后续功能扩展和升级 |

### 1.2 ⚠️ 关键约束：车端不能使用Python

| 约束 | 原因 |
|------|------|
| **Android原生不支持Python** | 需嵌入解释器，增加APK 20-50MB |
| **性能不达标** | 解释型语言，无法满足<500ms实时响应 |
| **系统集成困难** | 无法直接调用AIDL/Service/Vehicle HAL |
| **依赖问题** | LangChain等框架的依赖在Android上无法编译 |
| **稳定性风险** | Python运行时在嵌入式环境稳定性差 |

---

## 2. 车端 vs 云端技术栈对比

### 2.1 技术栈总览

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          车端 vs 云端 技术栈对比                                 │
│                                                                                 │
│  ┌─────────────────────────────────┐    ┌─────────────────────────────────┐   │
│  │           车端 (8295)           │    │             云端                │   │
│  │         Android 10              │    │         Linux Server            │   │
│  ├─────────────────────────────────┤    ├─────────────────────────────────┤   │
│  │                                 │    │                                 │   │
│  │  开发语言:                      │    │  开发语言:                      │   │
│  │  ├── Kotlin (主语言) ✅         │    │  ├── Python (主语言) ✅         │   │
│  │  ├── Java (兼容层)             │    │  ├── Go (高性能服务)            │   │
│  │  └── C++ (性能敏感)            │    │  └── TypeScript (前端)         │   │
│  │                                 │    │                                 │   │
│  │  Agent框架:                     │    │  Agent框架:                     │   │
│  │  └── 自研Kotlin框架 ✅          │    │  ├── LangChain ✅               │   │
│  │      (参考LangChain设计)        │    │  ├── LangGraph                  │   │
│  │                                 │    │  └── FastAPI                    │   │
│  │  AI推理:                        │    │                                 │   │
│  │  ├── QNN (高通原生) ✅          │    │  AI推理:                        │   │
│  │  ├── TensorFlow Lite           │    │  ├── vLLM ✅                     │   │
│  │  └── llama.cpp (Android)       │    │  ├── TGI                        │   │
│  │                                 │    │  └── 云厂商API                  │   │
│  │  语音引擎:                      │    │                                 │   │
│  │  ├── Porcupine (唤醒)          │    │  MCP服务:                       │   │
│  │  ├── Vosk (离线ASR)            │    │  ├── FastAPI ✅                  │   │
│  │  └── 讯飞SDK (在线)            │    │  └── 标准MCP协议                │   │
│  │                                 │    │                                 │   │
│  │  存储:                          │    │  存储:                          │   │
│  │  ├── Room (SQLite)             │    │  ├── PostgreSQL                 │   │
│  │  ├── MMKV                      │    │  ├── Redis                      │   │
│  │  └── Faiss-Android             │    │  └── MongoDB                    │   │
│  │                                 │    │                                 │   │
│  │  通信:                          │    │  通信:                          │   │
│  │  ├── AIDL (进程间)             │    │  ├── gRPC                       │   │
│  │  ├── OkHttp (网络)             │    │  ├── REST API                   │   │
│  │  └── EventBus (组件间)         │    │  └── WebSocket                  │   │
│  │                                 │    │                                 │   │
│  └─────────────────────────────────┘    └─────────────────────────────────┘   │
│                                                                                 │
│  ❌ 车端禁止使用:                                                               │
│  • Python / LangChain / AutoGen / CrewAI                                       │
│  • Node.js / JavaScript运行时                                                  │
│  • 任何需要嵌入解释器的动态语言                                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 语言选型对比分析

| 维度 | 车端 (Kotlin/Java) | 云端 (Python) |
|------|-------------------|---------------|
| **执行效率** | ⭐⭐⭐⭐⭐ 编译型，JIT优化 | ⭐⭐⭐ 解释型 |
| **内存管理** | ⭐⭐⭐⭐⭐ JVM优化 | ⭐⭐⭐ GC不可控 |
| **系统集成** | ⭐⭐⭐⭐⭐ Android原生 | ⭐ 需要桥接 |
| **AI生态** | ⭐⭐⭐ TFLite/QNN | ⭐⭐⭐⭐⭐ 最丰富 |
| **开发效率** | ⭐⭐⭐⭐ 类型安全 | ⭐⭐⭐⭐⭐ 快速迭代 |
| **实时性** | ⭐⭐⭐⭐⭐ 可控 | ⭐⭐ GIL限制 |

---

## 3. 车端技术选型详解

### 3.1 开发语言选型

#### 3.1.1 主开发语言: Kotlin

| 选择 | 理由 |
|------|------|
| **Kotlin** | Android官方推荐语言，空安全，协程支持 |

```kotlin
// Kotlin协程非常适合Agent异步编排
class MasterAgent {
    suspend fun execute(request: AgentRequest): AgentResponse {
        return coroutineScope {
            // 并行调用多个Agent
            val weatherDeferred = async { weatherAgent.execute(request) }
            val vehicleDeferred = async { vehicleAgent.execute(request) }
            
            // 聚合结果
            aggregateResponses(
                weatherDeferred.await(),
                vehicleDeferred.await()
            )
        }
    }
}
```

#### 3.1.2 性能敏感层: C++ (NDK)

| 场景 | 说明 |
|------|------|
| **AI推理** | QNN/TFLite底层都是C++实现 |
| **语音处理** | VAD/AEC/降噪等信号处理 |
| **音频流** | 低延迟音频采集和播放 |

#### 3.1.3 ❌ 禁止使用Python的原因

```
┌─────────────────────────────────────────────────────────────────┐
│            为什么车端不能使用Python/LangChain                    │
│                                                                 │
│  1. 性能问题                                                    │
│     ┌─────────────────────────────────────────────────────┐    │
│     │ 场景: 用户说"打开空调"                               │    │
│     │                                                      │    │
│     │ Kotlin实现:                                          │    │
│     │ 唤醒(200ms) → ASR(300ms) → Agent(50ms) → 车控(100ms)│    │
│     │ 总延迟: ~650ms ✅                                    │    │
│     │                                                      │    │
│     │ Python实现(如果可能):                                │    │
│     │ 唤醒(200ms) → ASR(300ms) → Python启动(500ms)        │    │
│     │ → LangChain加载(1000ms) → Agent(200ms) → 车控(100ms)│    │
│     │ 总延迟: ~2300ms ❌ 用户体验差                        │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
│  2. 依赖问题                                                    │
│     ┌─────────────────────────────────────────────────────┐    │
│     │ LangChain依赖链:                                     │    │
│     │ langchain → pydantic → numpy → ... (100+依赖)       │    │
│     │                                                      │    │
│     │ 问题:                                                │    │
│     │ • numpy需要编译C扩展，Android上编译困难              │    │
│     │ • 很多依赖只支持x86/x64，不支持ARM                   │    │
│     │ • 版本冲突难以解决                                   │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
│  3. 系统集成问题                                                │
│     ┌─────────────────────────────────────────────────────┐    │
│     │ 车控调用链:                                          │    │
│     │                                                      │    │
│     │ Kotlin:                                              │    │
│     │ Agent → AIDL → VehicleService → HAL → CAN          │    │
│     │ (原生支持，直接调用) ✅                              │    │
│     │                                                      │    │
│     │ Python:                                              │    │
│     │ LangChain → ??? → AIDL → VehicleService → HAL      │    │
│     │ (需要JNI桥接，复杂且不稳定) ❌                       │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 车端Agent框架选型

#### 3.2.1 框架选型对比

| 框架 | 语言 | 车端可行性 | 说明 |
|------|------|------------|------|
| **LangChain** | Python | ❌ 不可行 | 无法在Android运行 |
| **AutoGen** | Python | ❌ 不可行 | 依赖问题严重 |
| **CrewAI** | Python | ❌ 不可行 | 同上 |
| **Semantic Kernel** | C#/Python | ❌ 不可行 | Android支持差 |
| **自研Kotlin框架** | Kotlin | ✅ 推荐 | 原生适配，完全可控 |

#### 3.2.2 推荐方案: 自研Kotlin Agent框架

**设计思路**: 参考LangChain/AutoGen的设计理念，用Kotlin重新实现适合车载场景的轻量级框架

```
┌─────────────────────────────────────────────────────────────────┐
│              自研Kotlin Agent框架架构                            │
│                                                                 │
│  核心设计理念 (借鉴自LangChain/AutoGen/ReAct):                  │
│  • Agent抽象: 统一的Agent接口定义                               │
│  • Tool机制: 可插拔的工具调用                                   │
│  • Chain编排: 任务链式执行                                      │
│  • Memory: 上下文记忆管理                                       │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    AgentFramework                          │ │
│  │                                                            │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │ │
│  │  │AgentRegistry│  │ ToolRegistry│  │MemoryStore  │       │ │
│  │  │ • 注册发现  │  │ • Skill注册 │  │ • 对话历史  │       │ │
│  │  │ • 生命周期  │  │ • MCP注册   │  │ • 用户画像  │       │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │ │
│  │                                                            │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │ │
│  │  │ChainExecutor│  │ContextPool  │  │ EventBus    │       │ │
│  │  │ • 任务编排  │  │ • 会话上下文│  │ • 事件驱动  │       │ │
│  │  │ • 条件分支  │  │ • 槽位管理  │  │ • 异步通信  │       │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │ │
│  │                                                            │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  关键类设计:                                                    │
│                                                                 │
│  // Agent基类 (类似LangChain的Agent)                           │
│  abstract class BaseAgent {                                     │
│      abstract val name: String                                  │
│      abstract val tools: List<ITool>                           │
│      abstract suspend fun plan(input: String): List<Action>    │
│      abstract suspend fun execute(action: Action): Result      │
│  }                                                              │
│                                                                 │
│  // Tool接口 (类似LangChain的Tool)                             │
│  interface ITool {                                              │
│      val name: String                                           │
│      val description: String                                    │
│      val parameters: ToolSchema                                 │
│      suspend fun invoke(params: Map<String, Any>): ToolResult  │
│  }                                                              │
│                                                                 │
│  // Chain编排 (类似LangChain的Chain)                           │
│  class AgentChain {                                             │
│      fun addStep(agent: BaseAgent): AgentChain                 │
│      fun addCondition(predicate: (Result) -> Boolean)          │
│      suspend fun execute(input: String): ChainResult           │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 车端AI推理选型

| 框架 | 支持模型 | 8295优化 | 推荐度 |
|------|----------|----------|--------|
| **QNN (Qualcomm)** | 通用 | ⭐⭐⭐⭐⭐ NPU原生 | ✅ 首选 |
| **TensorFlow Lite** | TF模型 | ⭐⭐⭐⭐ GPU委托 | 次选 |
| **ONNX Runtime** | ONNX | ⭐⭐⭐ | 备选 |
| **llama.cpp** | LLM | ⭐⭐⭐ 需适配 | LLM专用 |
| **MLC-LLM** | LLM | ⭐⭐⭐⭐ | LLM备选 |

**推荐组合**:
- 意图识别/NER: QNN + TFLite (小模型，快速)
- 端侧LLM: llama.cpp Android版 + QNN加速

### 3.4 车端存储选型

| 组件 | 技术 | 用途 |
|------|------|------|
| **结构化存储** | Room (SQLite) | 对话历史、配置 |
| **KV存储** | MMKV | 用户偏好、缓存 |
| **向量存储** | Faiss-Android | 本地知识库 |
| **文件存储** | Internal Storage | 模型、音频 |

### 3.5 车端通信选型

| 场景 | 技术 | 说明 |
|------|------|------|
| **进程间通信** | AIDL | 调用VehicleService |
| **组件间通信** | EventBus / Kotlin Flow | Agent间消息 |
| **网络通信** | OkHttp + Retrofit | 调用云端MCP |
| **实时通信** | WebSocket | 长连接服务 |

---

## 4. 云端技术选型详解

### 4.1 云端可以使用Python/LangChain ✅

| 优势 | 说明 |
|------|------|
| **AI生态丰富** | LangChain/LangGraph/AutoGen等成熟框架 |
| **开发效率高** | 快速迭代，原型验证 |
| **资源充足** | 服务器资源不受限 |
| **部署灵活** | Docker/K8s标准化部署 |

### 4.2 云端MCP服务技术栈

```
┌─────────────────────────────────────────────────────────────────┐
│                     云端MCP服务技术栈                            │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                      API Gateway                           │ │
│  │  技术: Kong / Nginx / AWS API Gateway                     │ │
│  │  职责: 路由、鉴权、限流、日志                              │ │
│  └───────────────────────────────────────────────────────────┘ │
│                              │                                  │
│         ┌────────────────────┼────────────────────┐            │
│         ▼                    ▼                    ▼            │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐  │
│  │  LLM Service    │ │  MCP Services   │ │  基础服务        │  │
│  │                 │ │                 │ │                 │  │
│  │  技术栈:        │ │  技术栈:        │ │  技术栈:        │  │
│  │  • Python 3.11  │ │  • Python 3.11  │ │  • Go 1.21      │  │
│  │  • FastAPI      │ │  • FastAPI      │ │  • Gin          │  │
│  │  • LangChain ✅ │ │  • Pydantic     │ │  • gRPC         │  │
│  │  • vLLM         │ │  • httpx        │ │                 │  │
│  │                 │ │                 │ │                 │  │
│  │  服务:          │ │  服务:          │ │  服务:          │  │
│  │  • 对话生成     │ │  • Weather MCP  │ │  • Auth         │  │
│  │  • 意图增强     │ │  • Coffee MCP   │ │  • User         │  │
│  │  • 知识问答     │ │  • Map MCP      │ │  • Config       │  │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘  │
│                              │                                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                      数据层                                │ │
│  │  • PostgreSQL: 用户/订单数据                              │ │
│  │  • Redis: 缓存/会话                                       │ │
│  │  • MongoDB: 日志/非结构化数据                             │ │
│  │  • Milvus/Pinecone: 向量数据库                            │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 云端LangChain使用示例

```python
# 云端MCP服务可以使用LangChain
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from fastapi import FastAPI

app = FastAPI()

# 定义MCP Tool
@tool
def query_weather(location: str) -> dict:
    """查询指定位置的天气信息"""
    # 调用天气API
    return weather_api.get_realtime(location)

@tool  
def create_coffee_order(store_id: str, items: list) -> dict:
    """创建咖啡订单"""
    return coffee_api.create_order(store_id, items)

# 创建LangChain Agent
llm = ChatOpenAI(model="gpt-4")
tools = [query_weather, create_coffee_order]
agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

# MCP服务接口
@app.post("/mcp/execute")
async def execute_mcp(request: MCPRequest):
    result = await agent_executor.ainvoke({"input": request.query})
    return MCPResponse(result=result)
```

---

## 5. Skills vs MCP Tools 技术实现对比

### 5.1 实现方式对比

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Skills vs MCP Tools 实现对比                              │
│                                                                             │
│  ┌────────────────────────────────┐  ┌────────────────────────────────┐   │
│  │     Skill (车端/Kotlin)        │  │    MCP Tool (云端/Python)      │   │
│  ├────────────────────────────────┤  ├────────────────────────────────┤   │
│  │                                │  │                                │   │
│  │  // Kotlin实现                 │  │  # Python/LangChain实现        │   │
│  │  class AirConditionSkill :     │  │  @tool                         │   │
│  │      ISkill {                  │  │  def query_weather(            │   │
│  │                                │  │      location: str             │   │
│  │    override val name =         │  │  ) -> dict:                    │   │
│  │      "air_condition"           │  │      """查询天气"""            │   │
│  │                                │  │      return weather_api.get(   │   │
│  │    override suspend fun        │  │          location              │   │
│  │      execute(                  │  │      )                         │   │
│  │        intent: String,         │  │                                │   │
│  │        params: Map             │  │                                │   │
│  │      ): SkillResult {          │  │                                │   │
│  │        // 直接调用AIDL         │  │                                │   │
│  │        return vehicleService   │  │                                │   │
│  │          .setTemperature(      │  │                                │   │
│  │            params["temp"]      │  │                                │   │
│  │          )                     │  │                                │   │
│  │    }                           │  │                                │   │
│  │  }                             │  │                                │   │
│  │                                │  │                                │   │
│  │  执行位置: 车端本地            │  │  执行位置: 云端服务器          │   │
│  │  延迟: <100ms                  │  │  延迟: 200ms-2s                │   │
│  │  网络依赖: 无                  │  │  网络依赖: 必须                │   │
│  │  框架: 自研Kotlin              │  │  框架: LangChain               │   │
│  │                                │  │                                │   │
│  └────────────────────────────────┘  └────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 调用链路对比

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          调用链路对比                                        │
│                                                                             │
│  Skill调用 (本地):                                                          │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│  │ Vehicle │───▶│AirCond  │───▶│ Vehicle │───▶│  HAL    │                 │
│  │  Agent  │    │  Skill  │    │ Service │    │ (CAN)   │                 │
│  │(Kotlin) │    │(Kotlin) │    │ (AIDL)  │    │         │                 │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘                 │
│       │              │              │              │                        │
│       ▼              ▼              ▼              ▼                        │
│     10ms           20ms           50ms          20ms = 100ms ✅            │
│                                                                             │
│                                                                             │
│  MCP Tool调用 (云端):                                                       │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│  │ Service │───▶│  MCP    │───▶│   网络   │───▶│ Weather │───▶│ 天气API │ │
│  │  Agent  │    │ Client  │    │  传输   │    │  MCP    │    │         │ │
│  │(Kotlin) │    │(Kotlin) │    │         │    │(Python) │    │         │ │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘ │
│       │              │              │              │              │        │
│       ▼              ▼              ▼              ▼              ▼        │
│     10ms           20ms         200ms          100ms          200ms       │
│                                                        = 530ms ✅          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. 语音技术选型

### 6.1 语音唤醒 (KWS)

| 方案 | 类型 | 功耗 | 准确率 | 自定义 | 价格 |
|------|------|------|--------|--------|------|
| **Porcupine** | 离线 | 低 | 高 | 支持 | 商用收费 |
| **Snowboy** | 离线 | 低 | 中 | 支持 | 开源 |
| **讯飞离线唤醒** | 离线 | 低 | 高 | 支持 | 商用收费 |

**推荐: Porcupine**
- 高准确率，低误唤醒
- 支持自定义唤醒词
- 提供Android SDK (Java/Kotlin)

### 6.2 语音识别 (ASR)

| 方案 | 中文支持 | 识别率 | Android SDK | 推荐 |
|------|----------|--------|-------------|------|
| **Vosk** | 优秀 | 良好 | ✅ Java | 离线首选 |
| **讯飞** | 优秀 | 优秀 | ✅ Java | 在线首选 |
| **Whisper.cpp** | 优秀 | 优秀 | ✅ JNI | 备选 |

### 6.3 语音合成 (TTS)

| 方案 | 自然度 | Android SDK | 离线 | 推荐 |
|------|--------|-------------|------|------|
| **讯飞TTS** | 优秀 | ✅ Java | ✅ | 首选 |
| **百度TTS** | 良好 | ✅ Java | ✅ | 备选 |

---

## 7. 端侧LLM选型

### 7.1 模型选型

| 模型 | 参数量 | 内存占用 | 中文能力 | 推荐 |
|------|--------|----------|----------|------|
| **Qwen2-1.5B** | 1.5B | ~3GB | 优秀 | ✅ 首选 |
| **MiniCPM-2B** | 2B | ~4GB | 优秀 | 备选 |
| **Gemma-2B** | 2B | ~4GB | 一般 | 备选 |

### 7.2 推理框架选型

| 框架 | 语言 | Android支持 | 8295优化 |
|------|------|-------------|----------|
| **llama.cpp** | C++ | ✅ JNI封装 | 需定制 |
| **MLC-LLM** | C++ | ✅ | 支持 |
| **QNN** | C++ | ✅ | 最优 |

**推荐: llama.cpp + QNN**
- llama.cpp提供模型加载和推理接口
- QNN提供NPU硬件加速

---

## 8. 技术选型总结

### 8.1 最终技术栈

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           最终技术栈总览                                     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         车端 (8295 + Android 10)                     │   │
│  │                                                                      │   │
│  │  开发语言: Kotlin (主) + C++ (性能层)                               │   │
│  │  Agent框架: 自研Kotlin框架 (参考LangChain设计)                       │   │
│  │  AI推理: QNN + llama.cpp                                            │   │
│  │  语音: Porcupine + Vosk + 讯飞                                       │   │
│  │  存储: Room + MMKV + Faiss-Android                                   │   │
│  │  通信: AIDL + EventBus + OkHttp                                      │   │
│  │                                                                      │   │
│  │  ❌ 不使用: Python / LangChain / AutoGen / Node.js                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                              云端                                    │   │
│  │                                                                      │   │
│  │  开发语言: Python (AI服务) + Go (高性能服务)                         │   │
│  │  Agent框架: LangChain / LangGraph ✅                                 │   │
│  │  MCP服务: FastAPI + Pydantic                                         │   │
│  │  LLM推理: vLLM / 云厂商API                                           │   │
│  │  存储: PostgreSQL + Redis + Milvus                                   │   │
│  │  网关: Kong / Nginx                                                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 关键决策总结

| 决策点 | 车端 | 云端 | 理由 |
|--------|------|------|------|
| **开发语言** | Kotlin/C++ | Python/Go | 车端需原生性能，云端需AI生态 |
| **Agent框架** | 自研Kotlin | LangChain | 车端无法运行Python |
| **LLM推理** | QNN+llama.cpp | vLLM/API | 车端需NPU加速 |
| **通信** | AIDL | gRPC/REST | 车端需Android IPC |

---

**文档版本历史**

| 版本 | 日期 | 修改内容 |
|------|------|----------|
| V1.0 | 2026-01-14 | 初始版本 |
| V2.0 | 2026-01-14 | 明确车端禁止使用Python，区分车端/云端技术栈 |
