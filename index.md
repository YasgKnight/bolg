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
| Rule | 描述 |
|---|---|
| `noop()` | 按Tree原样返回输入。 |
| `chain(rules: Rule[])` | 返回Rule与其他Rules 串联的a  |
| `forEach(op: FileOperator)` | 返回一个Rule将运算符应用于input的每个文件的Tree。 |
| `move(root: string)` | 将所有文件从输入移动到子目录 |
| `merge(other: Tree)` | 将输入Tree与另一个合并Tree |
| `contentTemplate<T>(options: T)` |将内容模板（请参见“模板”部分）应用于整个模板Tree |
| `pathTemplate<T>(options: T)` | 将路径模板（请参阅模板部分）应用于整个Tree. |
| `template<T>(options: T)` | 将路径和内容模板（请参阅“模板”部分）应用于整个Tree. |
| `filter(predicate: FilePredicate<boolean>)` | 返回Tree包含未通过的文件的输入FilePredicate. |


## 模板化 
某些功能基于文件模板系统，该系统由路径和内容模板组成。

系统在文件中定义的占位符或加载到中的路径上操作，Tree并使用传入Rule模板的值（即template<T>(options: T)），按照以下定义填充这些占位符。

## 路径模板
| 占位符 | 描述 |
|---|---|
| __variable__ | 替换为的值variable |
| __variable@function__ | 替换为调用结果function(variable)。可以链接到左侧（__variable@function1@function2__ 等）。  |

## 内容模板
| 占位符 | 描述 |
|---|---|
| <%= expression %> | 用给定表达式的调用结果替换。这仅支持直接表达式，不支持结构化（for / if / ...）JavaScript。 |
| <%- expression %> | 与上面相同，但是插入时结果的值将针对HTML进行转义（即用'<'替换'<'） |
| <% inline code %> | 将给定的代码插入模板结构中，从而允许插入结构性JavaScript。 |
| <%# text %> | 插入一个文本 |

## Simple
一个简单的原理图示例，它使用一个选项来确定其路径来创建“ hello world”文件：

```typescript
import {Tree} from '@angular-devkit/schematics';

export default function MySchematic(options: any) {
  return (tree: Tree) => {
    tree.create(options.path + '/hi', 'Hello world!');
    return tree;
  };
}
```

此示例中的一些内容：

1.该功能从工具接收选项列表。
2.它返回一个 Rule，这是从Tree到另一个Tree 的转化。
 
示意图的简化示例，该示例创建一个包含新类的文件，并使用一个选项来确定其名称：

```typescript
// files/__name@dasherize__.ts

export class <%= classify(name) %> {
}
```

```typescript
// index.ts

import { strings } from '@angular-devkit/core';
import {
  Rule, SchematicContext, SchematicsException, Tree,
  apply, branchAndMerge, mergeWith, template, url,
} from '@angular-devkit/schematics';
import { Schema as ClassOptions } from './schema';

export default function (options: ClassOptions): Rule {
  return (tree: Tree, context: SchematicContext) => {
    if (!options.name) {
      throw new SchematicsException('Option (name) is required.');
    }

    const templateSource = apply(
      url('./files'),
      [
        template({
          ...strings,
          ...options,
        }),
      ]
    );

    return branchAndMerge(mergeWith(templateSource));
  };
}
```
