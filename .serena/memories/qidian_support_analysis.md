# 腾讯企点支持判断

- `packages/napcat-core/services/NodeIKernelQiDianService.ts` 仅声明底层 `NodeIKernelQiDianService` 接口，包含 `requestQidianUidFromUin`、`requestWpaUserInfo` 等方法。
- `packages/napcat-core/wrapper.ts` 暴露 `getQiDianService()` 类型入口，但代码检索未发现实际调用。
- `packages/napcat-onebot/action/router.ts` 中 `QidianGetAccountInfo : 'qidian_get_account_info'` 被注释，`packages/napcat-onebot/action` 下没有 Qidian action 文件。
- `packages/napcat-core/types/msg.ts` 包含 `KCHATTYPETEMPBUSSINESSCRM = 102`、`KCHATTYPETEMPWPA = 117` 等枚举，但 `SendMsg.createContext` 只暴露 group、private、群临时会话等发送路径。
- 当前代码不能认为完整支持腾讯企点；更准确地说是保留/映射了部分 NTQQ 企点相关底层接口和类型，但没有可用的 OneBot/API/事件业务链路。