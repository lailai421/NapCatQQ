# 腾讯企点适配二次开发执行计划

日期：2026-07-23

## 0. 结论与目标

本计划用于指导 NapCatQQ 逐步适配腾讯企点。当前项目已经存在企点相关底层入口和会话枚举，但缺少稳定 API、OneBot action、事件转换、发送链路，因此当前状态不是完整支持腾讯企点。

第一轮开发目标是完成最小闭环：

1. 运行时确认 `getQiDianService()` 可用。
2. 封装 `QiDianApi`。
3. 上报企点文本消息。
4. 发送企点文本消息。

第一轮开发不处理历史消息、已读、图片、文件、语音、富文本、联系人列表同步。这些能力放入第二轮。

## 1. 已确认代码事实

- `packages/napcat-core/services/NodeIKernelQiDianService.ts` 已声明底层接口：
  - `requestQidianUidFromUin`
  - `requestWpaCorpInfo`
  - `requestWpaUserInfo`
  - `requestWpaSigT`
  - `addKernelQiDianListener`
  - `removeKernelQiDianListener`
  - `isNull`
- `packages/napcat-core/wrapper.ts` 已暴露 `getQiDianService()`。
- `packages/napcat-core/types/msg.ts` 已包含企点会话类型：
  - `ChatType.KCHATTYPETEMPBUSSINESSCRM = 102`
  - `ChatType.KCHATTYPETEMPWPA = 117`
- `packages/napcat-core/index.ts` 已加入 `NAPCAT_QIDIAN_PROBE=1` 运行时探针。
- `packages/napcat-onebot/action/router.ts` 中 `QidianGetAccountInfo : 'qidian_get_account_info'` 仍处于注释状态。
- `packages/napcat-onebot/action/msg/SendMsg.ts` 当前 `message_type` 只支持 `private` 和 `group`。
- `packages/napcat-onebot/api/msg.ts` 当前 `parseMessageV2()` 只转换群聊、好友私聊、群临时会话，遇到 `chatType=102` 和 `chatType=117` 会返回 `undefined`。

## 2. 阶段总览

### 阶段 0：运行时验证

目标：确认当前 QQNT、wrapper、账号环境具备企点底层能力和企点会话数据。

交付物：

- 一份真实运行日志。
- 一份企点会话样本记录。
- 一个明确门禁结论：进入阶段 1，中止阶段 1。

### 阶段 1：企点 API 封装

目标：把 `getQiDianService()` 封装成稳定的 `NTQQQiDianApi`，并提供 OneBot 扩展 action 查询状态。

交付物：

- `packages/napcat-core/apis/qidian.ts`
- `core.apis.QiDianApi`
- `get_qidian_status`

### 阶段 2：接收企点消息

目标：把 `chatType=102` 和 `chatType=117` 的原始消息转换成 OneBot 扩展事件。

交付物：

- `qidian_message` 事件结构。
- 纯文本消息上报。
- 自己发送的企点消息回显上报。

### 阶段 3：发送企点文本消息

目标：新增 `send_qidian_msg` action，通过现有 `MsgApi` 发送纯文本企点消息。

交付物：

- `packages/napcat-onebot/action/qidian/SendQiDianMsg.ts`
- `send_qidian_msg`
- 纯文本发送成功返回 `message_id`

### 阶段 4：验收与后续扩展

目标：固定第一轮验收标准，记录第二轮扩展入口。

交付物：

- 类型检查通过。
- 新增文件 ESLint 通过。
- 真实账号收发闭环通过。
- 第二轮扩展清单。

## 3. REQ-0 运行时验证

### 3.1 需求标识

- `ID`：`REQ-0`
- `原始描述`：确认运行时 `getQiDianService()` 和企点会话数据真实可用。

### 3.2 实现目标

使用真实企点账号启动 NapCat，打开运行时探针，确认 native service 存在并且企点会话消息进入 NapCat 原始消息监听器。

### 3.3 前端改动点

本阶段不改 WebUI。

### 3.4 后端改动点

- 使用现有 `packages/napcat-core/index.ts` 中的探针能力。
- 保留环境变量开关 `NAPCAT_QIDIAN_PROBE=1`。
- 保留日志前缀：
  - `[QiDianProbe] service`
  - `[QiDianProbe] message`

### 3.5 数据结构与接口调整

本阶段不新增公开接口。

运行日志必须记录以下字段：

- `exists`
- `isNull`
- `methods`
- `chatType`
- `peerUid`
- `peerUin`
- `senderUid`
- `senderUin`
- `msgId`
- `msgSeq`
- `elementTypes`

### 3.6 影响范围与风险

- 本阶段只启用日志探针，不改变消息收发行为。
- 当日志中没有 `[QiDianProbe] service` 时，说明启动路径没有走到探针位置。
- 当 `exists=false` 时，说明当前 wrapper 未暴露企点 service。
- 当 `isNull=true` 时，说明 native service 当前不可用。
- 当没有 `chatType=102` 和 `chatType=117` 消息时，说明账号没有产出企点会话样本。

### 3.7 实施步骤

1. 执行依赖安装。

   ```bash
   pnpm install
   ```

2. 执行类型检查，确认基线能编译。

   ```bash
   pnpm typecheck
   ```

3. 使用真实企点账号启动 NapCat，并打开探针。

   ```bash
   NAPCAT_QIDIAN_PROBE=1 pnpm dev:shell
   ```

4. 使用外部访客、WPA 入口、CRM 会话入口给当前账号发送一条纯文本消息。

5. 在 NapCat 日志中查找 `[QiDianProbe] service`。

6. 在 NapCat 日志中查找 `[QiDianProbe] message`，并记录 `chatType=102` 和 `chatType=117` 中出现的类型。

7. 当 `exists=true`、`isNull=false`、`methods` 包含 `requestWpaSigT` 和 `requestQidianUidFromUin`，并且出现企点消息日志时，进入阶段 1。

8. 当 service 不存在、service 为空、企点消息样本缺失三类情况任一出现时，中止阶段 1，先解决账号、QQ 版本、wrapper 加载路径。

### 3.8 测试要点

- 正向测试：真实企点会话发来纯文本，日志出现 `chatType=102` 和 `chatType=117` 中的一个类型。
- 反向测试：不设置 `NAPCAT_QIDIAN_PROBE=1` 时，不输出企点探针日志。
- 稳定性测试：普通私聊、群聊消息日志行为保持不变。

## 4. REQ-1 企点 API 封装

### 4.1 需求标识

- `ID`：`REQ-1`
- `原始描述`：封装 `getQiDianService()`，提供稳定企点 API 和状态查询 action。

### 4.2 实现目标

新增 `NTQQQiDianApi`，统一访问 native 企点 service。上层业务不直接调用 `this.context.session.getQiDianService()`。

### 4.3 前端改动点

本阶段不改 WebUI。

### 4.4 后端改动点

新增文件：

- `packages/napcat-core/apis/qidian.ts`
- `packages/napcat-onebot/action/qidian/GetQiDianStatus.ts`

修改文件：

- `packages/napcat-core/apis/index.ts`
- `packages/napcat-core/index.ts`
- `packages/napcat-onebot/action/router.ts`
- `packages/napcat-onebot/action/index.ts`

`packages/napcat-core/apis/qidian.ts` 新增导出：

```ts
export const QIDIAN_CHAT_TYPES = new Set<ChatType>([
  ChatType.KCHATTYPETEMPBUSSINESSCRM,
  ChatType.KCHATTYPETEMPWPA,
]);

export interface QiDianServiceStatus {
  exists: boolean;
  isNull: boolean | null;
  methods: string[];
}

export class NTQQQiDianApi {
  getStatus(): QiDianServiceStatus;
  isQiDianChatType(chatType: ChatType): boolean;
  requestQidianUidFromUin(payload: unknown): unknown;
  requestWpaCorpInfo(payload: unknown): unknown;
  requestWpaUserInfo(payload: unknown): unknown;
  requestWpaSigT(arg1: unknown, arg2: unknown): unknown;
}
```

`packages/napcat-core/index.ts` 修改点：

- import `NTQQQiDianApi`。
- 在 `this.apis` 中加入 `QiDianApi: new NTQQQiDianApi(this.context, this)`。
- 在 `StableNTApiWrapper` 中加入 `QiDianApi: NTQQQiDianApi`。
- 将 `QIDIAN_PROBE_METHODS` 和 `QIDIAN_PROBE_CHAT_TYPES` 迁移到 `qidian.ts`，探针复用 `QiDianApi.getStatus()`。

`packages/napcat-onebot/action/router.ts` 修改点：

```ts
QiDianGetStatus: 'get_qidian_status',
```

`packages/napcat-onebot/action/qidian/GetQiDianStatus.ts` 行为：

- 返回 `this.core.apis.QiDianApi.getStatus()`。
- 当 service 不存在时也正常返回结构化状态，不抛异常。

`packages/napcat-onebot/action/index.ts` 修改点：

- import `GetQiDianStatus`。
- 在 `getAllHandlers()` 的 action 列表中加入 `new GetQiDianStatus(obContext, core)`。

### 4.5 数据结构与接口调整

新增 action：`get_qidian_status`

返回结构：

```json
{
  "exists": true,
  "is_null": false,
  "methods": [
    "requestWpaSigT",
    "requestQidianUidFromUin",
    "requestWpaCorpInfo",
    "requestWpaUserInfo"
  ],
  "supported_chat_types": [102, 117]
}
```

字段说明：

- `exists`：当前 session 是否返回 service 对象。
- `is_null`：native service 的 `isNull()` 结果。没有 `isNull` 方法时返回 `null`。
- `methods`：已检测到的企点方法名。
- `supported_chat_types`：当前代码识别的企点会话类型列表。

### 4.6 影响范围与风险

- 影响 `NapCatCore.apis` 类型定义和初始化。
- 影响 OneBot action 列表。
- 当 native 方法参数形状未确认时，查询类方法只能作为透传封装使用。
- `get_qidian_status` 不依赖真实企点会话样本，只依赖 native service。

### 4.7 实施步骤

1. 新建 `packages/napcat-core/apis/qidian.ts`，实现 `NTQQQiDianApi`。
2. 修改 `packages/napcat-core/apis/index.ts`，导出 `qidian.ts`。
3. 修改 `packages/napcat-core/index.ts`，注册 `QiDianApi`。
4. 修改 `packages/napcat-onebot/action/router.ts`，新增 `QiDianGetStatus`。
5. 新建 `packages/napcat-onebot/action/qidian/GetQiDianStatus.ts`。
6. 修改 `packages/napcat-onebot/action/index.ts`，注册 `GetQiDianStatus`。
7. 使用 HTTP、WebSocket、插件管理器中任一路径调用 `get_qidian_status`。

### 4.8 测试要点

- `pnpm typecheck` 通过。
- `pnpm exec eslint "packages/napcat-core/apis/qidian.ts"` 通过。
- `pnpm exec eslint "packages/napcat-onebot/action/qidian/GetQiDianStatus.ts"` 通过。
- 未登录企点会话时，`get_qidian_status` 返回结构化数据。
- 普通消息 action 不受影响。

## 5. REQ-2 接收企点消息

### 5.1 需求标识

- `ID`：`REQ-2`
- `原始描述`：识别 `chatType=102` 和 `chatType=117`，并上报企点消息事件。

### 5.2 实现目标

当 NapCat 收到企点会话消息时，OneBot 网络适配器收到 `qidian_message` 扩展事件。事件中保留原始 `chatType`、企点 peer、发送方信息、消息段和 raw message。

### 5.3 前端改动点

本阶段不改 WebUI。

### 5.4 后端改动点

新增文件：

- `packages/napcat-onebot/event/message/OB11QiDianMessageEvent.ts`

修改文件：

- `packages/napcat-onebot/event/message/index.ts`
- `packages/napcat-onebot/api/msg.ts`
- `packages/napcat-onebot/index.ts`

`OB11QiDianMessageEvent.ts` 新增事件类：

```ts
export class OB11QiDianMessageEvent extends OB11BaseMessageEvent {
  message_type = 'qidian';
  sub_type = 'business_crm';
  qidian_chat_type: number;
  qidian_peer_uid: string;
  qidian_peer_uin: string;
  sender_uid: string;
  target_id: number;
}
```

`packages/napcat-onebot/api/msg.ts` 修改点：

- 新增 `isQiDianChatType()` 调用，复用 `this.core.apis.QiDianApi.isQiDianChatType(msg.chatType)`。
- 在 `parseMessageV2()` 处理 `senderUin` 和 `peerUin` 补齐逻辑时，为企点会话保留 `peerUid`。
- 新增 `handleQiDianMessage()`：
  - `message_type` 设为 `qidian`。
  - `sub_type` 在 `chatType=102` 时设为 `business_crm`。
  - `sub_type` 在 `chatType=117` 时设为 `wpa`。
  - `qidian_chat_type` 写入原始 `chatType`。
  - `qidian_peer_uid` 写入 `msg.peerUid`。
  - `qidian_peer_uin` 写入 `msg.peerUin`。
  - `target_id` 使用 `Number(msg.peerUin)`，当 `peerUin` 为空时使用 `0`。
- `parseMessageV2()` 对企点会话不再返回 `undefined`。

`packages/napcat-onebot/index.ts` 修改点：

- `emitMsg()` 中仍复用 `handleMsg(message)`。
- `handleMsg()` 中允许 `parseMessageV2()` 返回 `qidian` 扩展消息。
- `handleGroupEvent()` 只在群聊执行。
- `handlePrivateMsgEvent()` 不处理 `qidian`，避免把企点消息当好友私聊事件解析。

### 5.5 数据结构与接口调整

新增上报事件：

```json
{
  "post_type": "message",
  "message_type": "qidian",
  "sub_type": "business_crm",
  "self_id": 123456,
  "user_id": 0,
  "time": 1710000000,
  "message_id": 10001,
  "real_id": 10001,
  "real_seq": "123",
  "qidian_chat_type": 102,
  "qidian_peer_uid": "u_xxx",
  "qidian_peer_uin": "0",
  "sender_uid": "u_yyy",
  "message": [
    {
      "type": "text",
      "data": {
        "text": "你好"
      }
    }
  ],
  "raw_message": "你好"
}
```

### 5.6 影响范围与风险

- 影响 `parseMessageV2()` 的消息类型分发。
- 企点消息不进入好友私聊事件解析，减少误报好友撤回、好友通知的风险。
- 当 `senderUin` 无法通过 `UserApi.getUinByUidV2()` 补齐时，事件中的 `user_id` 使用 `0`，同时保留 `sender_uid`。
- 文本消息段复用现有 `textElement` 转换逻辑。

### 5.7 实施步骤

1. 新建 `OB11QiDianMessageEvent.ts`，定义扩展事件类。
2. 修改 `packages/napcat-onebot/event/message/index.ts`，导出事件类。
3. 修改 `packages/napcat-onebot/api/msg.ts`，新增企点分支。
4. 修改 `initializeMessage()`，允许企点消息填充 `message_type=qidian`。
5. 修改 `handleMsg()` 分发逻辑，企点消息只执行消息上报。
6. 使用真实企点账号接收一条纯文本。
7. 检查 HTTP 上报、WebSocket 上报、插件事件中的 JSON。

### 5.8 测试要点

- 真实 `chatType=102` 消息上报为 `message_type=qidian`。
- 真实 `chatType=117` 消息上报为 `message_type=qidian`。
- `sub_type` 与 `chatType` 一一对应。
- `message` 中包含 text 消息段。
- 普通私聊、群聊、群临时会话上报结构保持不变。

## 6. REQ-3 发送企点文本消息

### 6.1 需求标识

- `ID`：`REQ-3`
- `原始描述`：新增 `send_qidian_msg`，向企点会话发送纯文本消息。

### 6.2 实现目标

外部调用方通过 `send_qidian_msg` 指定 `chat_type`、`peer_uid`、`message`，NapCat 构造企点 `Peer`，复用现有消息元素构造和发送逻辑发送纯文本。

### 6.3 前端改动点

本阶段不改 WebUI。

### 6.4 后端改动点

新增文件：

- `packages/napcat-onebot/action/qidian/SendQiDianMsg.ts`

修改文件：

- `packages/napcat-onebot/action/router.ts`
- `packages/napcat-onebot/action/index.ts`
- `packages/napcat-onebot/action/msg/SendMsg.ts`

`SendQiDianMsg.ts` 行为：

- action 名称：`send_qidian_msg`
- 只接收企点专用 payload。
- 只允许文本消息段。
- 构造 `Peer`：

```ts
{
  chatType: ChatType.KCHATTYPETEMPBUSSINESSCRM,
  peerUid: payload.peer_uid,
  guildId: '',
}
```

当 `payload.chat_type=117` 时，`chatType` 使用 `ChatType.KCHATTYPETEMPWPA`。

发送逻辑：

```ts
const messages = normalize(payload.message, true);
const { sendElements, deleteAfterSentFiles } =
  await this.obContext.apis.MsgApi.createSendElements(messages, peer);
const returnMsg =
  await this.obContext.apis.MsgApi.sendMsgWithOb11UniqueId(peer, sendElements, deleteAfterSentFiles, payload.timeout);
return { message_id: returnMsg.id! };
```

`packages/napcat-onebot/action/msg/SendMsg.ts` 修改点：

- 导出 `normalize()`，供 `SendQiDianMsg.ts` 复用。
- 保持原有 `send_msg` 的 `message_type` 枚举不变。
- 不把 `qidian` 加入 `send_msg`，避免破坏 OneBot 11 兼容行为。

`packages/napcat-onebot/action/router.ts` 修改点：

```ts
SendQiDianMsg: 'send_qidian_msg',
```

`packages/napcat-onebot/action/index.ts` 修改点：

- import `SendQiDianMsg`。
- 在 `getAllHandlers()` 加入 `new SendQiDianMsg(obContext, core)`。

### 6.5 数据结构与接口调整

新增 action：`send_qidian_msg`

请求结构：

```json
{
  "chat_type": 102,
  "peer_uid": "u_xxx",
  "message": "你好",
  "timeout": 30000
}
```

请求字段：

- `chat_type`：必填，只允许 `102` 和 `117`。
- `peer_uid`：必填，来自接收事件中的 `qidian_peer_uid`。
- `message`：必填，第一轮只允许字符串文本。
- `timeout`：非必填，单位毫秒。

返回结构：

```json
{
  "message_id": 10002
}
```

### 6.6 影响范围与风险

- 影响 action 注册列表。
- 第一轮只发送文本，避免媒体上传链路干扰企点验证。
- 当 QQNT 要求 WPA 签名时，直接发送会返回错误。此时进入补充任务 `REQ-3A`，先调用 `requestWpaSigT` 取得签名，再执行发送。
- 当 `peer_uid` 来自非企点事件时，native 发送会返回错误。

### 6.7 实施步骤

1. 新建 `packages/napcat-onebot/action/qidian/SendQiDianMsg.ts`。
2. 定义 payload schema，限制 `chat_type` 为 `102` 和 `117`。
3. 校验 `message` 只包含字符串文本。
4. 使用 `payload.chat_type` 构造企点 `Peer`。
5. 调用 `createSendElements()` 创建发送元素。
6. 调用 `sendMsgWithOb11UniqueId()` 发送消息。
7. 修改 `router.ts` 和 `action/index.ts` 注册 action。
8. 使用阶段 2 收到的 `qidian_peer_uid` 调用 `send_qidian_msg`。
9. 检查对端企点会话收到文本。
10. 检查 NapCat 是否通过 `onAddSendMsg` 上报自己发送的企点消息。

### 6.8 测试要点

- `chat_type=102` 可发送纯文本。
- `chat_type=117` 可发送纯文本。
- `chat_type` 传入非 `102`、非 `117` 时返回参数错误。
- `peer_uid` 为空时返回参数错误。
- `message` 为空字符串时返回参数错误。
- 发送成功后返回 `message_id`。
- 对端实际收到文本。
- 自己发送消息回显上报为 `message_type=qidian`。

## 7. REQ-3A WPA 签名补充任务

### 7.1 触发条件

当 `send_qidian_msg` 构造企点 `Peer` 后直接发送失败，并且日志显示缺少 WPA 签名、临时会话准备失败、native 返回权限错误时，执行本任务。

### 7.2 实现目标

在发送前补齐 `requestWpaSigT` 调用，取得 native 层发送所需签名数据。

### 7.3 后端改动点

修改文件：

- `packages/napcat-core/apis/qidian.ts`
- `packages/napcat-onebot/action/qidian/SendQiDianMsg.ts`

新增 `NTQQQiDianApi.prepareQiDianPeer()`：

- 输入 `chatType` 和 `peerUid`。
- 当 `chatType=117` 时调用 `requestWpaSigT`。
- 记录 native 返回结构到 debug 日志。
- 返回可用于发送的 `Peer`。

### 7.4 实施步骤

1. 在失败日志中记录 native 原始错误。
2. 使用 `requestWpaSigT` 对阶段 2 收到的 `qidian_peer_uid` 发起签名请求。
3. 打印返回结构，确认关键字段。
4. 将签名结果写入发送前准备逻辑。
5. 再次调用 `send_qidian_msg`。

### 7.5 测试要点

- 无签名发送失败时，错误信息可读。
- 有签名后发送成功。
- 签名失败时，不调用 `sendMsgWithOb11UniqueId()`。

## 8. 第一轮验收标准

第一轮适配完成时必须满足以下标准：

1. `pnpm typecheck` 通过。
2. 新增企点相关 TypeScript 文件 ESLint 通过。
3. `get_qidian_status` 返回结构化状态。
4. 真实企点文本消息上报为 `message_type=qidian`。
5. 上报事件保留 `qidian_chat_type`、`qidian_peer_uid`、`sender_uid`。
6. `send_qidian_msg` 能向同一企点会话发送纯文本。
7. 发送成功返回 `message_id`。
8. 普通私聊、群聊、群临时会话收发行为不变。

推荐验证命令：

```bash
pnpm typecheck
pnpm exec eslint "packages/napcat-core/apis/qidian.ts"
pnpm exec eslint "packages/napcat-onebot/action/qidian/GetQiDianStatus.ts"
pnpm exec eslint "packages/napcat-onebot/action/qidian/SendQiDianMsg.ts"
pnpm exec eslint "packages/napcat-onebot/api/msg.ts"
pnpm exec eslint "packages/napcat-onebot/index.ts"
```

## 9. 第二轮扩展清单

第一轮闭环通过后，再按以下顺序扩展：

1. `get_qidian_msg_history`：读取企点历史消息。
2. `mark_qidian_msg_as_read`：标记企点消息已读。
3. `get_qidian_user_info`：查询 WPA 用户信息。
4. `get_qidian_corp_info`：查询 WPA 企业信息。
5. 图片消息接收与发送。
6. 文件消息接收与发送。
7. 语音消息接收与发送。
8. 富文本、卡片、转发消息支持。
9. WebUI 调试页。
10. OpenAPI 文档更新。

## 10. 推荐开发顺序

严格按以下顺序开发：

1. 完成 `REQ-0`，拿到真实日志。
2. 完成 `REQ-1`，让 `get_qidian_status` 可调用。
3. 完成 `REQ-2`，让企点文本消息可以上报。
4. 完成 `REQ-3`，让企点文本消息可以发送。
5. 当直接发送失败时，完成 `REQ-3A`。
6. 完成第一轮验收。
7. 进入第二轮扩展。

## 11. 开发纪律

- 第一轮只做文本消息，不碰媒体能力。
- 第一轮不改 `send_msg` 的标准 `message_type`。
- 第一轮新增 NapCat 扩展 action，不强行塞进 OneBot 11 标准私聊语义。
- 所有企点 native 调用都集中在 `NTQQQiDianApi`。
- 所有真实企点样本都记录 `chatType`、`peerUid`、`senderUid`、`msgId`。
- 每完成一个阶段都执行类型检查。
- 每次改动 action 注册后都调用一次真实 action 验证。

## 12. 关键文件索引

- 核心入口：`packages/napcat-core/index.ts`
- 企点 service 类型：`packages/napcat-core/services/NodeIKernelQiDianService.ts`
- wrapper 入口：`packages/napcat-core/wrapper.ts`
- 消息类型枚举：`packages/napcat-core/types/msg.ts`
- OneBot 消息转换：`packages/napcat-onebot/api/msg.ts`
- OneBot 监听入口：`packages/napcat-onebot/index.ts`
- 发送 action：`packages/napcat-onebot/action/msg/SendMsg.ts`
- action 路由：`packages/napcat-onebot/action/router.ts`
- action 注册：`packages/napcat-onebot/action/index.ts`
