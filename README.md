# [webpack-cos-plugin](https://git.io/fjbHm)

[![Travis CI](https://img.shields.io/travis/GuiGu0101/webpack-qcloud-cos-plugin/master.svg)](https://travis-ci.org/GuiGu0101/webpack-qcloud-cos-plugin)
[![node version](https://img.shields.io/node/v/webpack-cos-plugin.svg)](https://nodejs.org)
[![npm version](https://img.shields.io/npm/v/webpack-cos-plugin.svg)](https://www.npmjs.com/package/webpack-cos-plugin)
<span style="position:relative;top: -4px;margin-left: 8px;">
  <a target="_blank" href="https://git.io/fjb1l">更新记录</a>
  <a target="_blank" href="https://git.io/fjbHO">测试报告</a>
  <a target="_blank" href="https://git.io/fjbH3">eslint 报告</a>
</span>
fork from https://travis-ci.org/GuiGu0101/webpack-qcloud-cos-plugin
腾讯云 COS 上传插件, 可在 webpack 打包结束后将来 webpack 生成的文件自动上传到 COS 中。

> **特色**

* 上传文件后, 可删除该文件(默认开启) - 避免将无用文件上传至服务器
* 上传失败后自动重试 (默认重试3次)
* gzip 压缩后再上传 (默认开启)
  * CDN 全局 gzip 是否开启不影响该功能 - 若上传的文件的 header 设置为 gzip, 则 CDN 不会再次对该文件进行 gzip 压缩
* 上传前添加检测机制
  * 若已存在, 则可不重复上传
  * 对于类似 `abc.js` 这种不带 hash 值的文件, 可通过 `existCheck: false` 来取消该功能

> **最佳实践:**

* 将 webpack 生成的所有文件 (除了 html、mainfest 文件外) 自动上传至 COS 中, 并通过腾讯 CDN 来访问上传的文件
* webpack 自动生成的 `js`、`css` 文件设置 `hash` 值, eg: `detail-question.34d71b4.js`
* CDN 可设置为允许使用缓存, 提高加载速度
* 个人服务器或企业服务器只发布 `html` 文件
* 体积较小的图片(300KB 以下)直接使用 webpack 的 `url-loader` 自动打包到 js 文件中(base64)
* 体积较大的图片(300KB 以上)手动上传至 COS 中, 或通过服务端接口自动上传至 COS
* COS 安全: 使用腾讯推荐的访问控制 ACL 来限制某个 `SecretId` 只能访问指定文件夹

## 安装

```bash
npm install -D webpack-cos-plugin
```

## 使用示例

> **注意:** 需修改 `webpackConfig.output.publicPath` 为 cos 路径对应的访问 url, eg: `'//res.example.com/auto_upload_ci/your-project-name/'`

> 用法一: 结合环境变量 (**推荐**)

```javascript
// 先在 CI 的构建环境中设置以下环境变量:

// WEBPACK_QCCOS_PLUGIN_SECRET_ID 对应配置项 SecretId
// WEBPACK_QCCOS_PLUGIN_SECRET_KEY 对应配置项 SecretKey
// WEBPACK_QCCOS_PLUGIN_BUCKET 对应配置项 bucket
// WEBPACK_QCCOS_PLUGIN_REGION 对应配置项 region
// (可选, 默认为 'auto_upload_ci') WEBPACK_QCCOS_PLUGIN_COS_BASE_DIR 对应配置项 cosBaseDir

const QcCOSPlugin = require('webpack-cos-plugin')

webpackConfig.plugins.push(new QcCOSPlugin())
```

> 用法二: 设置配置项

```javascript
const QcCOSPlugin = require('webpack-cos-plugin')

webpackConfig.plugins.push(new QcCOSPlugin({
  auth: {
    SecretId: '', // 在腾讯 COS 控制台获取
    SecretKey: '', // 在腾讯 COS 控制台获取 
  },
  bucket: {
    Bucket: "ap-guangzhou", // COS 服务节点, 示例: oss-cn-hangzhou
    Region: "sp-124879"// COS 存储空间, 在腾讯 COS 控制台获取
  },
  cosBaseDir: 'auto_upload_ci',
  project: 'my-project-name', // 项目名(用于存放文件的直接目录)
}))
```

## 参数说明

> 为了防止 COS 的 SecretId 及 SecretKey 被提交到代码仓库, 本插件提供了环境变量的支持, 所有参数及其环境变量对应关系参考下表。
> 构建函数中的参数优先级大于环境变量

> 参数列表如下所示:

构造参数 | 环境变量 | 默认值 | 说明 |
---  | --- | --- | --- |
SecretId | `WEBPACK_QCCOS_PLUGIN_SECRET_ID` | 空 | COS 访问 id |
SecretKey | `WEBPACK_QCCOS_PLUGIN_SECRET_KEY` | 空 | COS 访问 key |
Bucket | `WEBPACK_QCCOS_PLUGIN_BUCKET` | 空 | COS 存储空间 |
Region | `WEBPACK_QCCOS_PLUGIN_REGION` | 空 | COS 服务节点 |
exclude | - | `/.*\.html$/` | 即匹配该正则的文件名 不会被上传到 COS |
retry | - | 3 | 上传失败后重试次数, 0 代表不重试 |
gzip | - | `true` | 是否在上传前进行 gzip 压缩 |
existCheck | - | `true` | 上传前是否先检测已存在(已存在则不重复上传, 不存在才进行上传) |
enableLog | `WEBPACK_QCCOS_PLUGIN_ENABLE_LOG` | false | 是否输出详细的日志信息 |
ignoreError | `WEBPACK_QCCOS_PLUGIN_IGNORE_ERROR` | false | 上传过程中出现错误是否继续 webpack 构建 |
removeMode | `WEBPACK_QCCOS_PLUGIN_REMOVE_MODE` | true | 生成的文件自动上传至 COS 后, 是否删除本地的对应文件 |
useVersion | `WEBPACK_QCCOS_PLUGIN_USE_VERSION` | false | 是否在目录中添加版本号 |
cosBaseDir | `WEBPACK_QCCOS_PLUGIN_COS_BASE_DIR` | `auto_upload_ci` | COS 中存放上传文件的一级目录名 |
project | - | 默认会自动读取 `package.json` 中的 `name` | COS 中存放上传文件的二级目录, 一般为项目名 |
version | - | 默认会自动读取 `package.json` 中的 `version` | COS 中存放上传文件的三级目录, 一般为版本号，仅在`useVersion`为`true`下有用 |
options | - | undefined | 对象类型. [可用于设置文件的请求头、超时时间等](https://cloud.tencent.com/document/product/436/36119#.E5.8F.82.E6.95.B0.E8.AF.B4.E6.98.8E2) |

* ignoreError: 如果上传过程中出现错误是否继续 webpack 构建
  - true: 忽略错误, 继续构建, webpack 不会报错
  - false: 中止构建, webpack 构建会以失败结束
* removeMode: 生成的文件自动上传至 COS 后, 是否删除本地的对应文件
  - true: 删除 (默认删除)
  - false: 不删除
