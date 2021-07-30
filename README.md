# springcloud-document

## 文档规范

1. 文档目录结构，采用跟[Spring官方](https://spring.io/projects/spring-boot)文档一致的路径
2. 文件 & 文件夹命名用英文，小写，不要带空格（短横线代替）。

## 构建工具

[mdBook](https://rust-lang.github.io/mdBook/)

## 版权声明

版权文件([license.md](/src/license.md))在[src](/src)目录中。

每个文档都需要在底部声明版权，在文档底部通过 `{{#include [path]}}` 统一“incloud”。

注意，注意，注意`path`是一个相对路径，可以参考下面的几个写法。

```text
{{#include license.md}}
{{#include ../license.md}}
{{#include ../../license.md}}
```
