# gpt-watch

**快速检测 ChatGPT / Codex 是否在偷偷降智。** 当请求的模型被静默换成更小、更旧或更弱的模型时，界面可能毫无提示；gpt-watch 用本地日志和主动探针帮你第一时间发现并留证。纯 Python 标准库，数据不出本机。

**中文** | [English](README.en.md)

> 基于 [ysh1112/codex-model-watch](https://github.com/ysh1112/codex-model-watch) 二次开发，感谢原作者 **ysh1112**。

![探针测试](screenshots/panel-probe.png)

## 为什么需要它

容量紧张或策略调整时，ChatGPT / Codex 可能把请求悄悄派给更弱的模型，甚至把响应里的模型名伪装成你请求的那个。回答质量下降，但你很难判断是「模型变笨了」还是别的原因。gpt-watch 把「降智」变成可验证的数据：

- **降智检测**：快速探针抓直接换绑；深度金丝雀用身份指纹、知识截止 / 当代性、推理深度和答案校验，抓「自报也造假」的软换。
- **证据与历史**：每次探测记录请求 → 实际、通过 / 不通过 / 不确定；支持按模型、类型、结果、时间、判定原因搜索，原因悬停查看。
- **用量与额度**：按模型 / 项目统计轮次、token、时长，展示 5h / 7d 额度水位与容量拒单，作为辅助判断。

![监控面板](screenshots/panel-overview.png)

## 快速开始

需要 Python 3.8+，无第三方依赖。

```bash
git clone https://github.com/liandu2024/gpt-watch.git
cd gpt-watch

python3 codex_model_watch.py          # 读取真实 Codex 日志
python3 codex_model_watch.py --demo   # 演示数据
```

打开 <http://127.0.0.1:8787>。启动后会增量扫描 `~/.codex/sessions/**/*.jsonl`，数据库在 `~/.codex-model-watch/state.db`。

## 两种探针

| | 快速探针 | 深度金丝雀 |
|---|---|---|
| 做什么 | 发 `hi`，读取响应中的模型名 | 身份、知识、推理多维测试 |
| 适用 | 验证链路与直接换绑 | 怀疑自报造假、旧壳顶包 |
| 成本 | 秒级、额度极少 | 数分钟、额度较多 |
| 结果 | 通过 / 不通过 / 不确定 | 通过 / 不通过 / 不确定 |

> 2026-10 起，服务端软换时可能连 `response.created` 里的模型名一起伪造。快速探针只能证明「链路活着」，不能单独证明「货是真的」；这时用深度金丝雀交叉判断。

记录里的「原因」在结果上悬停查看。完整判定只代表探测那一刻；容量紧张时换绑是动态开关的。

### 参考信号与证据

快速探针会同时记录 `x-codex-turn-state` 的长度、`x-codex-safety-buffering-faster-model`、`request-id` 和 `cf-ray`。社区观察到 turn-state 长度与降级路由相关（例如个人号约 292 健康 / 312 降级），但这是**启发式**且可能随服务端变化，**不参与通过 / 不通过判定**；权威信号仍是「请求模型 = 返回模型」和金丝雀行为。

探测记录右上角的「导出证据」可下载 JSON，包含请求 / 实际模型、耗时、判定、request-id、cf-ray 和 turn-state 长度，方便附到工单。

## 数据来源与限制

- **日志侧**：解析 `turn_context`、`token_usage_record`、`task_complete`、`token_count`，得到实际模型、token、时长、错误与 5h/7d 额度。
- **历史偷换无法从日志还原**：Codex 会把被换后的模型一致化写进日志，请求字段和实际字段都会变成替换后的模型；只有主动探针能验证当下。
- **探针是旁证**：模型自报身份可能被指令覆盖或训练残留污染；金丝雀结果建议结合多种信号判断。
- rollout 是 Codex 内部格式，可能随版本变化（本工具在 Codex CLI 0.153–0.155 实测）。

## 常用参数

| 参数 | 说明 |
|---|---|
| `--host 0.0.0.0` | 允许同一局域网 / 手机访问（默认仅 `127.0.0.1`） |
| `--port 8787` | 网页端口 |
| `--max-age-days 30` | 只解析最近 N 天日志，`0` = 全部 |
| `--codex-home PATH` | Codex 主目录（默认 `~/.codex`） |
| `--demo` | 内置演示数据，不读取真实日志 |
| `--scan-only` | 只扫描并打印模型分布摘要，不启动网页 |
| `--no-open` | 不自动打开浏览器 |
| `--api-base` / `--api-key` | 金丝雀改打中转站 / 号池的 `/v1/chat/completions` |

局域网示例：

```bash
python3 codex_model_watch.py --host 0.0.0.0 --no-open
# 同一 Wi-Fi 访问 http://本机局域网IP:8787
```

界面跟随系统主题，支持亮 / 暗色；手机顶栏两行显示，表格可横向滚动。探测进度与结果用顶部单行提示条展示，不挤压页面。

## 隐私

- 解析、统计、存储全部在本机；网页默认只监听 `127.0.0.1`。
- 唯一外发请求是你手动触发的探针（发给你自己的 Codex 后端或指定中转站）。
- 仓库不含凭据、遥测或上报。

## 致谢

本项目基于 [ysh1112/codex-model-watch](https://github.com/ysh1112/codex-model-watch) 二次开发，感谢原作者 **ysh1112** 的开源工作。探针与金丝雀的方法也参考了 BazaarLink / LLMprobe-engine 的公开思路，以及 [codex_state_monitor](https://github.com/YueDongSun/codex_state_monitor) 对 `x-codex-turn-state` / safety-buffering 头的公开分析。

## License

[MIT](LICENSE)
