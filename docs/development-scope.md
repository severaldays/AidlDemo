# 汽车座舱AI智能体系统 - 开发边界与工作划分

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档类型 | 开发边界划分 |
| 版本 | V1.0 |
| 创建日期 | 2026-01-14 |

---

## 1. 系统边界总览

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                      云端 (Cloud)                                        │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                              云端AI服务                                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │
│  │  │ LLM推理服务  │  │ ASR云服务   │  │ TTS云服务   │  │ 知识库服务   │            │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                           MCP服务层 (MCP Server)                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │
│  │  │ 天气MCP服务  │  │ 咖啡MCP服务  │  │ 地图MCP服务  │  │ 用户MCP服务  │            │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                          │
                                    4G/5G 网络
                                          │
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                               车端 (On-Device / 8295)                                    │
│                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                            AI Agent应用层                                        │   │
│  │  ┌───────────────────────────────────────────────────────────────────────────┐  │   │
│  │  │                         Agent Framework                                    │  │   │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │  │   │
│  │  │  │Master Agent │  │Vehicle Agent│  │Service Agent│  │Assistant    │      │  │   │
│  │  │  │  (协调者)   │  │  (车控)     │  │  (云服务)   │  │  Agent      │      │  │   │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │  │   │
│  │  └───────────────────────────────────────────────────────────────────────────┘  │   │
│  │                                                                                  │   │
│  │  ┌───────────────────────────────────────────────────────────────────────────┐  │   │
│  │  │                          Skills层 (本地技能)                               │  │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │  │   │
│  │  │  │空调控制   │ │座椅控制   │ │车窗控制   │ │氛围灯    │ │本地查询   │       │  │   │
│  │  │  │ Skill    │ │ Skill    │ │ Skill    │ │ Skill    │ │ Skill    │       │  │   │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘       │  │   │
│  │  └───────────────────────────────────────────────────────────────────────────┘  │   │
│  │                                                                                  │   │
│  │  ┌───────────────────────────────────────────────────────────────────────────┐  │   │
│  │  │                       MCP Client (云服务调用)                              │  │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                     │  │   │
│  │  │  │天气Client │ │咖啡Client │ │地图Client │ │用户Client │                     │  │   │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘                     │  │   │
│  │  └───────────────────────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                            语音处理层                                            │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │
│  │  │ 离线唤醒     │  │ 离线ASR     │  │ 离线TTS     │  │ VAD/AEC    │            │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                            车控服务层 (Vehicle HAL)                              │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │
│  │  │ 空调Service  │  │ 座椅Service  │  │ 车窗Service  │  │ 灯光Service  │            │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 车端开发工作 (On-Device Development)

### 2.1 车端组件清单

| 模块 | 组件 | 技术栈 | 职责 | 优先级 |
|------|------|--------|------|--------|
| **Agent Framework** | AgentManager | Kotlin | Agent生命周期管理、注册发现 | P0 |
| | MasterAgent | Kotlin | 意图路由、任务编排、结果聚合 | P0 |
| | AgentExecutor | Kotlin | Agent执行引擎 | P0 |
| | ContextManager | Kotlin | 会话上下文、多轮对话管理 | P0 |
| | EventBus | Kotlin | Agent间消息通信 | P0 |
| **Vehicle Agent** | VehicleAgent | Kotlin | 车控领域Agent | P0 |
| | VehicleSkillManager | Kotlin | 车控技能管理 | P0 |
| **Service Agent** | ServiceAgent | Kotlin | 云服务领域Agent | P1 |
| | MCPClientManager | Kotlin | MCP服务调用管理 | P1 |
| **Assistant Agent** | AssistantAgent | Kotlin | 通用对话Agent | P2 |
| **语音模块** | WakeupEngine | Kotlin/C++ | 离线唤醒引擎 | P0 |
| | ASREngine | Kotlin/C++ | 语音识别引擎(离线+在线) | P0 |
| | TTSEngine | Kotlin/C++ | 语音合成引擎(离线+在线) | P0 |
| | AudioManager | Kotlin | 音频采集/播放管理 | P0 |
| **本地AI** | LocalLLM | Kotlin/C++ | 端侧LLM推理 | P1 |
| | IntentClassifier | Kotlin | 本地意图分类 | P0 |
| | NERExtractor | Kotlin | 本地实体提取 | P0 |
| **车控服务** | VehicleService | Java/AIDL | 车控AIDL服务 | P0 |
| | VehicleHAL | C++/HIDL | 硬件抽象层 | P0 |
| **数据存储** | LocalDatabase | Kotlin/Room | 本地数据存储 | P1 |
| | PreferenceManager | Kotlin | 用户偏好存储 | P1 |

### 2.2 车端详细模块设计

#### 2.2.1 Agent Framework (车端核心)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Agent Framework 架构                          │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    AgentManager                            │ │
│  │  • Agent注册/注销                                          │ │
│  │  • Agent生命周期管理                                       │ │
│  │  • Agent健康检查                                           │ │
│  └───────────────────────────────────────────────────────────┘ │
│                              │                                  │
│         ┌────────────────────┼────────────────────┐            │
│         ▼                    ▼                    ▼            │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│  │ MasterAgent │     │ContextMgr  │     │ SkillRegistry│      │
│  │ • 意图路由  │     │ • 会话管理  │     │ • Skill注册 │      │
│  │ • 任务编排  │     │ • 历史记录  │     │ • Skill发现 │      │
│  │ • 结果聚合  │     │ • 槽位管理  │     │ • Skill调用 │      │
│  └─────────────┘     └─────────────┘     └─────────────┘      │
│                              │                                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                      EventBus                              │ │
│  │  • 发布/订阅模式                                           │ │
│  │  • Agent间异步通信                                         │ │
│  │  • 事件优先级队列                                          │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 2.2.2 Vehicle Agent (车控Agent)

```
┌─────────────────────────────────────────────────────────────────┐
│                     Vehicle Agent 架构                           │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    VehicleAgent                            │ │
│  │  • 车控意图处理                                            │ │
│  │  • Skill选择与调用                                         │ │
│  │  • 车辆状态查询                                            │ │
│  └───────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                  VehicleSkillManager                       │ │
│  │                                                            │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │ │
│  │  │AirCondition│ │  Seat    │ │ Window  │ │  Light   │     │ │
│  │  │  Skill   │ │  Skill   │ │  Skill   │ │  Skill   │     │ │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘     │ │
│  │       │            │            │            │            │ │
│  └───────┼────────────┼────────────┼────────────┼────────────┘ │
│          │            │            │            │              │
│          ▼            ▼            ▼            ▼              │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                   VehicleService (AIDL)                    │ │
│  └───────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    Vehicle HAL                             │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 云端开发工作 (Cloud Development)

### 3.1 云端组件清单

| 模块 | 组件 | 技术栈 | 职责 | 优先级 |
|------|------|--------|------|--------|
| **AI服务** | LLM Gateway | Python/FastAPI | LLM调用网关、负载均衡 | P0 |
| | LLM Service | Python | 大模型推理服务 | P0 |
| | ASR Service | Python | 云端语音识别 | P1 |
| | TTS Service | Python | 云端语音合成 | P1 |
| **MCP服务** | Weather MCP | Python/FastAPI | 天气查询MCP服务 | P1 |
| | Coffee MCP | Python/FastAPI | 咖啡订购MCP服务 | P1 |
| | Map MCP | Python/FastAPI | 地图导航MCP服务 | P2 |
| | User MCP | Python/FastAPI | 用户服务MCP | P1 |
| **基础服务** | API Gateway | Go/Kong | API网关、鉴权、限流 | P0 |
| | Auth Service | Go | 认证授权服务 | P0 |
| | Config Service | Go | 配置中心 | P1 |
| **数据服务** | User DB | PostgreSQL | 用户数据存储 | P0 |
| | Order DB | PostgreSQL | 订单数据存储 | P1 |
| | Cache | Redis | 缓存服务 | P0 |
| | MQ | RabbitMQ/Kafka | 消息队列 | P1 |

### 3.2 云端架构设计

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              云端服务架构                                    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          API Gateway                                 │   │
│  │  • 统一入口      • 鉴权认证      • 限流熔断      • 日志监控         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│          ┌─────────────────────────┼─────────────────────────┐             │
│          ▼                         ▼                         ▼             │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐     │
│  │   AI服务集群      │    │   MCP服务集群     │    │   基础服务集群    │     │
│  │                  │    │                  │    │                  │     │
│  │  ┌────────────┐ │    │  ┌────────────┐ │    │  ┌────────────┐ │     │
│  │  │LLM Gateway │ │    │  │Weather MCP │ │    │  │Auth Service│ │     │
│  │  └────────────┘ │    │  └────────────┘ │    │  └────────────┘ │     │
│  │  ┌────────────┐ │    │  ┌────────────┐ │    │  ┌────────────┐ │     │
│  │  │LLM Service │ │    │  │Coffee MCP  │ │    │  │User Service│ │     │
│  │  └────────────┘ │    │  └────────────┘ │    │  └────────────┘ │     │
│  │  ┌────────────┐ │    │  ┌────────────┐ │    │  ┌────────────┐ │     │
│  │  │ASR/TTS Svc │ │    │  │Map MCP     │ │    │  │Config Svc  │ │     │
│  │  └────────────┘ │    │  └────────────┘ │    │  └────────────┘ │     │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘     │
│                                    │                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          数据存储层                                  │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐    │   │
│  │  │ PostgreSQL │  │   Redis    │  │  MongoDB   │  │   Kafka    │    │   │
│  │  │  用户/订单  │  │    缓存    │  │   日志     │  │   消息队列  │    │   │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. MCP服务定义 (Model Context Protocol)

### 4.1 MCP服务概述

MCP (Model Context Protocol) 是为AI Agent提供外部能力的标准化服务协议。每个MCP服务封装一个特定领域的能力，提供标准化的接口供Agent调用。

### 4.2 MCP服务清单

| MCP服务 | 部署位置 | 职责 | 提供的Tools |
|---------|----------|------|-------------|
| **Weather MCP** | 云端 | 天气信息服务 | query_weather, get_forecast, get_alert |
| **Coffee MCP** | 云端 | 咖啡订购服务 | query_menu, find_stores, create_order, get_order_status |
| **Map MCP** | 云端 | 地图导航服务 | search_poi, get_route, get_traffic |
| **User MCP** | 云端 | 用户信息服务 | get_profile, get_preferences, update_preferences |
| **Music MCP** | 云端 | 音乐服务 | search_music, play_music, get_playlist |
| **Vehicle MCP** | 车端 | 车辆信息服务 | get_vehicle_status, get_vehicle_info |

### 4.3 MCP服务接口规范

```
┌─────────────────────────────────────────────────────────────────┐
│                      MCP服务接口规范                             │
│                                                                 │
│  MCP服务需要实现以下标准接口:                                    │
│                                                                 │
│  1. 服务发现接口                                                │
│     GET /mcp/info                                               │
│     返回: 服务名称、版本、描述、支持的Tools列表                  │
│                                                                 │
│  2. Tool列表接口                                                │
│     GET /mcp/tools                                              │
│     返回: 所有Tool的定义(名称、描述、参数Schema)                 │
│                                                                 │
│  3. Tool执行接口                                                │
│     POST /mcp/tools/{tool_name}/execute                         │
│     请求: Tool参数                                              │
│     返回: Tool执行结果                                          │
│                                                                 │
│  4. 健康检查接口                                                │
│     GET /mcp/health                                             │
│     返回: 服务健康状态                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.4 MCP服务详细定义

#### 4.4.1 Weather MCP (天气服务)

```yaml
service:
  name: weather-mcp
  version: "1.0"
  description: "提供天气查询、预报、预警等服务"
  
tools:
  - name: query_weather
    description: "查询指定位置的实时天气"
    parameters:
      - name: location
        type: string
        description: "位置(城市名或经纬度)"
        required: true
    returns:
      type: object
      properties:
        temperature: { type: integer, description: "温度(℃)" }
        humidity: { type: integer, description: "湿度(%)" }
        weather: { type: string, description: "天气状况" }
        wind: { type: string, description: "风向风力" }
        aqi: { type: integer, description: "空气质量指数" }
        
  - name: get_forecast
    description: "获取天气预报"
    parameters:
      - name: location
        type: string
        required: true
      - name: days
        type: integer
        description: "预报天数(1-7)"
        default: 3
    returns:
      type: array
      items:
        type: object
        properties:
          date: { type: string }
          high_temp: { type: integer }
          low_temp: { type: integer }
          weather: { type: string }
          
  - name: get_alert
    description: "获取气象预警"
    parameters:
      - name: location
        type: string
        required: true
    returns:
      type: array
      items:
        type: object
        properties:
          title: { type: string }
          level: { type: string }
          description: { type: string }
```

#### 4.4.2 Coffee MCP (咖啡订购服务)

```yaml
service:
  name: coffee-mcp
  version: "1.0"
  description: "提供咖啡菜单查询、门店查找、订单管理等服务"
  
tools:
  - name: query_menu
    description: "查询咖啡菜单"
    parameters:
      - name: store_id
        type: string
        description: "门店ID(可选)"
        required: false
      - name: category
        type: string
        description: "分类筛选"
        required: false
    returns:
      type: object
      properties:
        categories: { type: array }
        recommendations: { type: array }
        
  - name: find_stores
    description: "查找附近门店"
    parameters:
      - name: latitude
        type: number
        required: true
      - name: longitude
        type: number
        required: true
      - name: radius
        type: integer
        description: "搜索半径(米)"
        default: 5000
    returns:
      type: array
      items:
        type: object
        properties:
          store_id: { type: string }
          name: { type: string }
          address: { type: string }
          distance: { type: integer }
          status: { type: string }
          
  - name: create_order
    description: "创建咖啡订单"
    parameters:
      - name: store_id
        type: string
        required: true
      - name: items
        type: array
        required: true
        items:
          type: object
          properties:
            item_id: { type: string }
            quantity: { type: integer }
            size: { type: string }
            options: { type: object }
      - name: pickup_type
        type: string
        enum: ["store_pickup", "car_pickup"]
        default: "car_pickup"
    returns:
      type: object
      properties:
        order_id: { type: string }
        status: { type: string }
        total_amount: { type: number }
        payment_url: { type: string }
        
  - name: get_order_status
    description: "查询订单状态"
    parameters:
      - name: order_id
        type: string
        required: true
    returns:
      type: object
      properties:
        order_id: { type: string }
        status: { type: string }
        status_text: { type: string }
        pickup_code: { type: string }
        estimated_ready_time: { type: string }
```

---

## 5. Skills定义 (本地技能)

### 5.1 Skills概述

Skills是车端本地执行的能力单元，不依赖云端服务，可以离线执行。每个Skill封装一个具体的车控或本地功能。

### 5.2 Skills清单

| Skill名称 | 所属Agent | 执行位置 | 依赖 | 描述 |
|-----------|-----------|----------|------|------|
| **AirConditionSkill** | VehicleAgent | 车端 | VehicleService | 空调控制 |
| **SeatSkill** | VehicleAgent | 车端 | VehicleService | 座椅调节 |
| **WindowSkill** | VehicleAgent | 车端 | VehicleService | 车窗控制 |
| **LightSkill** | VehicleAgent | 车端 | VehicleService | 灯光控制 |
| **VolumeSkill** | VehicleAgent | 车端 | AudioService | 音量控制 |
| **LocalQuerySkill** | AssistantAgent | 车端 | LocalDB | 本地信息查询 |
| **TimerSkill** | AssistantAgent | 车端 | AlarmManager | 定时/提醒 |
| **CalculatorSkill** | AssistantAgent | 车端 | 无 | 计算 |

### 5.3 Skills vs MCP Tools 对比

```
┌─────────────────────────────────────────────────────────────────┐
│                   Skills vs MCP Tools 对比                       │
│                                                                 │
│  ┌─────────────────────────────┬─────────────────────────────┐ │
│  │         Skills              │       MCP Tools             │ │
│  ├─────────────────────────────┼─────────────────────────────┤ │
│  │ 执行位置: 车端本地           │ 执行位置: 云端服务          │ │
│  │ 网络依赖: 无                 │ 网络依赖: 必须              │ │
│  │ 响应速度: 快(<100ms)        │ 响应速度: 中(200ms-2s)     │ │
│  │ 离线可用: 是                 │ 离线可用: 否                │ │
│  │ 功能范围: 车控、本地功能     │ 功能范围: 云端服务、外部API │ │
│  │ 更新方式: OTA更新           │ 更新方式: 服务端部署        │ │
│  └─────────────────────────────┴─────────────────────────────┘ │
│                                                                 │
│  选择原则:                                                      │
│  • 车控操作 → Skill                                            │
│  • 本地数据查询 → Skill                                        │
│  • 需要网络数据 → MCP Tool                                     │
│  • 需要云端计算 → MCP Tool                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Skill详细定义

#### 5.4.1 AirConditionSkill (空调控制技能)

```kotlin
/**
 * 空调控制技能
 * 执行位置: 车端
 * 依赖: VehicleService (AIDL)
 */
class AirConditionSkill : ISkill {
    
    override fun getName() = "air_condition"
    
    override fun getDescription() = "控制车辆空调，支持开关、温度、风速、模式调节"
    
    override fun getSupportedIntents() = listOf(
        "turn_on_ac",           // 打开空调
        "turn_off_ac",          // 关闭空调
        "set_ac_temperature",   // 设置温度
        "increase_temperature", // 升高温度
        "decrease_temperature", // 降低温度
        "set_ac_fan_speed",     // 设置风速
        "set_ac_mode",          // 设置模式
        "turn_on_ac_auto",      // 开启自动模式
        "defrost_front",        // 前挡除雾
        "defrost_rear",         // 后挡除雾
        "query_ac_status"       // 查询空调状态
    )
    
    override fun getParameterSchema() = mapOf(
        "temperature" to ParameterDef(
            type = ParamType.INTEGER,
            description = "目标温度",
            minimum = 16,
            maximum = 32
        ),
        "fan_speed" to ParameterDef(
            type = ParamType.INTEGER,
            description = "风速档位",
            minimum = 1,
            maximum = 7
        ),
        "mode" to ParameterDef(
            type = ParamType.ENUM,
            description = "风向模式",
            enumValues = listOf("face", "feet", "face_feet", "windshield")
        )
    )
    
    override suspend fun execute(intent: String, params: Map<String, Any>): SkillResult
}
```

#### 5.4.2 SeatSkill (座椅控制技能)

```kotlin
/**
 * 座椅控制技能
 * 执行位置: 车端
 * 依赖: VehicleService (AIDL)
 */
class SeatSkill : ISkill {
    
    override fun getName() = "seat"
    
    override fun getDescription() = "控制车辆座椅，支持位置、加热、通风、按摩调节"
    
    override fun getSupportedIntents() = listOf(
        "adjust_seat_position",   // 调节座椅位置
        "move_seat_forward",      // 座椅前移
        "move_seat_backward",     // 座椅后移
        "adjust_seat_backrest",   // 调节靠背
        "turn_on_seat_heating",   // 开启座椅加热
        "turn_off_seat_heating",  // 关闭座椅加热
        "set_seat_heating_level", // 设置加热档位
        "turn_on_seat_ventilation",  // 开启座椅通风
        "turn_off_seat_ventilation", // 关闭座椅通风
        "turn_on_seat_massage",   // 开启按摩
        "turn_off_seat_massage",  // 关闭按摩
        "save_seat_memory",       // 保存座椅记忆
        "restore_seat_memory",    // 恢复座椅记忆
        "query_seat_status"       // 查询座椅状态
    )
    
    override fun getParameterSchema() = mapOf(
        "seat_id" to ParameterDef(
            type = ParamType.ENUM,
            description = "座椅位置",
            enumValues = listOf("driver", "passenger", "rear_left", "rear_right")
        ),
        "position" to ParameterDef(
            type = ParamType.INTEGER,
            description = "位置(0-100)",
            minimum = 0,
            maximum = 100
        ),
        "heating_level" to ParameterDef(
            type = ParamType.INTEGER,
            description = "加热档位",
            minimum = 0,
            maximum = 3
        ),
        "ventilation_level" to ParameterDef(
            type = ParamType.INTEGER,
            description = "通风档位",
            minimum = 0,
            maximum = 3
        ),
        "massage_mode" to ParameterDef(
            type = ParamType.ENUM,
            description = "按摩模式",
            enumValues = listOf("off", "wave", "pulse", "comfort")
        ),
        "memory_slot" to ParameterDef(
            type = ParamType.INTEGER,
            description = "记忆槽位",
            minimum = 1,
            maximum = 3
        )
    )
    
    override suspend fun execute(intent: String, params: Map<String, Any>): SkillResult
}
```

---

## 6. 开发工作分工

### 6.1 工作量估算

```
┌─────────────────────────────────────────────────────────────────┐
│                        开发工作分工                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    车端开发 (60%)                        │   │
│  │                                                          │   │
│  │  • Agent Framework核心框架        3人周                   │   │
│  │  • Master Agent                   2人周                   │   │
│  │  • Vehicle Agent + Skills         3人周                   │   │
│  │  • Service Agent + MCP Client     2人周                   │   │
│  │  • 语音模块集成                   2人周                   │   │
│  │  • 本地LLM/NLU集成                2人周                   │   │
│  │  • Vehicle Service (AIDL)         2人周                   │   │
│  │  • UI界面开发                     2人周                   │   │
│  │  ────────────────────────────────────                    │   │
│  │  小计: 18人周                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    云端开发 (40%)                        │   │
│  │                                                          │   │
│  │  • API Gateway + 基础服务         2人周                   │   │
│  │  • LLM Gateway服务                2人周                   │   │
│  │  • Weather MCP服务                1人周                   │   │
│  │  • Coffee MCP服务                 2人周                   │   │
│  │  • User MCP服务                   1人周                   │   │
│  │  • Map MCP服务                    1人周                   │   │
│  │  • 数据库设计与实现               1人周                   │   │
│  │  • 运维部署                       2人周                   │   │
│  │  ────────────────────────────────────                    │   │
│  │  小计: 12人周                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  总计: 30人周 (约2个月，3人团队)                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 团队分工建议

| 角色 | 人数 | 职责范围 |
|------|------|----------|
| **车端架构师** | 1 | Agent Framework、Master Agent、整体架构 |
| **车端开发** | 2 | Vehicle Agent、Skills、语音集成、UI |
| **云端开发** | 1-2 | MCP服务、API Gateway、数据库 |
| **AI工程师** | 1 | LLM集成、NLU模型、意图识别 |
| **测试工程师** | 1 | 功能测试、集成测试、性能测试 |

### 6.3 开发里程碑

| 阶段 | 周期 | 车端工作 | 云端工作 | 交付物 |
|------|------|----------|----------|--------|
| **M1** | 2周 | Agent Framework基础 | API Gateway | 框架Demo |
| **M2** | 3周 | Vehicle Agent + Skills | Weather/Coffee MCP | 车控+天气 |
| **M3** | 2周 | Service Agent + 语音 | User MCP | 语音交互 |
| **M4** | 2周 | 本地LLM + NLU | LLM Gateway | 智能对话 |
| **M5** | 2周 | 集成测试 + 优化 | 性能优化 | 完整系统 |
| **M6** | 1周 | Bug修复 | Bug修复 | 发布版本 |

---

## 7. 总结

### 7.1 核心概念关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           核心概念关系                                       │
│                                                                             │
│                        ┌─────────────┐                                      │
│                        │   Agent     │                                      │
│                        │  (智能体)   │                                      │
│                        └──────┬──────┘                                      │
│                               │                                             │
│               ┌───────────────┼───────────────┐                            │
│               ▼               ▼               ▼                            │
│        ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                    │
│        │   Skills    │ │ MCP Tools   │ │   Context   │                    │
│        │ (本地技能)  │ │ (云端工具)  │ │  (上下文)   │                    │
│        └──────┬──────┘ └──────┬──────┘ └─────────────┘                    │
│               │               │                                            │
│               ▼               ▼                                            │
│        ┌─────────────┐ ┌─────────────┐                                    │
│        │  Vehicle    │ │    MCP      │                                    │
│        │  Service    │ │   Server    │                                    │
│        │ (车端服务)  │ │ (云端服务)  │                                    │
│        └──────┬──────┘ └──────┬──────┘                                    │
│               │               │                                            │
│               ▼               ▼                                            │
│        ┌─────────────┐ ┌─────────────┐                                    │
│        │  Vehicle    │ │   Cloud     │                                    │
│        │    HAL      │ │   Infra     │                                    │
│        │ (车辆硬件)  │ │ (云端基础)  │                                    │
│        └─────────────┘ └─────────────┘                                    │
│                                                                             │
│  Skill: 本地执行的能力，如空调控制、座椅调节                                 │
│  MCP Tool: 通过MCP协议调用的云端能力，如天气查询、咖啡订购                   │
│  MCP Server: 云端部署的服务，实现MCP协议，提供Tools                         │
│  Agent: 智能体，根据意图选择调用Skill或MCP Tool                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 一句话总结

| 概念 | 定义 |
|------|------|
| **车端** | 运行在8295芯片上的所有组件，包括Agent、Skills、语音引擎、车控服务 |
| **云端** | 运行在云服务器上的组件，包括MCP服务、LLM服务、API网关 |
| **Skill** | 车端本地技能，离线可用，主要用于车控操作 |
| **MCP Server** | 云端服务，遵循MCP协议，提供标准化的Tools供Agent调用 |
| **MCP Tool** | MCP Server提供的具体能力，如查询天气、创建订单 |
| **Agent** | 智能体，负责理解意图、调度Skill和MCP Tool、生成响应 |

---

**文档版本历史**

| 版本 | 日期 | 修改内容 |
|------|------|----------|
| V1.0 | 2026-01-14 | 初始版本 |
