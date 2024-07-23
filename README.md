# Vue 3 + Vite

## 浏览器兼容

build.target
类型： string | string[]
默认： 'modules'
相关内容： 浏览器兼容性
1. 支持原生 ES 模块、原生 ESM 动态导入 和 import.meta 的浏览器（现代浏览器）
2. target选项为modules 为 ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14']
3. target选项为esnext并且 build.minify 选项为 'terser'，安装的 Terser 版本小于 5.16.0，'esnext' 将会强制降级为 'es2021'，其他情况下将完全不会执行转译

两种针对旧版本兼容
1. polyfill 包的服务 可以部署私服
2. 通过插件 [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) 来支持，将自动生成传统版本的 chunk 及与其相对应 ES 语言特性方面的 polyfill。兼容版的 chunk 只会在不支持原生 ESM 的浏览器中进行按需加载。
3. 针对多版本或者跨内核的浏览器建议使用 renderModernChunks 配置为false保证每个polyfill支持目标 browsers
4. renderModernChunks 默认为true 可能会为了兼容除目标之外的浏览器browsers导致目标browsers出现问题

兼容思路
> repository 的 demo 可以自己测试一下用 serve cli 启动端口验证是否兼容成功
> npm i serve -g
> serve dist
1. 确认好目标browsers范围 比如[es6](https://caniuse.com/es6)基本上是 vite 默认兼容 polyfill 概念上的现代浏览器，vite只处理语法转译，且不包含任何 polyfill
2. 在兼容后发现报错例如 hasOwn is not a function，
3. 除了特殊情况，vite在旧版本完全兼容不了需要配置 polyfill外（大多数现代浏览器都不需要！），因为core-js@3它支持所有前沿功能，因此在 polyfill 包含方面非常积极。即使以原生 ESM 支持为目标，它也会注入 15kb 的 polyfill！
4. 如果不依赖前沿的运行时功能，那么完全避免在现代版本中使用 polyfill 并不难。使用按需服务[polyfill](https://cdnjs.cloudflare.com/polyfill/)可以部署私服会比较保险。

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import legacy from '@vitejs/plugin-legacy'
export default defineConfig({
  plugins: [vue(),legacy({
    targets: ['chrome >= 87, firefox >= 78, edge >= 88'],
    additionalLegacyPolyfills: ['regenerator-runtime/runtime'],
    renderModernChunks:false
  })],
})
```

### 例子
hasOwn chrome 92版本才可以使用
  https://caniuse.com/?search=hasown

这个例子设置为 true会出现 hasOwn is not a function
```js
if(Object.hasOwn({a:1},'a')){
    console.log([123].at(0),Object.keys({a:1}).at(0));
  }
```

浏览器兼容是一个费力费时漫长的过程，有遇到其他的问题欢迎一起讨论！