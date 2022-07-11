---
date: 2022-01-21
title: 微信小程序开发指南-基本配置篇
tags:
  - 2022
  - 小程序
describe: 基于TS原生微信小程序的基本配置
---

## 基本配置

### tsconfig.json

由于选择使用 ts 进行开发，所以要对 ts 编译进行配置，可以参考以下配置。

```json
//配置文档 https://www.tslang.cn/docs/handbook/tsconfig-json.html
{
  "compilerOptions": {
    "strictNullChecks": true, //在严格的 null检查模式下， null和 undefined值不包含在任何类型里，只允许用它们自己和 any来赋值（有个例外， undefined可以赋值到 void）。
    "noImplicitAny": true, //在表达式和声明上有隐含的 any类型时报错。
    "module": "CommonJS", //指定生成哪个模块系统代码： "None"， "CommonJS"， "AMD"， "System"， "UMD"， "ES6"或 "ES2015"。
    "target": "ES5", //指定ECMAScript目标版本 "ES3"（默认）， "ES5"， "ES6"/ "ES2015"， "ES2016"， "ES2017"或 "ESNext"。
    "allowJs": false, //允许编译javascript文件。
    "experimentalDecorators": true, //启用实验性的ES装饰器。
    "noImplicitThis": true, //当 this表达式的值为 any类型的时候，生成一个错误。
    "noImplicitReturns": true, //不是函数的所有返回路径都有返回值时报错。（函数的所有情况下都确保要有返回值）
    "alwaysStrict": true, //以严格模式解析并为每个源文件生成 "use strict"语句
    "inlineSourceMap": true, //生成单个sourcemaps文件，而不是将每sourcemaps生成不同的文件。
    "inlineSources": true, //将代码与sourcemaps生成到一个文件中，要求同时设置了 --inlineSourceMap或 --sourceMap属性。
    "noFallthroughCasesInSwitch": true, //报告switch语句的fallthrough错误。（即，不允许switch的case语句贯穿）
    "noUnusedLocals": true, //若有未使用的局部变量则抛错。
    "noUnusedParameters": true, //若有未使用的参数则抛错。
    "strict": true, //启用所有严格类型检查选项。启用 --strict相当于启用 --noImplicitAny, --noImplicitThis, --alwaysStrict， --strictNullChecks和 --strictFunctionTypes和--strictPropertyInitialization。
    "removeComments": true, //删除所有注释，除了以 /!*开头的版权信息。
    "pretty": true, //给错误和消息设置样式，使用颜色和上下文。
    "strictPropertyInitialization": true, //确保类的非undefined属性已经在构造函数里初始化。若要令此选项生效，需要同时启用--strictNullChecks。
    "lib": ["es6"], //编译过程中需要引入的库文件的列表。
    "typeRoots": [
      //要包含的类型声明文件路径列表。默认所有可见的"@types"包会在编译过程中被包含进来。 node_modules/@types文件夹下以及它们子文件夹下的所有包都是可见的； 也就是说， ./node_modules/@types/，../node_modules/@types/和../../node_modules/@types/等等。如果指定了typeRoots，只有typeRoots下面的包才会被包含进来。
      "./typings" //这个配置文件会包含所有./typings下面的包，而不包含./node_modules/@types里面的包。
    ]
    //如果指定了types，只有被列出来的包才会被包含进来。 比如：
    //"types" : ["node", "lodash", "express"]//这个tsconfig.json文件将仅会包含 ./node_modules/@types/node，./node_modules/@types/lodash和./node_modules/@types/express。/@types/。 node_modules/@types/*里面的其它包不会被引入进来。
    //指定"types": []来禁用自动引入@types包。
    //注意，自动引入只在你使用了全局的声明（相反于模块）时是重要的。 如果你使用 import "foo"语句，TypeScript仍然会查找node_modules和node_modules/@types文件夹来获取foo包。
  },
  "include": ["./**/*.ts"],
  "exclude": ["node_modules"],
  "compileOnSave": true
}
```

### project.config.json

该文件是针对一些个性化配置，例如界面颜色、编译配置等等，这样当你换了另外一台电脑重新安装工具的时候，不必重新配置。
[全部配置](https://developers.weixin.qq.com/miniprogram/dev/devtools/projectconfig.html)

```json
{
  "description": "项目配置文件",
  "packOptions": {
    //用以配置项目在打包过程中的选项
    "includes": [
      //用以配置打包时需要强制带上的文件或者文件夹，匹配的这些文件或文件夹将一定会出现在预览或上传的结果内。优先级高于ignore字段
      {
        "type": "file",
        "value": "test/test.js"
      },
      {
        "type": "folder",
        "value": "test"
      },
      {
        "type": "suffix",
        "value": ".webp"
      },
      {
        "type": "prefix",
        "value": "test-"
      },
      {
        "type": "glob",
        "value": "test/**/*.js"
      },
      {
        "type": "regexp",
        "value": "\\.jsx$"
      }
    ],
    "ignore": [
      {
        //用以配置打包时对符合指定规则的文件或文件夹进行忽略，以跳过打包的过程，这些文件或文件夹将不会出现在预览或上传的结果内。
        "type": "file",
        "value": "test/test.js"
      }
    ]
  },
  "miniprogramRoot": "miniprogram/",//指定小程序源码的目录(需为相对路径)
  "compileType": "miniprogram",//编译类型
  "libVersion": "2.20.1",//基础库版本
  "projectname": "speedtest",//项目名字，只在新建项目时读取
  "scripts": {//自定义预处理
    "beforeCompile": "npm run tsc",//编译前预处理命令
    "beforePreview": "npm run tsc",//预览前预处理命令
    "beforeUpload": "npm run tsc"//上传前预处理命令
  },
  "setting": {//项目的编译设置
    "urlCheck": true,//是否检查安全域名和 TLS 版本
    "es6": true,//是否启用 es6 转 es5
    "enhance": false,//是否打开增强编译
    "postcss": true,//上传代码时样式是否自动补全
    "preloadBackgroundData": false,//小程序加载时是否数据预拉取
    "minified": true,//上传代码时是否自动压缩
    "newFeature": false,//已废弃
    "coverView": true,//是否使用工具渲染 CoverView
    "nodeModules": false,//已废弃
    "autoAudits": false,//是否自动运行体验评分
    "showShadowRootInWxmlPanel": true,//是否开启调试器 WXML 面板展示 shadow-root
    "scopeDataCheck": false,//是否自动校验结构化数据
    "uglifyFileName": false,//是否进行代码保护
    "checkInvalidKey": true,//是否展示 JSON 文件校验错误信息
    "checkSiteMap": true,//是否打开SiteMap索引提示(（默认为true）
    "uploadWithSourceMap": true,//上传时是否带上 sourcemap（默认为true）
    "compileHotReLoad": false,//是否开启文件保存后自动热重载
    "useMultiFrameRuntime": true,//是否开启模拟器预先载入小程序的某些资源。此设定为 false 时会导致 useIsolateContext 失效
    "useApiHook": true,//是否启用 API Hook 功能
    "useApiHostProcess": true,//是否在额外的进程处理一些小程序 API
    "babelSetting": {//增强编译下Babel的配置项
      "ignore": [],//配置需要跳过Babel编译(包括代码压缩)处理的文件或目录
      "disablePlugins": [],
      "outputPath": ""//Babel 辅助函数的输出目录，默认为 @babel/runtime
    },
    "useIsolateContext": false,//是否开启小程序独立域调试特性
    "userConfirmedBundleSwitch": false,//已废弃
    "packNpmManually": false,//是否手动配置构建 npm 的路径
    "packNpmRelationList": [],//仅 packNpmManually 为 true 时生效，详细参考构建 npm 文档
    "minifyWXSS": true,//上传代码时是否自动压缩样式文件
    "disableUseStrict": false,//是否禁用增强编译的严格模式
    "minifyWXML": true,//上传代码时是否自动压缩 WXML 文件
    "showES6CompileOption": false,//是否在本地设置中展示传统的 ES6 转 ES5 开关
    "useCompilerPlugins": [//编译插件配置(String[] | false)目前支持编译插件有 typescript、less、sass
      "typescript",
      "less"
    ],
    "ignoreUploadUnusedFiles": true//上传时是否过滤无依赖文件（默认为true）
  },
  "simulatorType": "wechat",
  "simulatorPluginLibVersion": {},
  "appid": "wxb49e81abc175bc3c",
  "condition": {}
}
```

### sitemap 配置

小程序内搜索配置，可以通过 sitemap.json 配置，或者管理后台页面收录开关来配置其小程序页面是否允许微信索引。

```json
{
  "rules":[{
    "action": "allow",
    "page": "path/to/page",
    "params": ["a", "b"],
    "matching": "inclusive"
  }, {
    "action": "disallow",
    "page": "*"
  }, {
    "action": "allow",
    "page": "*"
  }]
}
```
### 页面配置

#### 全局页面配置

对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。

详细配置见[链接](https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/app.html)

#### 单个页面配置

每一个小程序页面也可以使用同名 .json 文件来对本页面的窗口表现进行配置，页面中配置项会覆盖 app.json 的 window 中相同的配置项。
完整配置项说明请参考[小程序页面配置](https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/page.html)
