# guoapp 站源模块模板

用这个模板新建仓库，写一个 guoapp 的站源模块。模板里是只返回假数据的示例，不访问网络。

## 开始

1. 在 GitHub 上点 Use this template 新建仓库，仓库名就是模块标识（小写字母、数字、`_`、`-`，最长 32 个字符）。
2. 改 `manifest.json`：`id` 改成仓库名，填 `name`、`description`、`adult`（是否限制级内容）、`hosts`（模块允许请求的域名）和 `capabilities`。
3. 改 `main.js`。清单字段、导出函数和宿主接口见 [guoapp 开发指南的「站源模块」](https://github.com/Xanerix/guoapp/blob/main/DEVELOPMENT.md#站源模块)。
4. 本机调试：把模块文件夹放进应用数据目录的 `modules/` 下，或在 guoapp 的 `native` 目录执行 `go run ./cmd/moduletool <模块目录> <请求 JSON>`。

## 发版

1. 改 `manifest.json` 的 `version`（格式 `1.2.3`），提交。
2. 打标签并推送：`git tag v1.2.3 && git push origin main v1.2.3`。
3. CI 校验通过后自动发布 Release，附件是可直接安装的 `<标识>-<版本>.zip`。

每次推送和 PR 也会跑同样的校验：清单格式、仓库名与标识一致、`main.js` 语法，以及不接受压缩或混淆过的代码（单行不超过 1000 个字符）。

## 上架

模块要出现在应用内的模块商店，需在 [index](https://github.com/guoapp-modules/index) 申请，审核通过后转入 guoapp-modules 组织再上架，流程见那里的说明。未上架的模块仍可用 Release 里的 zip 或其地址手动安装。
