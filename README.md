# agent-anyapp

把任意联网软件收敛成 AI 可直接调用的一层能力：不打开界面、不点按钮，一个 key 接入，AI 做完一切。

## 核心命题

任何桌面/联网软件被 AI 接管，通道只有三条，优先级固定：

| 档位 | 通道 | 覆盖 | 代价 |
|---|---|---|---|
| 1 | official-api | 云端全量、最权威 | 配额、审批、OAuth |
| 2 | local-db | 本机同步副本、离线、无配额 | 只有本机有的数据 |
| 3 | client-drive | CDP / 命令行 / 窗口 | 脆弱，等价于替人点 |

关键不是这三条各自怎么用，而是**上层只看见一个契约**：`{name, args, returns, channel}`。
AI 不需要知道背后是 HTTP、SQLite 还是点浏览器。

## 组成

```
probe.py     判定一个已装软件该走哪条通道（只读探测，不发写请求）
contract.py  统一工具契约注册表：任意通道 -> 同一个 manifest
demo.py      三条通道各一个真实实现，跑起来即验证
EVIDENCE.md  本机 47 个应用的实测证据矩阵
```

## 已实测（全部真机，非设想）

- 通道 2 通：WPS `wps.db` 3171 条命令定义，直接实例化为工具
- 通道 3 通：Cursor 的 CDP 端点在 9222 实测 200，拿到 page target
- 通道 0 通：对 LarkShell 探测返回 `["local-db", "client-drive"]`，对 Ollama 探测 `official-api` 未起时正确降档
- 规模：本机 47 个应用有可自治通道证据；28 个有明文 SQLite，共 122 个库

## 立刻可跑

```bash
python demo.py                       # 打印统一 manifest
python -c "import sys;sys.path.insert(0,'.');from demo import reg;\
print(reg.call('db.tables', path=r'<任意 .db>'))"
```

## 边界（写清楚，不吹）

- local-db 只拿得到本机同步过的副本；撤回、未同步、他端数据不在其中
- client-drive 需要目标进程已在运行且开 CDP，不是任意软件都开
- 加密库（如微信 v4、飞书 im.db）需要额外取密钥，不在本层范围内
