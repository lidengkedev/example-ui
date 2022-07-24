# 使用 vitejs 和 TypeScript 从零搭建一个 vue3 组件库

## 使用 pnpm 搭建一个 monorepo 环境

### 什么是 monorepo

就是指在一个大的项目仓库中，管理多个模块/包（package），这种类型的项目大都在项目根目录下有一个packages文件夹，分多个项目管理。就是单仓库 多项目。

### 安装 pnpm

```bash
$ npm install -g pnpm
```
### 初始化项目

```bash
$ pnpm init
```
### 配置 .npmrc 文件

```
phantomjs_cdnurl=http://cnpmjs.org/downloads
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
registry=https://registry.npm.taobao.org
shamefully-hoist=true
```

如果 `shamefully-hoist` 不设置为 true ，会出现找不到依赖包的情况。

### 配置 monorepo 环境

#### 新建 pnpm-workspace.yaml

```yaml
packages:
    - 'packages/**'
    - 'examples'
```
#### 安装依赖

发环境中的依赖一般全部安装在整个项目根目录下，方便下面我们每个包都可以引用,所以在安装的时候需要加个 -w

```bash
$ pnpm i vue@next typescript less -D -w
```
#### 配置 tsconfig.json

```bash
# 执行命令行会自动生成一个 tsconfig.json 文件
$ npx tsc --init
```

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "jsx": "preserve",
    "strict": true,
    "target": "ES2015",
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "moduleResolution": "Node",
    "lib": ["esnext", "dom"]
  }
}
```


## 使用 vitejs 搭建一个 vue3 脚手架项目

### 初始化仓库

```bash
$ cd examples && pnpm init

$ pnpm install vite @vitejs/plugin-vue -D -w
```
### 新建 vite.config.ts 文件

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [vue()]
})
```

### 新建 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="app"></div>
    <script src="main.ts" type="module"></script>
</body>
</html>
```
### 新建 app.vue

```html
<template>
    <div>
        启动调试
    </div>
</template>
```
### 新建 main.ts

此处会出现找不到依赖性 app.vue，因为 ts 找不到对应的类型声明，所以需要建一个 types 文件夹来处理这些 .vue 文件
```ts
import { createApp } from 'vue'

import App from './app.vue'

const app = createApp(App)

app.mount('#app')
```
### 新建 vue-shim.d.ts
新建文件夹 types ，并新建一个 vue-shim.d.ts 文件，用来声明 .vue 文件的对应类型声明
```ts
declare module '*.vue' {
    import type { DefineComponent } from 'vue'
    const component: DefineComponent<{}, {}, any>
}
```
### 启动

在 examples/packages.json 文件中 添加启动脚本
```json
{
    "scripts": {
        "dev": "vite"
    }
}
```
启动
```bash
$ pnpm run dev
```

## 开发调试 UI 库

### 工具包文件

新建一个文件夹 packages/utils ， 然后初始化 packages/utils 文件夹

```bash
$ pnpm init
```

更改工具包 package.json 的项目名称和入口文件
```json
{
    "name": "@example-ui/utils",
    "main": "index.ts"
}
```

### 组件库文件

新建文件夹 packages/components 并初始化文件夹

```bash
$ pnpm init
```
更改组件库 package.json 的名称和入口文件
```json
{
    "name": "example-ui",
    "main": "index.ts"
}
```

### 新建组件库的 入口文件

新建文件 index.ts

注意：这个时候引入的 @example-ui/utils 是无法被找到的。
```ts
import { testfun } from '@example-ui/utils'
const result = testfun(1,2)
console.log(testfun)
```
由于组件库是基于ts的，所以需要安装esno来执行ts文件便于测试组件之间的引入情况。

```bash
$ npm i esno -g
```
然后进入 packages/components 文件夹中安装工具包
```bash
$ pnpm install @example-ui/utils
```
安装完成之后就可以正常的引入 @example-ui/utils 工具包了，而且在 packages/components/package.json 中可以看到 @example-ui/utils 已经被安装了。
因为pnpm是由workspace管理的，所以有一个前缀workspace可以指向utils下的工作空间从而方便本地调试各个包直接的关联引用。

### 开发组件

在 components 文件中新建文件文件

- packages/components/src/button/button.vue
- packages/components/src/button/types.ts
- packages/components/src/button/index.ts
- packages/components/src/icon/
- packages/components/src/index.ts

#### 编写 button.vue 组件

> packages/components/src/button/button.vue

```html
<template>
    <button>TEST</button>
</template>
```

> packages/components/button/types.ts

```ts
import { ExtractPropTypes } from 'vue'
export const ButtonType = ['default', 'primary', 'success', 'warning', 'danger']
export const ButtonSize = ['large', 'normal', 'small', 'mini']
export const buttonProps = {
    type: {
        type: String,
        values: ButtonType
    },
    size: {
        type: String,
        values: ButtonSize
    }
}
export type ButtonProps = ExtractPropTypes<typeof buttonProps>
```

> packages/components/src/button/index.ts

```ts
import Button from './button.vue'
import type { App, Plugin } from "vue"
type SFCWithInstall<T> = T&Plugin
// 要使用app.use函数的话,需要让每个组件都提供一个install方法，app.use()的时候就会调用这个方法
// withInstall方法也可以做个公共方法放到工具库里
const withInstall = <T>(comp:T) => {
    (comp as SFCWithInstall<T>).install = (app:App)=>{
        // 注册组件
        app.component((comp as any).name,comp)
    }
    return comp as SFCWithInstall<T>
}
const Button = withInstall(button)
export default Button
```

> packages/components/src/index.ts

```ts
import Button from './button'
export {
    Button
}
```
### 在 vue3 项目中使用 button 组件

#### 安装组件库

```bash
$ cd examples && pnpm install example-ui
```
#### 在 app.vue 中使用 button 组件
```html
<template>
    <div>
        <Button />
    </div>
</template>
<script lang="ts" setup>
    import { Button } from 'example-ui'
</script>
```

## 使用 vitejs 打包发布自己的 ui 库

### vite 打包配置

打包有 cjs(CommonJS) 和 esm(ESModule) 两种形式, cjs 模式主要用于服务端引用(ssr),而 esm 就是现在经常使用的方式，它本身自带 treeShaking 而不需要额外配置按需引入(前提是将模块分别导出)。

> packages/components/vite.config.ts

```ts
import { defineConfig } from "vite"
import vue from "@vitejs/plugin-vue"
export default defineConfig({
    build: {
        target: 'modules',
        //打包文件目录
        outDir: "es",
        //压缩
        minify: false,
        //css分离
        //cssCodeSplit: true,
        rollupOptions: {
            external: ['vue'],
            input: ['src/index.ts'],
            output: [
                {
                    format: 'es',
                    //不用打包成.es.js,这里我们想把它打包成.js
                    entryFileNames: '[name].js',
                    //让打包目录和我们目录对应
                    preserveModules: true,
                    //配置打包根目录
                    dir: 'es',
                    preserveModulesRoot: 'src'
                }, {
                    format: 'cjs',
                    entryFileNames: '[name].js',
                    //让打包目录和我们目录对应
                    preserveModules: true,
                    //配置打包根目录
                    dir: 'lib',
                    preserveModulesRoot: 'src'
                }
            ]
        },
        lib: {
            entry: './index.ts',
            formats: ['es', 'cjs']
        }
    },
    plugins: [vue()]
})
```
### 执行打包

在 packages/components 文件夹中执行如下命令

```bash
$ pnpm run build
```
### 向打包文件中加入声明文件

需要引入 vite-plugin-dts

```bash
$ pnpm install vite-plugin-dts -D -w
```

然后修改 vite.config.ts

```ts
import dts from 'vite-plugin-dts'
export default defineConfig({
    build: {...},
    plugins: [
        vue(),
        dts({
            //指定使用的tsconfig.json为我们整个项目根目录下掉,如果不配置,你也可以在components下新建tsconfig.json
            tsConfigFilePath: '../../tsconfig.json'
        }),
        //因为这个插件默认打包到es下，我们想让lib目录下也生成声明文件需要再配置一个
        dts({
            outputDir:'lib',
            tsConfigFilePath: '../../tsconfig.json'
        })
    ]
})
```
发布之前更改一下 packages/components/package.json
```json
{
    "main": "lib/index.js",
    "module":"es/index.js",
    "scripts": {
        "build": "vite build"
    },
    "files": ["es", "lib"],
    "keywords": ["example-ui", "vue3 组件库"],
    "license": "MIT",
    "typings": "lib/index.d.ts"
}
```

- module: 组件库默认入口文件是传统的CommonJS模块,如果环境支持ESModule的话，构建工具会优先使用module入口
- files: 是指需要发布到npm上的目录，因为不可能components下的所有目录都被发布上去

### 把样式拆分打包

我们需要的组件库是每个css样式放在每个组件其对应目录下，这样就不需要每次都全量导入我们的css样式。

#### 处理less文件

新建文件 packages/components/build/buildLess.ts

安装依赖

- cpy: 它可以直接复制我们规定的文件并将我们的文件copy到指定目录


```bash
$ pnpm install cpy fast-glob -D -w
```

```ts
import cpy from 'cpy'
import { resolve } from 'path'

const sourceDir = resolve(__dirname, '../src')
//lib文件
const targetLib = resolve(__dirname, '../lib')
//es文件
const targetEs = resolve(__dirname, '../es')
console.log(sourceDir);
const buildLess = async () => {
    await cpy(`${sourceDir}/**/*.less`, targetLib)
    await cpy(`${sourceDir}/**/*.less`, targetEs)
}
buildLess()
```
在package.json中新增命令
```json
{
    "scripts": {
        "build": "vite build",
        "build:less": "esno build/buildLess"
    }
}
```
在ts中引入less因为它本身没有声明文件所以会出现类型错误,所以需要安装 @types/less

```bash
$ pnpm install --save-dev @types/less -D -w
```
修改 buildLess.ts 文件

```ts
import cpy from 'cpy'
import { resolve, dirname } from 'path'
import { promises as fs } from "fs"
import less from "less"
import glob from "fast-glob"
const sourceDir = resolve(__dirname, '../src')
//lib文件目录
const targetLib = resolve(__dirname, '../lib')
//es文件目录
const targetEs = resolve(__dirname, '../es')

//src目录
const srcDir = resolve(__dirname, '../src')

const buildLess = async () => {
    //直接将less文件复制到打包后目录
    await cpy(`${sourceDir}/**/*.less`, targetLib)
    await cpy(`${sourceDir}/**/*.less`, targetEs)

    //获取打包后.less文件目录(lib和es一样)
    const lessFils = await glob("**/*.less", { cwd: srcDir, onlyFiles: true })

    //遍历含有less的目录
    for (let path in lessFils) {

        const filePath = `${srcDir}/${lessFils[path]}`
        //获取less文件字符串
        const lessCode = await fs.readFile(filePath, 'utf-8')
        //将less解析成css

        const code = await less.render(lessCode, {
            //指定src下对应less文件的文件夹为目录
            paths: [srcDir, dirname(filePath)]
        })

        //拿到.css后缀path
        const cssPath = lessFils[path].replace('.less', '.css')

        //将css写入对应目录
        await fs.writeFile(resolve(targetLib, cssPath), code.css)
        await fs.writeFile(resolve(targetEs, cssPath), code.css)
    }
}
buildLess()
```

修改 vite.config.ts 文件

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    build: {...},
    plugins: [
        ...,
        {
            name: 'style',
            generateBundle(config, bundle) {
                //这里可以获取打包后的文件目录以及代码code
                const keys = Object.keys(bundle)

                for (const key of keys) {
                    const bundler: any = bundle[key as any]
                    //rollup内置方法,将所有输出文件code中的.less换成.css,因为我们当时没有打包less文件
                    this.emitFile({
                        type: 'asset',
                        fileName: key,//文件名名不变
                        source: bundler.code.replace(/\.less/g, '.css')
                    })
                }
            }
        }
    ]
})
```
修改 packages/components/packages.json 

```json
{
    "scripts": {
        "build": "vite build & esno build/buildLess"
    }
}
```

执行命令
```bash
$ pnpm run build
```