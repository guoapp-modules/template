# CLAUDE.md

本仓库是 guoapp 的一个 JS 站源模块：`manifest.json` 与 `main.js`。清单字段、导出函数和宿主接口见 guoapp 仓库 `DEVELOPMENT.md` 的「站源模块」。

- 运行环境是 goja，不是浏览器也不是 Node：没有 `fetch`、`URL`、`Buffer`、`atob`、`TextEncoder`，函数必须同步返回。网络、加密、解析都走全局对象 `host`。
- 不写压缩或混淆过的代码，单行不超过 1000 个字符，CI 会拒绝。
- 剧和分集的 ID 会写进用户的追剧、记录和下载，发布后不能改格式。
- 发版：改 `manifest.json` 的 `version`，打 `v<版本号>` 标签推送，CI 发布 Release。
- 本机校验：`python3 <guoapp-modules/.github 仓库>/scripts/check_module.py . --repo <仓库名>`。

## 站点访问规定

- 不得在会话里以任何形式直接访问站点，包括 `moduletool`、curl 和会联网的测试。
- 需要站点的真实数据时，把探测脚本写在临时目录，回复里只给一行执行命令，由用户在会话外运行、审查后把结果贴回对话。探测脚本的输出只打印到控制台，不写入文件，并对内容打码。
- 不得编写或运行会输出站点内容的测试。
