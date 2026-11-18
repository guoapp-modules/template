# guoapp 站源模块模板

用这个模板新建仓库，写一个 guoapp 的站源模块。模板里是只返回假数据的示例，不访问网络。模板以 GPL-3.0 许可发布，上架的模块也须使用 GPL-3.0，保留根目录的 `LICENSE`。

## 开始

1. 在 GitHub 上点 Use this template 新建仓库，仓库名就是模块标识（小写字母、数字、`_`、`-`，最长 32 个字符）。
2. 改 `manifest.json`：`id` 改成仓库名，填 `name`、`description`、`adult`（是否限制级内容）、`hosts`（模块允许请求的域名）和 `capabilities`。
3. 改 `main.js`。清单字段、导出函数和宿主接口见 [guoapp 开发指南的「站源模块」](https://github.com/Xanerix/guoapp/blob/main/DEVELOPMENT.md#站源模块)。
4. 本机调试：把模块文件夹放进应用数据目录的 `modules/` 下，或在 guoapp 的 `native` 目录执行 `go run ./cmd/moduletool <模块目录> <请求 JSON>`。

## 数据格式

导出函数、能力和 `host` 的完整列表见上面的开发指南，这里列出常用的参数和返回值格式。

### 剧集

`catalogPage`、`search` 返回的 `items` 和 `detail` 返回的 `drama`：

| 字段 | 说明 |
| --- | --- |
| `id` | 必填，用 `host.dramaId(sourceId)` 生成，格式为 `<模块标识>:<sourceId>` |
| `source` | 填 `host.source` |
| `sourceId` | 站点里的剧集 ID，`detail(sourceId)` 收到的就是它 |
| `title` | 剧名 |
| `intro` / `desc` | 简介，取第一个非空的 |
| `cover` | 封面地址，须为 http(s)，`//` 开头的会补成 https |
| `totalEpisode` | 总集数 |
| `categoryName` | 分类名 |
| `tags` | 字符串数组 |
| `score` | 评分字符串，十分制，如 `"8.6"` |
| `badge` | 海报角标文字，不填时显示「共 N 集」 |
| `heat` / `views` | 热度、播放量文字 |
| `onlineDate` | 上线日期 |
| `releaseStatus` | `ongoing` 连载、`finished` 完结，不确定时不填 |
| `vip` | 是否会员剧，布尔值 |

### 分集

`detail` 返回的 `chapters`，每项：

| 字段 | 说明 |
| --- | --- |
| `id` | 必填，用 `host.chapterId(sourceId, key)` 生成，格式为 `<模块标识>:<sourceId>:<key>` |
| `source` | 填 `host.source` |
| `title` | 分集标题 |
| `currentEpisode` | 集数，数字或数字字符串 |
| `videoUrl` / `pageUrl` / `referer` | 播放时原样传回，可用来存地址 |
| `vip` | 是否需要会员或解锁 |
| `vertical` | 是否竖屏视频，不确定时不填 |

分集只保留上表字段，其他字段会被丢弃。播放时需要的信息要放进 `id` 的 `key` 或上面几个地址字段里。

### 播放

`resolveMedia(task)` 收到的 `task`：

| 字段 | 说明 |
| --- | --- |
| `dramaId` | 剧集 `id` |
| `dramaTitle` | 剧名 |
| `chapter` | 要播放的分集，字段同上 |
| `index` | 集数，同分集的 `currentEpisode` |
| `downloadQuality` | 下载时用户选的清晰度，0 为自动；播放时为 0 |

返回值：

| 字段 | 说明 |
| --- | --- |
| `url` | 必填，http(s) 播放地址，mp4 或 m3u8 |
| `referer` | 请求视频时带的 Referer |
| `quality` | 清晰度，按画面高度填，如 `1080`、`720`，显示为 `1080P` |
| `durationMs` | 时长，毫秒 |
| `playlist` | m3u8 文本；站点的播放列表要自己处理时，把改好的文本放这里，`url` 作为相对地址的基准 |
| `hlsKey` | HLS 的 16 字节 AES 密钥，Base64 字符串，替换播放列表里 `EXT-X-KEY` 指向的密钥 |
| `cencKey` | CENC 加密 mp4 的密钥，Base64 字符串 |
| `credentials` | 请求视频时附加的 `{ cookie, origin, referer, userAgent, expiresMs, omitFetchMetadata }`；`expiresMs` 是过期时刻的毫秒时间戳，过期后这组地址作废，需要重新解析播放；`omitFetchMetadata` 为 `true` 时请求视频不带 `Sec-Fetch-Mode`、`Sec-Fetch-Dest`，用于见到这两个头就断开连接的 CDN |
| `variants` | 其他线路或清晰度，数组，每项字段同上 |

多线路：把所有地址放进 `variants`，同一清晰度可以有多条。应用按地址去重，按 `quality` 从高到低排序，同一清晰度保持原顺序；当前线路播放失败时自动换下一条。不同的 `quality` 会出现在清晰度菜单里，用户选定清晰度后只在该清晰度的线路之间切换。

```js
resolveMedia(task) {
  const lines = [
    { url: "https://a.example.com/1.m3u8", quality: 1080 },
    { url: "https://b.example.com/1.m3u8", quality: 1080 },
    { url: "https://a.example.com/1-720.m3u8", quality: 720 },
  ];
  return Object.assign({}, lines[0], { variants: lines });
}
```

播放地址不受清单 `hosts` 限制，`hosts` 只管模块自己发出的请求。

### 请求

`host.fetch(request)`：

| 字段 | 说明 |
| --- | --- |
| `url` | 必填，域名须在清单 `hosts` 里 |
| `method` | 默认 `GET` |
| `headers` | 请求头对象 |
| `body` | 请求体字符串 |
| `timeoutMs` | 超时，毫秒 |
| `limit` | 响应体上限，字节 |
| `followRedirects` | 是否跟随重定向 |
| `anyStatus` | 任何状态码都按正常响应返回 |
| `acceptStatus` | 按正常响应返回的错误状态码数组，响应体须是 JSON |
| `direct` | 改用不带浏览器指纹的普通连接 |
| `ticket` | `host.reserve` 返回的限流名额 |
| `background` | 标记为后台请求 |

返回 `{ status, headers, body, blocked, oversized }`，`headers` 的每个值是字符串数组，`body` 是字符串。请求失败时抛出的错误带 `status`、`body`、`blocked`、`backoff`、`oversized` 属性。

`host.fetchText({ url, referer, userAgent, cookie, noCache, body, mirrors })` 返回 `{ body, url }`，`url` 是实际响应的地址，重定向或换用候选地址后与请求的不同；`mirrors` 是主地址失败后依次尝试的候选地址。

## 发版

1. 改 `manifest.json` 的 `version`（格式 `1.2.3`），提交。
2. 打标签并推送：`git tag v1.2.3 && git push origin main v1.2.3`。
3. CI 校验通过后自动发布 Release，附件是可直接安装的 `<标识>-<版本>.zip`。

每次推送和 PR 也会跑同样的校验：清单格式、仓库名与标识一致、`main.js` 语法，以及不接受压缩或混淆过的代码（单行不超过 1000 个字符）。

## 上架

模块要出现在应用内的模块商店，需在 [index](https://github.com/guoapp-modules/index) 申请，审核通过后转入 guoapp-modules 组织再上架，流程见那里的说明。未上架的模块仍可用 Release 里的 zip 或其地址手动安装。
