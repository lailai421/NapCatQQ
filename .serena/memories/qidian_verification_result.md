# 腾讯企点验证结果

本次验证环境：macOS，本机存在 `/Applications/QQ.app`，QQ package 版本 `6.9.65-31363`，`/Applications/QQ.app/Contents/Resources/app/wrapper.node` 存在。

已完成验证：
- 使用系统 Node 和 `ELECTRON_RUN_AS_NODE=1 /Applications/QQ.app/Contents/MacOS/QQ` 直接 `process.dlopen(wrapper.node)` 均失败，进程退出码 134；因此不能用单独加载 native 模块的方式确认运行时 service 对象。
- 对实际 QQ.app 的 `wrapper.node` / `major.node` 进行字符串过滤，确认二进制内存在大量企点相关符号：`getQiDianService`、`NodeIKernelQiDianService`、`requestQidianUidFromUin`、`requestWpaSigT`、`requestWpaCorpInfo`、`requestWpaUserInfo`、`BusinessCRMMsgCodec`、`WPAMsgCodec`、`PrepareTempChat. is kTempChatBusinessCRM`、`kTempChatWPA`、`DecodeTempChatInfo WPAMsgCodec`、`Recv Qidian msg` 等。
- 结论：本机 QQ native 层确实包含企点 service、WPA/CRM 临时会话、企点消息/远程控制相关实现痕迹；但这仍不是完整 NapCat 支持证明。

已加入代码探针：
- `packages/napcat-core/index.ts` 新增 `NAPCAT_QIDIAN_PROBE=1` 开关。
- 启动后打印 `[QiDianProbe] service`，报告 `getQiDianService()` 是否存在、`isNull()` 结果和关键方法列表。
- 收/发消息时若 `chatType` 为 `KCHATTYPETEMPBUSSINESSCRM` 或 `KCHATTYPETEMPWPA`，打印 `[QiDianProbe] message`，包含 `chatType`、`peerUid`、`senderUid`、`msgId`、`elementTypes` 等元数据。

验证命令：
- `pnpm install` 成功。
- `pnpm typecheck` 成功。
- `pnpm exec eslint "packages/napcat-core/index.ts"` 成功。
- `pnpm lint` 失败，但失败项均在本次未修改的已有文件：`packages/napcat-onebot/network/plugin/loader.ts`、`packages/napcat-webui-backend/src/helper/TotpHelper.ts`、`packages/napcat-webui-frontend/src/pages/dashboard/config/index.tsx`、`packages/napcat-webui-frontend/src/pages/dashboard/config/webui.tsx`、`packages/napcat-webui-frontend/src/pages/web_login.tsx`。

仍需真实账号验证：
- 用具有腾讯企点/WPA/CRM 会话的账号启动 NapCat，并设置 `NAPCAT_QIDIAN_PROBE=1`。
- 若日志出现 `service.exists=true`、`service.isNull=false` 且有方法列表，同时收到 `[QiDianProbe] message` 且 `chatType` 为 102 或 117，才可确认企点会话数据真实进入 NapCat。
- 随后再验证读取历史和发送文本消息，才能确认收发链路可二开。