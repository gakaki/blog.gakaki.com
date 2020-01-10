---
title: Wasmtime WebAssembly 教程
date: 2019-12-10 19:20:18
categories:
- Web
tags:
- Web
---

WASM 成为 HTML、CSS 与 JS 之后的第 4 门 Web 语言。
Mozilla、Fastly、Intel 与 Red Hat 宣布成立联合组织 Bytecode Alliance（字节码联盟）

## 什么是WASI 
    
官方解释： WASI WebAssembly 系统接口。它提供了一种方法，可以将不同的模块彼此隔离，并赋予它们对文件系统特定部分和其它资源的细粒度权限，以及对不同系统调用的细粒度权限。


Wasmtime A small and efficient runtime for WebAssembly & WASI

我的理解是把WebAssembly这种二进制指令格式拓展到了后端领域，这样Webassebly的二进制就可以同时在
前端浏览器和后端的世界里运行了，一套代码可以同时共享在前后端代码之中了，例如加密算法，例如类似的业务逻辑。Kotlin/Native 也在做类似的这种事情，不过WASI若成为标准的话也是好事。

以下步骤来自wasmtime的官网视频 https://wasmtime.dev

## 拉取Demos

```bash
    git clone git@github.com:bytecodealliance/wasmtime-demos.git
```

## 安装WASM-PACK

```bash
    cd wasmtime-demos/markdown
    curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
    WASM_INTERFACE_TYPES=1 wasm-pack build

    //这样就打包了一个通用的wasm文件了在 pkg/markdown.wasm 
    //copy到外层各种语言的文件夹内
    cp pkg/markdown.wasm ../python
    cp pkg/markdown.wasm ../rust
    cp pkg/markdown.wasm ../node
    cp pkg/markdown.wasm ../webpack
```

## 使用各种后端技术调用 .wasm文件里的方法

### 纯WebAssembly

#### 安装wasmtime

```bash
    curl https://wasmtime.dev/install.sh -sSf | bash
```

#### 运行

```bash
    wasmtime markdown.wasm --invoke render '# 你好，WebAssembly\!'

    warning: using `--invoke` with a function that takes arguments is experimental and may break in the future
    warning: using `--invoke` with a function that returns values is experimental and may break in the future
    <h1>你好，WebAssembly!</h1>
```

### Python3

```bash
    cd python
    pip install wasmtime 或 pip3 install wasmtime
    //运行
    python python/run.py
    //结果
    //<h1>Hello! Python</h1>
```

### Rust

```bash
    cd rust
    //运行 编译巨慢
    cargo run --release
    //结果
    //<h1>Hello! Rust</h1>
```

### Node

```bash
    cd nodejs && npm install
    //运行 
    node --experimental-wasm-mv --experimental-modules --loader ./loader.mjs ./run.mjs
    //结果
    (node:40672) ExperimentalWarning: The ESM module loader is experimental.
    (node:40672) ExperimentalWarning: --experimental-loader is an experimental feature. This feature could change at any time
    <h1>Hello, node!</h1>
```

### Web Chrome浏览器

```bash
    cd webpack && npm install && npm start
    //自动打开http://localhost:8081/
    //about:flags 里的 Experimental WebAssembly 需要设置为Enable 然后Relaunch才能看到效果
```
<!-- ### 另外还支持.Net平台哦，就差Java和Dart和GoLang了 -->

