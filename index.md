# angular 原理图（Schematic）的学习笔记

### 什么是Collection？
Collection是一系列的schematic。我们会在工程中collection.json中为每个schematic定义元数据。

### 安装
首先，使用npm或者yarn安装schematics的CLI：

```
$ npm install -g @angular-devkit/schematics-cli
$ yarn add -g @angular-devkit/schematics-cli
```
### 生成原理图
```
$ schematics schematic --name demo
```

## hello world
 创建一个空工程
```
$ schematics black --name=hello-world
$ cd hello-world
```
 打开src/collection.json:
 
 ```
 {
  "$schema": "../node_modules/@angular-devkit/schematics/collection-schema.json",
  "schematics": {
    "hello-world": {
      "description": "A blank schematic.",
      "factory": "./hello-world/index#helloWorld"
    }
  }
}
 ```
 
 

