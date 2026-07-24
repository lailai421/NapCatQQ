# Phase 0 执行步骤

> 在新上下文窗口中，AI 应按照以下步骤引导用户完成验证。

---

## 用户需要准备

- 一台能运行 NapCat 的电脑
- 一个**企点账号**（可登录 QQNT）
- 一个**外部身份**可以向该企点账号发送消息（访客页、WPA 入口、CRM 会话等）

---

## 步骤 1：确认基线可编译

```bash
cd <项目根目录>
pnpm install
pnpm typecheck
```

**AI 应确认：** `pnpm typecheck` 通过，无类型错误。如失败先修复。

---

## 步骤 2：确认探针代码存在

检查以下代码位置：

- `packages/napcat-core/index.ts` 中搜索 `QIDIAN_PROBE`，确认 `logQiDianServiceProbe()` 和 `logQiDianMessageProbe()` 存在。
- 搜索 `NAPCAT_QIDIAN_PROBE`，确认环境变量检测逻辑存在。

**AI 应对用户说：** "探针代码已就位，不需要修改任何文件。"

---

## 步骤 3：启动 NapCat（开启探针）

```bash
NAPCAT_QIDIAN_PROBE=1 pnpm dev:shell
```

**AI 应提醒用户：**
- 使用**企点账号**登录
- 等待登录完成，看到 QQ 在线状态
- 观察启动日志中是否出现 `[QiDianProbe] service`

---

## 步骤 4：采集 service 探针日志

启动完成后，让用户在日志中搜索 `[QiDianProbe] service`。

**AI 应帮助用户解读日志：**

示例日志格式：
```
[QiDianProbe] service {"exists":true,"isNull":false,"methods":["requestWpaSigT","requestQidianUidFromUin",...]}
```

| 字段 | 含义 | 通过条件 |
|---|---|---|
| `exists` | wrapper 是否暴露 `getQiDianService()` | `true` |
| `isNull` | native service 是否为空 | `false` |
| `methods` | 可用的企点方法列表 | 包含 `requestWpaSigT` 和 `requestQidianUidFromUin` |

**如果 `exists=false`：** 中止。记录：wrapper 未暴露企点 service。
**如果 `isNull=true`：** 中止。记录：native service 当前不可用。
**如果 methods 缺少关键方法：** 记录缺失项，但不一定中止——取决于缺少什么。

---

## 步骤 5：触发企点会话消息

**AI 应引导用户操作：**

1. 用**外部身份**（访客/客户）向企点账号发送一条纯文本消息
2. 方式可以是：
   - 通过企点 WPA 客服入口发送
   - 通过 CRM 客户会话入口发送
   - 通过任何能产生 `chatType=102` 或 `117` 会话的方式

**不会产生企点会话的场景（避免）：**
- 普通 QQ 好友私聊（`chatType=1`）
- 普通 QQ 群聊（`chatType=2`）

---

## 步骤 6：采集 message 探针日志

收到消息后，让用户在日志中搜索 `[QiDianProbe] message`。

**AI 应帮助用户提取样本：**

示例日志格式：
```
[QiDianProbe] message {"source":"recv","chatType":102,"peerUid":"u_xxx","peerUin":"123456",...}
```

AI 应让用户复制完整 JSON，并提取以下字段填入样本记录：

| 字段 | 示例值 | 说明 |
|---|---|---|
| `chatType` | `102` | `102`=B2B CRM, `117`=WPA |
| `peerUid` | `u_xxxxx` | 会话对端 UID |
| `peerUin` | `123456` | 会话对端 UIN |
| `senderUid` | `u_yyyyy` | 发送方 UID |
| `senderUin` | `987654` | 发送方 UIN |
| `msgId` | `10001` | 消息 ID |
| `msgSeq` | `123` | 消息序号 |
| `elementTypes` | `[1]` | 消息元素类型（1=文本） |

**如果没有 `[QiDianProbe] message` 日志：**
- 确认 `NAPCAT_QIDIAN_PROBE=1` 是否生效
- 确认消息确实进入了 `onRecvMsg`（看普通消息日志有没有）
- 确认 `chatType` 确实是 102 或 117（不是 1 或 2）
- 以上都确认仍无日志 → 中止：账号无企点会话数据

---

## 步骤 7：反向验证

为了确认探针不影响正常功能：

**AI 应让用户验证：**
1. 关掉 NapCat，**不加** `NAPCAT_QIDIAN_PROBE=1` 重新启动
2. 发一条普通 QQ 私聊消息和群聊消息
3. 确认日志中**没有** `[QiDianProbe]` 相关输出
4. 确认普通消息日志行为正常

---

## 步骤 8：输出门禁结论

**AI 应根据采集到的日志，按以下决策表给出明确结论：**

| 条件组合 | 结论 |
|---|---|
| `exists=true`, `isNull=false`, methods 含 `requestWpaSigT` 和 `requestQidianUidFromUin`, 有企点消息 | ✅ **进入阶段 1** |
| `exists=false` | ❌ **中止阶段 1**：wrapper 未暴露企点 service |
| `exists=true`, `isNull=true` | ❌ **中止阶段 1**：native service 不可用 |
| 无企点消息样本 | ❌ **中止阶段 1**：账号无企点会话数据 |
| methods 缺少关键方法 | ⚠️ **有条件进入**：记录缺失，但可尝试阶段 1 |

---

## 步骤 9：整理交付物

**AI 应帮助用户整理以下文件，保存在 `.trellis/tasks/07-24-qidian-phase0-verify/` 下：**

1. **`probe-service.log`** — `[QiDianProbe] service` 的完整日志行
2. **`probe-message.log`** — `[QiDianProbe] message` 的完整日志行（可含多条）
3. **`sample-record.md`** — 从日志中提取的企点会话样本字段表
4. **`gate-decision.md`** — 门禁结论及依据

---

## AI 执行注意

- 这个阶段**不写任何代码**。如果用户问"需要改什么文件"，回答：不需要。
- 如果环境变量在 Windows 上设置方式不同，提醒用 `set NAPCAT_QIDIAN_PROBE=1 && pnpm dev:shell`。
- 如果用户没有企点账号，提醒先获取账号后再来执行。
- 所有日志解读都基于用户提供的实际终端输出，不要推测。
