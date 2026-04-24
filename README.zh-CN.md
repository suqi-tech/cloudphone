# CloudPhone Plugin for OpenClaw

[English](./README.md)

OpenClaw 云手机插件，让 AI Agent 通过自然语言即可完成云手机自动化操作。

只需一条指令，Agent 就能把任务提交给后端 AI Agent，后端负责完整的执行闭环（截图观察、LLM 规划、UI 操作），并将结果实时流式返回。

## 快速开始

### 1. 安装插件

```bash
openclaw plugins install @suqiai/cloudphone
```

后续如需更新插件，可执行：

```bash
openclaw plugins update @suqiai/cloudphone
```

### 2. 配置插件

只需在 `plugins.entries.cloudphone.config` 中填写 **`apikey`**，其余可选项由插件内置默认值覆盖。若需要为云手机自动化 Agent（后端）指定**默认 LLM 提供商**，可额外填写可选的 **`llmApiKey`**、**`llmBaseUrl`**。

#### 方式一：配置文件（openclaw.json）

在 `openclaw.json` 中添加以下配置：

- **apikey**：在 [https://ai.suqi.tech](https://ai.suqi.tech) 登录或注册后，在账户/设置中获取 API Key。

```json
{
  "plugins": {
    "entries": {
      "cloudphone": {
        "enabled": true,
        "config": {
          "apikey": "你可以在该网站的用户中心获取 API 密钥"
        }
      }
    }
  }
}
```

可选 — 自动化默认 LLM 凭证（若后端已自带 LLM，可省略）：

```json
{
  "plugins": {
    "entries": {
      "cloudphone": {
        "enabled": true,
        "config": {
          "apikey": "你的 CloudPhone apikey",
          "llmApiKey": "your-zai-api-key",
          "llmBaseUrl": "https://open.bigmodel.cn/api/paas/v4"
        }
      }
    }
  }
}
```

#### 方式二：OpenClaw 控制台 UI

1. 在浏览器中打开 OpenClaw 控制台。
2. 进入「插件」相关页面，找到 **CloudPhone** 并启用。
3. 填写 **apikey**（在 [https://ai.suqi.tech](https://ai.suqi.tech) 登录或注册后，于账户/设置中获取）。
4. 如需插件级默认 LLM，可在表单中填写 **LLM API Key** 与 **LLM Base URL**。若使用 Z.AI，可参考 [Z.AI API 文档介绍页](https://docs.bigmodel.cn/cn/api/introduction) 申请 API Key。


### 3. 重启 Gateway

```bash
openclaw gateway restart
```

## 工作原理

本插件将云手机后端 AI Agent 能力封装为三个高层工具：

1. **`cloudphone_execute`** — 将自然语言指令提交给后端。后端负责 LLM 语义解析、云手机 UI 自动化（观察 → 规划 → 操作 闭环）。立即返回 `task_id`。

2. **`cloudphone_execute_and_wait`** — 自动串联调用：先执行 `cloudphone_execute`，再自动触发一次 `cloudphone_task_result`，返回首个 10 秒轮询窗口结果。

3. **`cloudphone_task_result`** — 订阅任务的 SSE 流；每次调用消费一个 10 秒窗口并返回该窗口内的 `thinking` 增量，直到终态。

Agent 不再需要直接控制 UI 坐标、管理截图或逐一调用 tap/swipe/input 等工具。后端 AI Agent 处理完整的自动化闭环。

## 配置说明

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `apikey` | string | 是 | — | Authorization 鉴权凭证（ApiKey） |
| `llmApiKey` | string | 否 | — | 云手机自动化所用默认 LLM 提供商 API Key（敏感；不需要可不填）。若使用 Z.AI，可从 [Z.AI API 文档介绍页](https://docs.bigmodel.cn/cn/api/introduction) 获取。 |
| `llmBaseUrl` | string | 否 | — | 云手机自动化所用默认 LLM 提供商 Base URL。Z.AI 示例：`https://open.bigmodel.cn/api/paas/v4`。 |

> `apikey` 请在 [https://ai.suqi.tech](https://ai.suqi.tech) 登录或注册后，在账户/设置中获取。

可选字段 `baseUrl`、`timeout`、`llmApiKey`、`llmBaseUrl` 的完整说明见 `openclaw.plugin.json`。`baseUrl`、`timeout` 省略时使用内置默认值；LLM 相关字段默认不配置，按需填写。

当使用 Z.AI 作为 LLM 提供商时，建议配置：
- `llmApiKey`：你的 Z.AI API Key
- `llmBaseUrl`：`https://open.bigmodel.cn/api/paas/v4`

## 工具一览

插件安装后，Agent 将自动获得以下工具能力：

### 用户与设备管理

| 工具 | 说明 |
|------|------|
| `cloudphone_get_user_profile` | 获取当前用户基本信息 |
| `cloudphone_list_devices` | 获取云手机设备列表，支持分页、关键字搜索和状态筛选 |
| `cloudphone_get_device_info` | 获取指定设备的详细信息 |
| `cloudphone_get_device_screenshot_url` | 按 `device_id` 获取最新截图 URL（默认可用；仅用户触发） |
| `cloudphone_create_share_link` | 按 `device_id` 生成设备串流分享链接（默认可用；仅用户触发） |

### AI Agent 任务执行

| 工具 | 说明 |
|------|------|
| `cloudphone_execute` | 提交自然语言指令，立即返回 task_id |
| `cloudphone_execute_and_wait` | 自动串联 execute + 首次 task_result 轮询 |
| `cloudphone_task_result` | 每次返回 10 秒窗口内的思考增量与当前状态 |

## 使用示例

安装配置完成后，可以直接通过自然语言对话操控云手机。

### 执行 UI 自动化任务

> 在云手机上打开微信，搜索"OpenClaw"公众号并关注

Agent 会：
1. 调用 `cloudphone_list_devices` 获取设备 ID
2. 调用 `cloudphone_execute_and_wait` 提交指令并自动触发首次结果轮询
3. 若状态为 `running`，继续每 10 秒调用一次 `cloudphone_task_result`，直到 `success`/`done`/`error`

### 查看设备列表

> 帮我看看我有哪些云手机

Agent 会调用 `cloudphone_list_devices` 返回设备列表。

### 提交任务并等待完成

```text
Agent: cloudphone_execute_and_wait
  instruction: "打开抖音，搜索美食视频并点赞第一条"
  device_id: "abc123"
→ 返回: { ok: false, task_result: { status: "running", thinking: [...] } }

Agent: cloudphone_task_result
  task_id: 42
→ 10 秒窗口增量，直到终态: { ok: true, status: "done", result: {...} }
```

## 工具参数详解

### `cloudphone_execute`

```text
instruction    : string  - 自然语言任务指令（必填）
device_id      : string  - 设备唯一 ID（推荐）
user_device_id : number  - 用户设备 ID（兼容字段，device_id 优先）
session_id     : string  - 可选会话 ID，用于流式内容持久化
lang           : string  - 语言提示："cn"（默认）或 "en"
api_key        : string  - 可选 LLM 提供商 API Key；若传入则覆盖插件级 llmApiKey
base_url       : string  - 可选 LLM 提供商 Base URL；若传入则覆盖插件级 llmBaseUrl
max_steps      : integer - 可选，云手机 Agent 单任务最大步骤数（取值范围 1-200）。
                           未传入时按 "插件级 maxSteps > 内置默认 50" 的顺序回落。
```

**`cloudphone_execute_and_wait`** 使用相同参数定义（同一套 schema）。

```text
task_id    : number - cloudphone_execute 返回的任务 ID（必填）
```

**返回字段：**

```text
ok         : boolean  - 操作是否成功
task_id    : number   - 输入的任务 ID 回显
status     : string   - "done" | "success" | "error" | "timeout"
thinking   : string[] - 当前 10 秒窗口内新增的 Agent 思考步骤（增量）
result     : object   - 后端返回的最终任务结果
message    : string   - 错误信息（status 为 "error" 或 "timeout" 时）
```

### `cloudphone_list_devices`

```text
keyword : string  - 搜索关键字（设备名称或设备 ID）
status  : string  - 状态筛选："online" | "offline"
page    : integer - 页码，默认 1
size    : integer - 每页条数，默认 20
```

### `cloudphone_get_device_info`

```text
device_id : string - 设备唯一 ID（32 位 hex 不透明 ID，必填）
```

### `cloudphone_get_device_screenshot_url`

```text
device_id : string - 设备唯一 ID（必填）
```

说明：
- 插件安装后该工具默认可用，无需额外白名单开启。
- 仅在用户明确要求获取截图 URL 时调用，禁止自主触发。
- 返回的 `screenshot_url` 为上游原样透传，应视为敏感的临时凭证链接。

### `cloudphone_create_share_link`

```text
device_id : string - 设备唯一 ID（32 位 hex 不透明 ID，必填）。
                     通常来自 cloudphone_list_devices 返回值中的 device_id 字段。
```

**返回字段：**

```text
ok        : boolean - 操作是否成功
device_id : string  - 输入的 device_id 回显
share_url : string  - 设备串流分享链接（带签名，敏感凭证；成功时返回）
code      : string  - 失败时的错误码（INVALID_PARAMS / HTTP_ERROR / INVALID_UPSTREAM_PAYLOAD 等）
message   : string  - 失败时的错误信息
```

说明：
- 插件安装后该工具默认可用，无需额外白名单开启。
- 仅在用户明确要求生成/获取分享链接时调用，禁止自主触发。
- 返回的 `share_url` 为上游原样透传（含签名查询串），应视为敏感凭证链接：时效有限、禁止对外二次转发，也不应完整出现在日志中。
- 入参 `device_id` 为 32 位 hex 不透明标识（非十进制数值），不存在长整型精度问题。
- 请求体字段名与入参一致（`device_id`，snake_case），内部不做字段名或数值转换。
- **后端契约**：需要后端 `/openapi/v1/devices/create/share/link` 与 `/openapi/v1/devices/info` 已完成"接受 `device_id`"的改造；未升级的后端会返回业务错误。

### 使用示例：获取设备分享链接

> 帮我把设备 `xxxxxx` 生成一个分享链接（`xxxxxx` 为设备的 32 位 hex `device_id`）

Agent 会：
1. 识别到用户"显式请求分享链接"
2. 调用 `cloudphone_create_share_link`，`device_id: "a1b2c3d4e5f67890a1b2c3d4e5f67890"`
3. 将返回的 `share_url` 回填到聊天框

若用户未提供具体设备 ID，Agent 可先调用 `cloudphone_list_devices` 或 `cloudphone_get_device_info` 协助用户确认目标设备（以 `device_id` 为键）后再生成分享链接。

## 常见问题

**Q: 安装后 Agent 找不到云手机工具？**

确认 `plugins.entries.cloudphone.enabled` 设置为 `true`，然后重启 Gateway。

**Q: `cloudphone_task_result` 为什么返回 `running`？**

这是正常行为，表示当前 10 秒轮询窗口未到终态。请继续每 10 秒调用一次 `cloudphone_task_result`，直到 `success`/`done`/`error`。

**Q: 调用工具报鉴权失败或请求错误？**

- 检查 `apikey` 是否有效，修改配置后是否已重启 Gateway
- 检查本机网络与云手机服务是否可达
- `401` 错误通常表示 `apikey` 无效或已过期

**Q: 如何获取 `apikey`？**

请在 [https://ai.suqi.tech](https://ai.suqi.tech) 登录或注册后，在账户/设置中获取 API Key。

**Q: `cloudphone_execute` 支持并发任务吗？**

同一 agent 上下文不支持并发。插件会按 agent key（优先 `session_id`，其次 `device_id`，再其次 `user_device_id`，最后 default）强制串行执行。  
如果上一个任务还未在 `cloudphone_task_result` 到达终态，你再次调用 `cloudphone_execute` 会返回 `code: "AGENT_BUSY"`，并携带 `blocking_task_id`。

推荐调用顺序：

1. `cloudphone_execute_and_wait`（自动触发首次轮询）
2. `cloudphone_task_result`（若返回 `running`，继续轮询到终态：`success`/`done`/`error`）
3. 再次 `cloudphone_execute`

## 更新日志

当前版本：**v2026.4.24**

### v2026.4.24

- 新增 Agent 工具 `cloudphone_create_share_link`，按 `device_id` 生成设备带签名的串流分享链接（默认可用；仅在用户显式请求时调用）
- `cloudphone_get_device_info` 入参由 `user_device_id`（number）切换为 `device_id`（32 位 hex 字符串），与插件其他工具统一的不透明设备标识保持一致
- 引入 `json-bigint` 作为全插件统一的上游响应 / SSE 事件（`agent_thinking` / `task_result` / `error`）JSON 解析器，避免 19 位雪花 ID 等 long 字段在解析过程中被转为 `Number` 导致精度丢失
- 强化 `normalizeTaskId`：同时兼容 `string` 与 `number` 输入，对超过安全整数区间的数字字符串显式拒绝，避免静默截断
- 为所有上游响应增加统一的防御性 JSON 解析错误分支，负载解析失败时返回结构化错误而非抛异常
- 新增依赖 `json-bigint` 及其类型声明 `@types/json-bigint`
- 同步 package/plugin/doc 的版本标识到 `v2026.4.24`

### v2026.4.20

- 新增可选插件配置 `maxSteps`（取值范围 1-200，默认 50），用于限制云手机 Agent 单任务最大步骤数
- 为 `cloudphone_execute` / `cloudphone_execute_and_wait` 增加可选参数 `max_steps`（范围 1-200），按调用粒度覆盖插件级 `maxSteps`
- 按 "调用入参 > 插件配置 > 内置默认 50" 的优先级解析最终生效的 `max_steps`，并始终透传至后端请求体
- 在 `cloudphone_execute` 的启动日志中附带当次生效的 `max_steps`，便于排查
- 同步 package/plugin/doc 的版本标识到 `v2026.4.20`

### v2026.4.14001

- 补充插件配置说明：支持可选默认 LLM 提供商字段 `llmApiKey`、`llmBaseUrl`
- 增加 OpenClaw 控制台可选 LLM 配置说明，并补充 Z.AI API 文档参考链接
- 对齐 `cloudphone_execute_and_wait` 与 `cloudphone_execute` 的参数定义，明确支持 `api_key`、`base_url` 覆盖
- 同步 package/plugin/doc 的版本标识到 `v2026.4.14001`

### v2026.4.14

- 新增可选插件配置 `llmApiKey`、`llmBaseUrl`，用于云手机自动化默认使用的 LLM 提供商凭证与地址
- 为 `cloudphone_execute` 增加可选参数 `api_key`、`base_url`（可覆盖插件级配置），并随请求体转发至后端
- 以 `v2026.4.14` 重新发版（取代误打的 `v2026.4.4` 标签）
- 同步 package/plugin/doc 的版本标识到 `v2026.4.14`

### v2026.4.3

- 新增 `cloudphone_get_device_screenshot_url`（默认可用；仅在用户明确要求时调用），对接 `POST /openapi/v1/devices/snapshot`
- 在日志与 MCP 摘要中对 `screenshot_url` 去除查询串，工具返回的 JSON 仍保留完整 URL
- 在 README 中补充该工具的参数说明、隐私与调用约束
- 为截图 URL 成功与上游错误路径补充 Node 测试；调整 `tsconfig` 对 `*.test.ts` 的 include/exclude
- 同步 package/plugin/doc 的版本标识到 `v2026.4.3`

### v2026.4.2

- 按设备/会话串行执行任务：`cloudphone_execute` 在已有任务未完成、`cloudphone_task_result` 未达终态时返回 `AGENT_BUSY`
- 改进 SSE 解析，兼容标准 `event:`/`data:` 分帧与后端将事件类型写在 JSON 内的格式
- 强化工具描述中的约束说明（禁止擅自加步骤、禁止仅截图类指令、限制重试次数等）
- npm 包作用域更名为 `@suqiai/cloudphone`（提交 `3c50f95`）
- 新增 `src/tools.serial-gating.test.ts`（Node 自带 test）；`tsconfig` 排除 `*.test.ts`，避免测试产物进入 `dist/`
- 更新内置 skill 与 README 中对「执行 → 轮询」流程的说明
- 同步 package/plugin/doc 的版本标识到 `v2026.4.2`

### v2026.4.1

- 新增 `cloudphone_execute_and_wait`，自动串联任务提交与首次结果轮询
- 明确任务提交、轮询与调用顺序的工具说明文档
- 在 `.gitignore` 中新增 `docs/` 与 `openspec/`，便于项目管理
- 同步 package/plugin/doc 的版本标识到 `v2026.4.1`

### v2026.3.31

- 增强插件工具中的任务执行与结果处理流程
- 完善内置 skill 的任务相关文档与参考示例
- 同步 package/plugin/doc 的版本标识到 `v2026.3.31`

### v2026.3.30

- 移除 12 个细粒度 UI 自动化工具（tap、swipe、snapshot 等），改由后端 AI Agent 统一处理
- 新增 `cloudphone_execute`：将自然语言指令提交给后端 AI Agent
- 新增 `cloudphone_task_result`：通过 SSE 流式获取 Agent 思考过程和最终结果
- 移除 AutoGLM 直接集成（后端现在负责完整的观察 → 规划 → 操作 闭环）
- 精简插件配置：移除所有 `autoglm*` 字段，仅保留 `apikey`、`baseUrl`、`timeout`
- 同步更新 skills、README 和工具参考文档

### v2026.3.27

- 基于目标提交 `1da1031` 汇总并对齐发布说明
- 同步 package/plugin/doc 的版本标识到 `v2026.3.27`

### v1.1.0

- 增强 `cloudphone_render_image` 的截图渲染处理，提升不同宿主中的兼容性
- 新增 `cloudphone-snapshot-url` skill

### v1.0.6

- 新增随插件发布的内置 skill：`basic-skill`
- 新增 `reference.md` 参数速查表

## 许可证

本插件遵循项目所在仓库的许可协议。
