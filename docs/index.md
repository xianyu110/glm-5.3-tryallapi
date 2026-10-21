# GLM-5.3 API 调用教程：用 TryAllAPI 做 OpenAI / Anthropic 兼容接入

> **GLM-5.3**（模型 ID：`glm-5.3`）是智谱的旗舰对话模型，价格接口标签为「对话 / 工具」，端点类型 `openai` + `anthropic`。本文记录如何通过聚合站 **[TryAllAPI](https://tryallapi.com/)**，从模型广场 / 价格页选型、读官方价与分组倍率，到 `POST /v1/chat/completions`（以及 Anthropic 格式 `/v1/messages`）的完整接入流程。价格与分组倍率取数 **2026-10-21（北京）**，**以页面实时为准**；示例仅为请求形态说明，未跑通真实长任务计费。

**Keywords:** GLM-5.3 · glm-5.3 · 智谱 · ChatGLM · TryAllAPI · OpenAI 兼容 · `/v1/chat/completions` · Anthropic `/v1/messages` · Claude Code

---

## 目录

- [为什么用聚合站调 GLM](#一为什么用聚合站调-glm)
- [在价格页 / 模型广场找到 GLM-5.3](#二在价格页--模型广场找到-glm-53)
- [模型怎么选：5.3 与相邻 GLM 档](#三模型怎么选53-与相邻-glm-档)
- [价格怎么读](#四价格怎么读)
- [调用流程：Key → Base URL → chat/completions](#五调用流程key--base-url--chatcompletions)
- [curl / SDK 最小示例](#六curl--sdk-最小示例)
- [常见注意](#七常见注意)
- [FAQ](#faq)
- [小结与试用入口](#八小结与试用入口)

---

## 一、为什么用聚合站调 GLM

只用 GLM、需要 GLM Coding Plan 套餐或智谱官方发票，直连智谱开放平台即可（OpenAI 兼容 `https://open.bigmodel.cn/api/paas/v4`，Anthropic 兼容 `https://open.bigmodel.cn/api/anthropic`）。如果同一个项目里还要调 Claude、GPT、DeepSeek，希望一个 Key、一份账单，聚合站更顺手：

1. **统一鉴权**：一个 Key，切换 `model` 字段即可；
2. **两种协议**：OpenAI 兼容 `https://tryallapi.com/v1`；Anthropic SDK / Claude Code 用 `https://tryallapi.com`；
3. **模型广场可对照**：标签、分组倍率、端点类型一目了然。

注册试用入口：[https://tryallapi.com/register?aff=5A6A](https://tryallapi.com/register?aff=5A6A)。

---

## 二、在价格页 / 模型广场找到 GLM-5.3

打开 [价格页](https://tryallapi.com/pricing) 或模型广场，按供应商 **ChatGLM**、类型 **对话**、关键词 `glm-5.3` 筛选。详情卡截图如下：

![GLM-5.3 模型广场详情卡：模型 ID、API 与分组价格](model-square.png)

详情卡要点（取自价格接口 2026-10-21 快照与文档站）：

| 字段 | 内容 |
| --- | --- |
| **模型 ID** | `glm-5.3`（小写） |
| **供应商** | ChatGLM（智谱） |
| **标签** | 对话、工具 |
| **端点类型** | `openai`（`POST /v1/chat/completions`）、`anthropic`（`POST /v1/messages`） |
| **可用分组** | Self-Deployed-2、Self-Deployed-3、Alibaba-3、glm-1 |
| **上下文 / 最大输出** | 1M tokens / 128K tokens（文档站引智谱模型页） |
| **输入模态** | 仅文本 |
| **思考** | 始终开启，仅支持 `thinking.type: "enabled"`；`reasoning_effort` 可选 `low` / `high` / `max`，默认 `max` |

广场描述：与 GLM-5.2 使用相同基础模型，所有提升来自后训练，强化复杂软件工程与 Agent 任务。完整参数卡见文档站：[GLM-5.3 API 指南](https://docs.tryallapi.com/glm-5.3-api/)。

---

## 三、模型怎么选：5.3 与相邻 GLM 档

只根据价格接口 / 文档站可见信息做对照，**不编造未出现的能力差异**。基价按价格接口 `model_ratio` / `completion_ratio` 换算（$/1M，未乘分组倍率）：

| 模型 ID（可见） | 标签 | 基价 输入 / 输出 | 怎么理解 |
| --- | --- | --- | --- |
| **`glm-5.3`** | 对话,工具 | $1.40 / $4.40 | 本文主线：编程与长程 Agent |
| **`glm-5.3-flash`** | 对话,工具,识图 | $0.15 / $0.50 | 需要图片输入时的同代选择（文档站：5.3 仅文本，图片改用 Flash 系列） |
| **`glm-5.2`** | 对话,工具 | $1.40 / $4.40 | 同基础模型的上一版；思考可关，迁移时注意差异 |
| **`glm-5.1`** | 对话,思考 | $1.40 / $4.40 | 官方国内价按 32K 输入分档 |

实务上：默认锁 `glm-5.3`；要图片输入就换 `glm-5.3-flash`；切换时只改 `model` 字符串。

---

## 四、价格怎么读

常见公式：**≈ 基价 × 分组倍率**（以控制台为准）。取数快照：**2026-10-21 北京**；**以页面实时为准**。

### 官方价（每百万 tokens，文档站核对）

| 计费项 | 国内（智谱开放平台） | 国际（Z.ai） |
| --- | --- | --- |
| 输入 | **¥8** | **$1.4** |
| 输出 | **¥28** | **$4.4** |
| 缓存命中 | **¥2** | **$0.26** |
| 缓存存储 | 限时免费 | 限时免费 |

1M 上下文内统一价，不按输入长度分档。来源：[docs.tryallapi.com/glm-5.3-api](https://docs.tryallapi.com/glm-5.3-api/)。价格接口中 `glm-5.3` 为 `model_ratio=0.7`、`completion_ratio≈3.143`、`cache_ratio=0.186`，换算基价输入 $1.40、输出 $4.40、缓存读 ≈ $0.26，与 Z.ai 国际价一致。

### 分组价格估算（基价 × group_ratio，$/1M）

| 分组 | group_ratio | 输入估 | 缓存读估 | 输出估 |
| --- | --- | --- | --- | --- |
| **Self-Deployed-2** | 1 | $1.40 | ≈ $0.26 | $4.40 |
| **Self-Deployed-3** | 1.5 | $2.10 | ≈ $0.39 | $6.60 |
| **Alibaba-3** | 2.2 | $3.08 | ≈ $0.573 | $9.68 |
| **glm-1** | 2.2 | $3.08 | ≈ $0.573 | $9.68 |

思考强制开启，思考 token 计入输出；轻任务用 `reasoning_effort: "low"` 控成本。**最终扣费以控制台流水为准**。

---

## 五、调用流程：Key → Base URL → chat/completions

```text
控制台创建 API Key（选 Self-Deployed / Alibaba / glm 分组）
  → OpenAI 兼容：Base URL = https://tryallapi.com/v1
      POST /v1/chat/completions   （model=glm-5.3）
  → Anthropic 兼容：Base URL = https://tryallapi.com（不带 /v1）
      POST /v1/messages           （model=glm-5.3）
```

建议：

- **thinking / reasoning_effort**：经 OpenAI SDK 用 `extra_body` 透传；`thinking.type` 只能是 `enabled`，`reasoning_effort` 只接受 `low` / `high` / `max`；
- **采样**：官方迁移指南默认 `temperature=1.0`、`top_p=0.95`，建议两者只调一个；
- **密钥**：只放环境变量（如 `TRYALLAPI_KEY`），勿提交仓库。

---

## 六、curl / SDK 最小示例

占位 Key `sk-xxx`（**不要**填入真实 Key）。

### 1）OpenAI 兼容 chat/completions

```bash
curl -sS https://tryallapi.com/v1/chat/completions \
  -H 'Authorization: Bearer sk-xxx' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "glm-5.3",
    "messages": [
      {"role": "user", "content": "这个 Go 服务的 goroutine 泄漏可能出在哪里？"}
    ],
    "thinking": {"type": "enabled"},
    "reasoning_effort": "high"
  }'
```

### 2）Python（OpenAI SDK）

```python
from openai import OpenAI

client = OpenAI(api_key="sk-xxx", base_url="https://tryallapi.com/v1")

r = client.chat.completions.create(
    model="glm-5.3",
    messages=[{"role": "user", "content": "给这个函数补上错误处理并解释改动"}],
    max_tokens=32768,  # 最大 128K，思考也会消耗输出
    extra_body={
        "thinking": {"type": "enabled"},   # GLM-5.3 只能是 enabled
        "reasoning_effort": "low",         # low / high / max，默认 max
    },
)
msg = r.choices[0].message
print(getattr(msg, "reasoning_content", None))  # 思考内容
print(msg.content)                              # 正式回答
```

### 3）Anthropic 格式（文档站示例形态）

```bash
curl -sS https://tryallapi.com/v1/messages \
  -H 'x-api-key: sk-xxx' \
  -H 'anthropic-version: 2023-06-01' \
  -H 'Content-Type: application/json' \
  -d '{"model":"glm-5.3","max_tokens":1024,"messages":[{"role":"user","content":"ping"}]}'
```

Claude Code：`ANTHROPIC_BASE_URL="https://tryallapi.com"`、`ANTHROPIC_AUTH_TOKEN` 填你的 Key、`ANTHROPIC_MODEL="glm-5.3"`。更多（tool_stream、交错思考回传）见 [文档站专页](https://docs.tryallapi.com/glm-5.3-api/)。

---

## 七、常见注意

1. **思考关不掉**：传 `thinking.type: "disabled"` 会 400；从 GLM-5.2 迁移时改为 `enabled` 并把 `reasoning_effort` 设为 `low`。
2. **effort 取值变窄**：`medium` / `xhigh` / `minimal` / `none` 在标准 API 报错，只用 `low` / `high` / `max`。
3. **工具调用**：`tool_choice` 默认且仅支持 `auto`；多轮工具调用要把 `reasoning_content` 随 assistant 消息原样回传。
4. **仅文本**：图片输入报错，改用带识图标签的 `glm-5.3-flash`。
5. **Anthropic 格式 404**：多半是 Base URL 多写了 `/v1`，SDK / Claude Code 用 `https://tryallapi.com`。
6. **缓存**：隐式自动缓存，固定前缀 500 token 以上更容易命中，看 `usage.prompt_tokens_details.cached_tokens`。

---

## FAQ

### GLM-5.3 的模型 ID 是什么？

**`glm-5.3`**（全小写），请求体 `"model"` 填该字符串。

### TryAllAPI 的 endpoint 是什么？

- OpenAI 兼容：Base URL `https://tryallapi.com/v1`，`POST /v1/chat/completions`
- Anthropic 兼容：Base URL `https://tryallapi.com`，`POST /v1/messages`

### 官方价是多少？

国内：输入 ¥8 / 输出 ¥28 / 缓存命中 ¥2；国际：输入 $1.4 / 缓存输入 $0.26 / 输出 $4.4（每百万 tokens）。聚合侧 ≈ 基价 $1.40 / $4.40 × 分组倍率（2026-10-21 快照：1 / 1.5 / 2.2）。

### 上下文和最大输出是多少？

上下文 1M tokens，最大输出 128K tokens，仅文本输入。

### 如何试用 TryAllAPI？

站点首页：[https://tryallapi.com/](https://tryallapi.com/)；注册试用入口：[https://tryallapi.com/register?aff=5A6A](https://tryallapi.com/register?aff=5A6A)。

---

## 八、小结与试用入口

把 GLM-5.3 接到业务里，核心是：**在价格页确认 model id 与两种 Base URL → 读懂基价 × 分组倍率 → 记住「思考只能开、effort 只有 low/high/max」→ 用 chat/completions 或 messages 出结果**。

| 资源 | 链接 |
| --- | --- |
| TryAllAPI 首页 | [https://tryallapi.com/](https://tryallapi.com/) |
| 注册试用（aff） | [https://tryallapi.com/register?aff=5A6A](https://tryallapi.com/register?aff=5A6A) |
| 文档站专页 | [glm-5.3-api](https://docs.tryallapi.com/glm-5.3-api/) |
| 价格页 | [tryallapi.com/pricing](https://tryallapi.com/pricing) |

价格取数 2026-10-21 北京，以页面实时为准。

---

## License

本文档以个人接入笔记形式发布，示例代码可自由复用。截图与模型广场文案归原平台；价格以实时页面为准。TryAllAPI 为第三方聚合，非智谱官方产品。
