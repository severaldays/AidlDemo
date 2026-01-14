# 汽车座舱AI智能体系统 - 接口设计规范

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档类型 | 接口设计规范 |
| 版本 | V1.0 |
| 创建日期 | 2026-01-14 |

---

## 1. 概述

本文档定义汽车座舱AI智能体系统的接口设计规范，包括Agent接口、Tool接口、车控服务接口和云服务API接口。

---

## 2. Agent接口规范

### 2.1 Agent基础接口

```kotlin
/**
 * Agent基础接口定义
 * 所有Agent必须实现此接口
 */
interface IAgent {
    
    /**
     * 获取Agent唯一标识
     * @return Agent ID，格式: {domain}_{name}_{version}
     */
    fun getAgentId(): String
    
    /**
     * 获取Agent名称
     * @return 可读的Agent名称
     */
    fun getAgentName(): String
    
    /**
     * 获取Agent描述
     * @return Agent功能描述
     */
    fun getDescription(): String
    
    /**
     * 获取Agent能力列表
     * @return 支持的意图列表
     */
    fun getCapabilities(): List<String>
    
    /**
     * 获取Agent优先级
     * @return 优先级，数值越大优先级越高
     */
    fun getPriority(): Int
    
    /**
     * 判断是否能处理指定意图
     * @param intent 意图标识
     * @return 是否能处理
     */
    fun canHandle(intent: String): Boolean
    
    /**
     * 执行Agent任务
     * @param request 请求对象
     * @return 响应对象
     */
    suspend fun execute(request: AgentRequest): AgentResponse
    
    /**
     * 初始化Agent
     * @param config 配置对象
     */
    fun initialize(config: AgentConfig)
    
    /**
     * 销毁Agent，释放资源
     */
    fun destroy()
    
    /**
     * 获取当前状态
     * @return Agent状态
     */
    fun getState(): AgentState
}

/**
 * Agent状态枚举
 */
enum class AgentState {
    CREATED,      // 已创建
    INITIALIZED,  // 已初始化
    READY,        // 就绪
    RUNNING,      // 运行中
    SUSPENDED,    // 暂停
    DESTROYED     // 已销毁
}
```

### 2.2 Agent请求/响应模型

```kotlin
/**
 * Agent请求对象
 */
data class AgentRequest(
    /** 请求唯一ID */
    val requestId: String,
    
    /** 会话ID，用于多轮对话 */
    val sessionId: String,
    
    /** 用户ID */
    val userId: String,
    
    /** 识别的意图 */
    val intent: String,
    
    /** 原始用户输入 */
    val rawInput: String,
    
    /** 提取的参数 */
    val parameters: Map<String, Any>,
    
    /** 对话上下文 */
    val context: ConversationContext,
    
    /** 请求优先级 */
    val priority: Priority = Priority.NORMAL,
    
    /** 超时时间(毫秒) */
    val timeoutMs: Long = 5000,
    
    /** 请求时间戳 */
    val timestamp: Long = System.currentTimeMillis(),
    
    /** 扩展属性 */
    val extras: Map<String, Any> = emptyMap()
)

/**
 * Agent响应对象
 */
data class AgentResponse(
    /** 对应的请求ID */
    val requestId: String,
    
    /** 是否执行成功 */
    val success: Boolean,
    
    /** 响应码 */
    val code: Int,
    
    /** 响应消息(用于TTS播报) */
    val message: String,
    
    /** 响应数据 */
    val data: Any? = null,
    
    /** 需要执行的动作列表 */
    val actions: List<Action> = emptyList(),
    
    /** 建议的下一轮意图 */
    val nextIntent: String? = null,
    
    /** 置信度 0.0-1.0 */
    val confidence: Float = 1.0f,
    
    /** 是否需要继续对话 */
    val requireFollowUp: Boolean = false,
    
    /** 后续问题提示 */
    val suggestions: List<String> = emptyList(),
    
    /** 响应时间戳 */
    val timestamp: Long = System.currentTimeMillis(),
    
    /** 扩展属性 */
    val extras: Map<String, Any> = emptyMap()
)

/**
 * 对话上下文
 */
data class ConversationContext(
    /** 会话ID */
    val sessionId: String,
    
    /** 对话轮次 */
    val turnCount: Int,
    
    /** 历史对话 */
    val history: List<DialogTurn>,
    
    /** 已识别的槽位 */
    val slots: Map<String, Any>,
    
    /** 用户偏好 */
    val userPreferences: UserPreferences,
    
    /** 当前车辆状态 */
    val vehicleState: VehicleState? = null
)

/**
 * 优先级枚举
 */
enum class Priority(val value: Int) {
    LOW(1),
    NORMAL(5),
    HIGH(8),
    URGENT(10)
}
```

### 2.3 Master Agent接口

```kotlin
/**
 * Master Agent接口
 * 负责任务协调和Agent调度
 */
interface IMasterAgent : IAgent {
    
    /**
     * 注册子Agent
     * @param agent 要注册的Agent
     */
    fun registerAgent(agent: IAgent)
    
    /**
     * 注销子Agent
     * @param agentId Agent ID
     */
    fun unregisterAgent(agentId: String)
    
    /**
     * 获取所有已注册的Agent
     * @return Agent列表
     */
    fun getRegisteredAgents(): List<IAgent>
    
    /**
     * 路由意图到合适的Agent
     * @param intent 意图
     * @return 匹配的Agent列表(按优先级排序)
     */
    fun routeIntent(intent: String): List<IAgent>
    
    /**
     * 分解复杂任务
     * @param request 原始请求
     * @return 子任务列表
     */
    fun decomposeTask(request: AgentRequest): List<SubTask>
    
    /**
     * 聚合多个Agent的响应
     * @param responses Agent响应列表
     * @return 聚合后的响应
     */
    fun aggregateResponses(responses: List<AgentResponse>): AgentResponse
}

/**
 * 子任务定义
 */
data class SubTask(
    /** 任务ID */
    val taskId: String,
    
    /** 目标Agent ID */
    val targetAgentId: String,
    
    /** 任务请求 */
    val request: AgentRequest,
    
    /** 依赖的任务ID列表 */
    val dependencies: List<String> = emptyList(),
    
    /** 执行顺序 */
    val order: Int = 0
)
```

---

## 3. Tool接口规范

### 3.1 Tool基础接口

```kotlin
/**
 * Tool基础接口
 * 定义Agent可调用的工具
 */
interface ITool {
    
    /**
     * 获取Tool名称
     * @return Tool名称，英文，下划线分隔
     */
    fun getName(): String
    
    /**
     * 获取Tool描述
     * @return Tool功能描述，用于LLM理解
     */
    fun getDescription(): String
    
    /**
     * 获取参数Schema
     * @return JSON Schema格式的参数定义
     */
    fun getParameterSchema(): ToolSchema
    
    /**
     * 执行Tool
     * @param parameters 参数Map
     * @return 执行结果
     */
    suspend fun execute(parameters: Map<String, Any>): ToolResult
    
    /**
     * 验证参数
     * @param parameters 参数Map
     * @return 验证结果
     */
    fun validateParameters(parameters: Map<String, Any>): ValidationResult
    
    /**
     * 获取Tool类型
     * @return Tool类型
     */
    fun getType(): ToolType
}

/**
 * Tool类型枚举
 */
enum class ToolType {
    VEHICLE_CONTROL,  // 车控类
    CLOUD_SERVICE,    // 云服务类
    LOCAL_QUERY,      // 本地查询类
    SYSTEM            // 系统类
}

/**
 * Tool Schema定义
 */
data class ToolSchema(
    /** 参数列表 */
    val parameters: List<ParameterDef>,
    
    /** 必填参数名称列表 */
    val required: List<String> = emptyList()
)

/**
 * 参数定义
 */
data class ParameterDef(
    /** 参数名称 */
    val name: String,
    
    /** 参数类型 */
    val type: ParamType,
    
    /** 参数描述 */
    val description: String,
    
    /** 枚举值(当type为ENUM时) */
    val enumValues: List<String>? = null,
    
    /** 最小值(当type为NUMBER/INTEGER时) */
    val minimum: Number? = null,
    
    /** 最大值(当type为NUMBER/INTEGER时) */
    val maximum: Number? = null,
    
    /** 默认值 */
    val default: Any? = null
)

/**
 * 参数类型
 */
enum class ParamType {
    STRING,
    INTEGER,
    NUMBER,
    BOOLEAN,
    ENUM,
    OBJECT,
    ARRAY
}

/**
 * Tool执行结果
 */
data class ToolResult(
    /** 是否成功 */
    val success: Boolean,
    
    /** 结果数据 */
    val output: Any? = null,
    
    /** 错误信息 */
    val error: String? = null,
    
    /** 错误码 */
    val errorCode: Int = 0,
    
    /** 执行耗时(毫秒) */
    val durationMs: Long = 0,
    
    /** 元数据 */
    val metadata: Map<String, Any> = emptyMap()
)
```

### 3.2 车控Tool示例

```kotlin
/**
 * 空调控制Tool
 */
class AirConditionTool : ITool {
    
    override fun getName() = "air_condition_control"
    
    override fun getDescription() = """
        控制车辆空调系统，支持以下操作：
        - 开关空调
        - 设置温度(16-32度)
        - 设置风速(1-7档)
        - 设置风向模式
        - 开启/关闭自动模式
    """.trimIndent()
    
    override fun getParameterSchema() = ToolSchema(
        parameters = listOf(
            ParameterDef(
                name = "action",
                type = ParamType.ENUM,
                description = "操作类型",
                enumValues = listOf("turn_on", "turn_off", "set_temperature", 
                    "set_fan_speed", "set_mode", "auto_on", "auto_off")
            ),
            ParameterDef(
                name = "temperature",
                type = ParamType.INTEGER,
                description = "目标温度，仅当action为set_temperature时需要",
                minimum = 16,
                maximum = 32
            ),
            ParameterDef(
                name = "fan_speed",
                type = ParamType.INTEGER,
                description = "风速档位，仅当action为set_fan_speed时需要",
                minimum = 1,
                maximum = 7
            ),
            ParameterDef(
                name = "mode",
                type = ParamType.ENUM,
                description = "风向模式",
                enumValues = listOf("face", "feet", "face_feet", "windshield")
            )
        ),
        required = listOf("action")
    )
    
    override fun getType() = ToolType.VEHICLE_CONTROL
    
    override suspend fun execute(parameters: Map<String, Any>): ToolResult {
        // 实现车控逻辑
    }
}
```

---

## 4. 车控服务AIDL接口

### 4.1 主服务接口

```aidl
// IVehicleService.aidl
package com.hzdongcheng.aiagent.vehicle;

import com.hzdongcheng.aiagent.vehicle.IAirConditionService;
import com.hzdongcheng.aiagent.vehicle.ISeatService;
import com.hzdongcheng.aiagent.vehicle.IWindowService;
import com.hzdongcheng.aiagent.vehicle.ILightService;
import com.hzdongcheng.aiagent.common.Result;

/**
 * 车辆控制主服务接口
 */
interface IVehicleService {
    
    /**
     * 获取空调控制服务
     * @return IBinder 空调服务Binder
     */
    IBinder getAirConditionService();
    
    /**
     * 获取座椅控制服务
     * @return IBinder 座椅服务Binder
     */
    IBinder getSeatService();
    
    /**
     * 获取车窗控制服务
     * @return IBinder 车窗服务Binder
     */
    IBinder getWindowService();
    
    /**
     * 获取灯光控制服务
     * @return IBinder 灯光服务Binder
     */
    IBinder getLightService();
    
    /**
     * 获取整车状态
     * @return Result 车辆状态JSON
     */
    Result getVehicleStatus();
    
    /**
     * 获取车辆基本信息
     * @return Result 车辆信息(VIN、里程等)
     */
    Result getVehicleInfo();
}
```

### 4.2 空调服务接口

```aidl
// IAirConditionService.aidl
package com.hzdongcheng.aiagent.vehicle;

import com.hzdongcheng.aiagent.common.Result;

/**
 * 空调控制服务接口
 */
interface IAirConditionService {
    
    /**
     * 打开空调
     * @return Result 操作结果
     */
    Result turnOn();
    
    /**
     * 关闭空调
     * @return Result 操作结果
     */
    Result turnOff();
    
    /**
     * 设置温度
     * @param temperature 目标温度(16-32)
     * @return Result 操作结果
     */
    Result setTemperature(int temperature);
    
    /**
     * 获取当前温度
     * @return Result 包含温度值
     */
    Result getTemperature();
    
    /**
     * 设置风速
     * @param level 风速档位(1-7)
     * @return Result 操作结果
     */
    Result setFanSpeed(int level);
    
    /**
     * 获取风速
     * @return Result 包含风速档位
     */
    Result getFanSpeed();
    
    /**
     * 设置风向模式
     * @param mode 0-吹脸 1-吹脚 2-吹脸+脚 3-吹玻璃
     * @return Result 操作结果
     */
    Result setAirFlowMode(int mode);
    
    /**
     * 设置自动模式
     * @param enabled 是否开启
     * @return Result 操作结果
     */
    Result setAutoMode(boolean enabled);
    
    /**
     * 设置AC开关
     * @param enabled 是否开启
     * @return Result 操作结果
     */
    Result setAcEnabled(boolean enabled);
    
    /**
     * 设置内循环
     * @param enabled 是否开启
     * @return Result 操作结果
     */
    Result setRecirculation(boolean enabled);
    
    /**
     * 前挡除雾
     * @return Result 操作结果
     */
    Result defrostFront();
    
    /**
     * 后挡除雾
     * @return Result 操作结果
     */
    Result defrostRear();
    
    /**
     * 获取空调完整状态
     * @return Result 空调状态JSON
     */
    Result getStatus();
}
```

### 4.3 座椅服务接口

```aidl
// ISeatService.aidl
package com.hzdongcheng.aiagent.vehicle;

import com.hzdongcheng.aiagent.common.Result;

/**
 * 座椅控制服务接口
 */
interface ISeatService {
    
    /**
     * 设置座椅位置
     * @param seatId 座椅ID: 0-主驾 1-副驾 2-后排左 3-后排右
     * @param position 位置(0-100)
     * @return Result 操作结果
     */
    Result setPosition(int seatId, int position);
    
    /**
     * 获取座椅位置
     * @param seatId 座椅ID
     * @return Result 包含位置值
     */
    Result getPosition(int seatId);
    
    /**
     * 设置靠背角度
     * @param seatId 座椅ID
     * @param angle 角度(0-100)
     * @return Result 操作结果
     */
    Result setBackrestAngle(int seatId, int angle);
    
    /**
     * 设置座椅高度
     * @param seatId 座椅ID
     * @param height 高度(0-100)
     * @return Result 操作结果
     */
    Result setHeight(int seatId, int height);
    
    /**
     * 设置座椅加热
     * @param seatId 座椅ID
     * @param level 档位: 0-关 1-低 2-中 3-高
     * @return Result 操作结果
     */
    Result setHeating(int seatId, int level);
    
    /**
     * 获取座椅加热状态
     * @param seatId 座椅ID
     * @return Result 包含加热档位
     */
    Result getHeatingStatus(int seatId);
    
    /**
     * 设置座椅通风
     * @param seatId 座椅ID
     * @param level 档位: 0-关 1-低 2-中 3-高
     * @return Result 操作结果
     */
    Result setVentilation(int seatId, int level);
    
    /**
     * 获取座椅通风状态
     * @param seatId 座椅ID
     * @return Result 包含通风档位
     */
    Result getVentilationStatus(int seatId);
    
    /**
     * 设置座椅按摩
     * @param seatId 座椅ID
     * @param mode 模式: 0-关 1-波浪 2-脉冲 3-舒缓
     * @param intensity 强度(1-3)
     * @return Result 操作结果
     */
    Result setMassage(int seatId, int mode, int intensity);
    
    /**
     * 设置腰部支撑
     * @param seatId 座椅ID
     * @param support 支撑程度(0-100)
     * @return Result 操作结果
     */
    Result setLumbarSupport(int seatId, int support);
    
    /**
     * 保存座椅记忆
     * @param seatId 座椅ID
     * @param memorySlot 记忆槽位(1-3)
     * @return Result 操作结果
     */
    Result saveMemory(int seatId, int memorySlot);
    
    /**
     * 恢复座椅记忆
     * @param seatId 座椅ID
     * @param memorySlot 记忆槽位(1-3)
     * @return Result 操作结果
     */
    Result restoreMemory(int seatId, int memorySlot);
    
    /**
     * 设置迎宾模式
     * @param seatId 座椅ID
     * @param enabled 是否开启
     * @return Result 操作结果
     */
    Result setWelcomeMode(int seatId, boolean enabled);
    
    /**
     * 获取座椅完整状态
     * @param seatId 座椅ID
     * @return Result 座椅状态JSON
     */
    Result getStatus(int seatId);
}
```

### 4.4 通用结果定义

```aidl
// Result.aidl
package com.hzdongcheng.aiagent.common;

/**
 * 通用操作结果
 */
parcelable Result {
    /** 结果码: 0-成功, 其他-失败 */
    int code;
    
    /** 结果消息 */
    String message;
    
    /** 结果数据(JSON格式) */
    String data;
}
```

---

## 5. 云服务API接口规范

### 5.1 通用规范

#### 5.1.1 请求格式

```
Base URL: https://api.example.com/v1
Content-Type: application/json
Authorization: Bearer {access_token}

请求头:
- X-Request-ID: 请求唯一ID
- X-Device-ID: 设备ID
- X-App-Version: 应用版本
```

#### 5.1.2 响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": { ... },
  "timestamp": 1705190400000,
  "request_id": "xxx"
}
```

#### 5.1.3 错误码定义

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 1001 | 参数错误 |
| 1002 | 认证失败 |
| 1003 | 权限不足 |
| 2001 | 服务内部错误 |
| 2002 | 服务不可用 |
| 3001 | 业务逻辑错误 |

### 5.2 天气服务API

#### 5.2.1 实时天气查询

```
GET /weather/realtime

请求参数:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| location | string | 是 | 位置，支持城市名或经纬度 |
| lang | string | 否 | 语言，默认zh |

请求示例:
GET /weather/realtime?location=杭州

响应示例:
{
  "code": 0,
  "message": "success",
  "data": {
    "location": {
      "name": "杭州",
      "province": "浙江",
      "country": "中国",
      "lat": 30.2741,
      "lon": 120.1551
    },
    "now": {
      "temperature": 25,
      "feels_like": 27,
      "humidity": 65,
      "weather": "晴",
      "weather_code": "100",
      "wind_direction": "东南风",
      "wind_speed": "3级",
      "wind_scale": 3,
      "visibility": 10,
      "pressure": 1013,
      "uv_index": 5,
      "aqi": 45,
      "aqi_level": "优"
    },
    "update_time": "2026-01-14T10:30:00+08:00"
  }
}
```

#### 5.2.2 天气预报查询

```
GET /weather/forecast

请求参数:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| location | string | 是 | 位置 |
| days | int | 否 | 预报天数(1-7)，默认3 |

请求示例:
GET /weather/forecast?location=杭州&days=3

响应示例:
{
  "code": 0,
  "message": "success",
  "data": {
    "location": { ... },
    "forecasts": [
      {
        "date": "2026-01-14",
        "day_weather": "晴",
        "day_weather_code": "100",
        "night_weather": "多云",
        "night_weather_code": "101",
        "high_temp": 28,
        "low_temp": 18,
        "humidity": 60,
        "wind_direction": "东南风",
        "wind_scale": "2-3级",
        "sunrise": "06:45",
        "sunset": "17:30",
        "uv_index": 5,
        "aqi": 50
      },
      ...
    ]
  }
}
```

#### 5.2.3 气象预警查询

```
GET /weather/alert

请求参数:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| location | string | 是 | 位置 |

响应示例:
{
  "code": 0,
  "data": {
    "alerts": [
      {
        "alert_id": "xxx",
        "title": "高温黄色预警",
        "type": "high_temperature",
        "level": "yellow",
        "description": "预计今日最高气温将达38度...",
        "start_time": "2026-01-14T08:00:00+08:00",
        "end_time": "2026-01-14T18:00:00+08:00",
        "source": "杭州市气象台"
      }
    ]
  }
}
```

### 5.3 咖啡订购API

#### 5.3.1 菜单查询

```
GET /coffee/menu

请求参数:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| store_id | string | 否 | 门店ID，获取特定门店菜单 |
| category | string | 否 | 分类筛选 |

响应示例:
{
  "code": 0,
  "data": {
    "categories": [
      {
        "id": "espresso",
        "name": "浓缩咖啡",
        "items": [
          {
            "item_id": "LATTE001",
            "name": "拿铁",
            "description": "经典意式拿铁，醇香牛奶与浓缩咖啡的完美结合",
            "price": 32.00,
            "image_url": "https://...",
            "sizes": [
              {"size": "中杯", "price_diff": 0},
              {"size": "大杯", "price_diff": 4}
            ],
            "options": [
              {
                "name": "温度",
                "values": ["热", "冰", "温"]
              },
              {
                "name": "糖度",
                "values": ["标准", "少糖", "无糖"]
              },
              {
                "name": "奶类",
                "values": ["全脂牛奶", "脱脂牛奶", "燕麦奶"]
              }
            ],
            "available": true
          }
        ]
      }
    ],
    "recommendations": [
      {
        "item_id": "LATTE001",
        "reason": "您的常点饮品"
      }
    ]
  }
}
```

#### 5.3.2 门店查询

```
GET /coffee/stores

请求参数:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| lat | double | 是 | 纬度 |
| lon | double | 是 | 经度 |
| radius | int | 否 | 搜索半径(米)，默认5000 |
| limit | int | 否 | 返回数量，默认10 |

响应示例:
{
  "code": 0,
  "data": {
    "stores": [
      {
        "store_id": "HZ001",
        "name": "星巴克(西湖银泰店)",
        "address": "杭州市上城区延安路98号西湖银泰1楼",
        "lat": 30.2505,
        "lon": 120.1699,
        "distance": 1200,
        "phone": "0571-88888888",
        "business_hours": "08:00-22:00",
        "status": "营业中",
        "services": ["堂食", "外带", "车载取餐"],
        "average_wait_time": 10
      }
    ]
  }
}
```

#### 5.3.3 创建订单

```
POST /coffee/orders

请求Body:
{
  "store_id": "HZ001",
  "items": [
    {
      "item_id": "LATTE001",
      "quantity": 1,
      "size": "大杯",
      "options": {
        "温度": "热",
        "糖度": "少糖",
        "奶类": "燕麦奶"
      },
      "note": "不要太烫"
    }
  ],
  "pickup_type": "car_pickup",
  "pickup_time": "2026-01-14T10:30:00+08:00",
  "vehicle_info": {
    "plate_number": "浙A12345",
    "color": "白色",
    "model": "Model 3"
  },
  "coupon_id": "xxx"
}

响应示例:
{
  "code": 0,
  "data": {
    "order_id": "ORD20260114001",
    "status": "pending_payment",
    "items": [...],
    "subtotal": 36.00,
    "discount": 5.00,
    "total_amount": 31.00,
    "payment_url": "https://pay.example.com/xxx",
    "estimated_ready_time": "2026-01-14T10:25:00+08:00",
    "created_at": "2026-01-14T10:15:00+08:00"
  }
}
```

#### 5.3.4 查询订单状态

```
GET /coffee/orders/{order_id}

响应示例:
{
  "code": 0,
  "data": {
    "order_id": "ORD20260114001",
    "status": "preparing",
    "status_text": "制作中",
    "items": [...],
    "total_amount": 31.00,
    "store": {
      "store_id": "HZ001",
      "name": "星巴克(西湖银泰店)",
      "address": "..."
    },
    "pickup_type": "car_pickup",
    "pickup_code": "A088",
    "estimated_ready_time": "2026-01-14T10:25:00+08:00",
    "timeline": [
      {"status": "created", "time": "10:15", "text": "订单已创建"},
      {"status": "paid", "time": "10:16", "text": "支付成功"},
      {"status": "preparing", "time": "10:18", "text": "开始制作"}
    ]
  }
}
```

#### 5.3.5 订单状态枚举

| 状态 | 说明 |
|------|------|
| pending_payment | 待支付 |
| paid | 已支付 |
| confirmed | 已确认 |
| preparing | 制作中 |
| ready | 已完成，待取餐 |
| picked_up | 已取餐 |
| cancelled | 已取消 |
| refunded | 已退款 |

---

## 6. Agent间通信协议

### 6.1 消息格式

```kotlin
/**
 * Agent间通信消息
 */
data class AgentMessage(
    /** 消息ID */
    val messageId: String,
    
    /** 消息类型 */
    val type: MessageType,
    
    /** 发送方Agent ID */
    val sourceAgentId: String,
    
    /** 接收方Agent ID */
    val targetAgentId: String,
    
    /** 会话ID */
    val sessionId: String,
    
    /** 消息优先级 */
    val priority: Priority,
    
    /** 消息载荷 */
    val payload: MessagePayload,
    
    /** 时间戳 */
    val timestamp: Long,
    
    /** 超时时间(毫秒) */
    val timeoutMs: Long,
    
    /** 是否需要响应 */
    val requireResponse: Boolean = true
)

/**
 * 消息类型
 */
enum class MessageType {
    TASK_REQUEST,    // 任务请求
    TASK_RESPONSE,   // 任务响应
    EVENT,           // 事件通知
    QUERY,           // 状态查询
    BROADCAST        // 广播消息
}

/**
 * 消息载荷
 */
data class MessagePayload(
    /** 意图/动作 */
    val action: String,
    
    /** 参数 */
    val parameters: Map<String, Any>,
    
    /** 上下文 */
    val context: Map<String, Any>? = null,
    
    /** 数据 */
    val data: Any? = null
)
```

### 6.2 示例场景

**场景: 用户说"查下天气，如果热就开空调"**

```
消息1: Master → Weather Agent
{
  "messageId": "msg_001",
  "type": "TASK_REQUEST",
  "sourceAgentId": "master_agent",
  "targetAgentId": "weather_agent",
  "payload": {
    "action": "query_weather",
    "parameters": {
      "location": "当前位置"
    }
  }
}

消息2: Weather Agent → Master
{
  "messageId": "msg_002",
  "type": "TASK_RESPONSE",
  "sourceAgentId": "weather_agent",
  "targetAgentId": "master_agent",
  "payload": {
    "action": "query_weather_result",
    "data": {
      "temperature": 35,
      "weather": "晴"
    }
  }
}

消息3: Master → Vehicle Agent (基于条件判断)
{
  "messageId": "msg_003",
  "type": "TASK_REQUEST",
  "sourceAgentId": "master_agent",
  "targetAgentId": "vehicle_agent",
  "payload": {
    "action": "control_ac",
    "parameters": {
      "operation": "turn_on",
      "temperature": 24
    }
  }
}

消息4: Vehicle Agent → Master
{
  "messageId": "msg_004",
  "type": "TASK_RESPONSE",
  "sourceAgentId": "vehicle_agent",
  "targetAgentId": "master_agent",
  "payload": {
    "action": "control_ac_result",
    "data": {
      "success": true,
      "message": "空调已开启，温度设为24度"
    }
  }
}
```

---

## 7. 版本历史

| 版本 | 日期 | 修改内容 |
|------|------|----------|
| V1.0 | 2026-01-14 | 初始版本 |
