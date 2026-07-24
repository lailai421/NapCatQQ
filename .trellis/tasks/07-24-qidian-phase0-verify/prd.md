# REQ-0：企点运行时验证

> 来源：`docs/qidian-adaptation-execution-plan.md` 阶段 0

## 目标

确认当前 QQNT、wrapper、账号环境具备企点底层能力和企点会话数据。

## 背景

NapCat 已有企点底层入口代码：

- `packages/napcat-core/services/NodeIKernelQiDianService.ts` — 已声明企点 native 接口
- `packages/napcat-core/wrapper.ts` — 已暴露 `getQiDianService()`
- `packages/napcat-core/types/msg.ts` — 已定义 `ChatType.KCHATTYPETEMPBUSSINESSCRM = 102` 和 `ChatType.KCHATTYPETEMPWPA = 117`
- `packages/napcat-core/index.ts` — 已有 `NAPCAT_QIDIAN_PROBE=1` 运行时探针

但从未在真实企点账号上验证过这些能力是否真正可用。

## 需求

本阶段**不写任何新代码**，只使用现有探针能力：

1. 用真实企点账号启动 NapCat（`NAPCAT_QIDIAN_PROBE=1`）
2. 检查 `[QiDianProbe] service` 日志，确认 native service 存在且关键方法可用
3. 用外部访客/WPA/CRM 入口发送一条文本消息给企点账号
4. 检查 `[QiDianProbe] message` 日志，确认 `chatType=102` 或 `117` 的消息进入监听器
5. 收集并保存完整日志

## 前置条件

- [ ] 拥有企点账号（可登录 QQNT）
- [ ] 知道如何触发企点会话消息（外部访客、WPA 入口、CRM 入口等）

## 交付物

1. **一份完整运行日志**（从启动到收到企点消息），必须包含：
   - `[QiDianProbe] service` 输出（含 `exists`, `isNull`, `methods`）
   - `[QiDianProbe] message` 输出（含 `chatType`, `peerUid`, `peerUin`, `senderUid`, `senderUin`, `msgId`, `msgSeq`, `elementTypes`）
   - 相关错误日志（如有）

2. **一份企点会话样本记录**，从日志中提取，包含：
   - `chatType` 值
   - `peerUid` 和 `peerUin`
   - `senderUid` 和 `senderUin`
   - `msgId` 和 `msgSeq`
   - 消息元素类型列表

3. **一个明确门禁结论**，根据下表判断：

   | 条件 | 结论 |
   |---|---|
   | `exists=true`、`isNull=false`、`methods` 含 `requestWpaSigT` 和 `requestQidianUidFromUin`，且有企点消息日志 | ✅ **进入阶段 1** |
   | service 不存在 | ❌ 中止：wrapper 未暴露企点 service |
   | `isNull=true` | ❌ 中止：native service 不可用 |
   | 无 `chatType=102` 或 `117` 消息 | ❌ 中止：账号无企点会话数据 |

## 本阶段不改动

- 不修改任何代码文件
- 不修改 WebUI
- 不新增接口或 action
- 不改变消息收发行为

## 探针工作机制

探针代码位于 `packages/napcat-core/index.ts`：

- `logQiDianServiceProbe()` — 启动时执行一次，检测 `getQiDianService()` 是否可用
- `logQiDianMessageProbe()` — 收到消息时执行，过滤 `chatType=102` 和 `117` 的消息并记录
- 仅在 `NAPCAT_QIDIAN_PROBE=1` 时激活
