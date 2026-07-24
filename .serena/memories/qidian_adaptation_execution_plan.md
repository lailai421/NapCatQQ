# 腾讯企点适配执行计划归档

- 已创建执行计划文档：`docs/qidian-adaptation-execution-plan.md`。
- 文档按运行时验证、企点 API 封装、接收消息、发送文本消息四条主线拆解。
- 第一轮开发目标：`get_qidian_status`、`qidian_message`、`send_qidian_msg` 纯文本闭环。
- 第一轮明确不做历史消息、已读、图片、文件、语音、富文本、联系人列表同步。
- 关键代码入口：
  - `packages/napcat-core/index.ts`
  - `packages/napcat-core/apis/qidian.ts`，计划新增
  - `packages/napcat-onebot/api/msg.ts`
  - `packages/napcat-onebot/index.ts`
  - `packages/napcat-onebot/action/qidian/GetQiDianStatus.ts`，计划新增
  - `packages/napcat-onebot/action/qidian/SendQiDianMsg.ts`，计划新增
  - `packages/napcat-onebot/action/router.ts`
  - `packages/napcat-onebot/action/index.ts`
- 文档校验：663 行；关键阶段和 action 名称均已命中；不确定表达检查无命中。
- 当前 git 状态包含既有修改 `packages/napcat-core/index.ts`、`.serena/`，以及新增 `docs/`。