# 腾讯企点运行时确认方案

确认企点支持必须运行时验证，静态代码只能说明存在底层类型入口。

验证顺序：
1. 在 `NapCatCore` 初始化后直接调用 `this.context.session.getQiDianService()`，检查 service 是否存在、`isNull()` 是否为 false、企点方法是否是 function。
2. 使用真实有企点权限/企点会话的账号运行 NapCat；普通 QQ 账号没有企点会话样本时不能验证消息能力。
3. 检查最近联系人或消息监听原始数据，重点观察 `chatType === 102` (`KCHATTYPETEMPBUSSINESSCRM`) 和 `chatType === 117` (`KCHATTYPETEMPWPA`) 是否出现，以及是否有稳定 `peerUid`、`senderUid`、`msgId`。
4. 若 service 可用且会话数据出现，再尝试调用 `getMsgsIncludeSelf`/`getMsgsByMsgId` 读取历史消息，最后再尝试构造 `Peer` 调用 `sendMsg`，验证发送能力。
5. 如果 `getQiDianService()` 不存在或 `isNull()` 为 true，说明当前 QQNT/wrapper 不具备可用企点 service；如果有 service 但没有 102/117 会话样本，只能认为可能支持查询类接口，不能证明支持企点会话收发。