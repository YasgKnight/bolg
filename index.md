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




## Hellow-World
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
 
* $schema属性指明了检验的schema
* schematics属性是一个对象，里面每一个键都是schematic的name
* 每个schematic都有description属性来描述这个schematic
* 每个schematic也有一个factory属性，它是一个string，指明了我们的schematic的入口点的文件位置，之后是一个#，然后是要调用的函数。在此处我们会调用在hello-world/index.js文件中的helloWorld()函数
* 可选的aliases属性是一个数组，我们能用来指定我们的schematic的一个或者多个别名。举个例子，在Angular CLI中"generate" schematic的别名是"g"。这允许我们通过$ ng g调用generate命令*
* 可选的schema属性可用来指明每个单独schematic的schema和所有的可用的命令行选项

src/collections.json
```
{
  "name": "hello-world",
  "version": "0.0.0",
  "description": "A blank schematics",
  "scripts": {
    "build": "tsc -p tsconfig.json",
    "test": "npm run build && jasmine src/**/*_spec.js"
  },
  "keywords": [
    "schematics"
  ],
  "author": "",
  "license": "MIT",
  "schematics": "./src/collection.json",
  "dependencies": {
    "@angular-devkit/core": "^7.0.2",
    "@angular-devkit/schematics": "^7.0.2",
    "@types/jasmine": "^2.6.0",
    "@types/node": "^8.0.31",
    "jasmine": "^2.8.0",
    "typescript": "^2.5.2"
  }
}
```

## 入口函数
src/hello-world/index.ts文件
```
import { Rule, SchematicContext, Tree } from '@angular-devkit/schematics';

// You don't have to export the function as default. You can also have more than one rule factory
// per file.
export function helloWorld(_options: any): Rule {
  return (tree: Tree, _context: SchematicContext) => {
    console.log(_options)
    tree.create("hello-world.html",`<h1>hello ${_options}</h1>`);
    return tree;
  };
}
```

* 函数接受一个options参数，它是命令行参数的键值对对象
* 这个函数是一个高阶函数，接受或者返回一个函数引用。此处，这个函数返回一个接受Tree和SchmaticsContext对象的函数

### Tree
我们能使用tree完成这些事情：
* read(path: string): Buffer | null: 读取指定的路径
* exists(path: string): boolean: 确定路径是否存在
* create(path: string, content: Buffer | string): void: 在指定路径使用指定内容创建新文件
* beginUpdate(path: string): UpdateRecorder: 为在指定路径的文件返回一个新的UpdateRecorder实例
* commitUpdate(record: UpdateRecorder): void: 提交UpdateRecorder中的动作

### Rule
```
declare type Rule = 
 (tree: Tree, context: SchematicContext) => 
 Tree | Observable<Tree> | Rule | void;
```



 
 

