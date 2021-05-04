+++
author = "盒木"
title =  "文章配置"
date = "2019-03-05"
description = "所谓程序员的工作，就是复制粘贴"
image = "screenshot.png"
categories = [
    "Hugo"
]

+++

## 配置变量说明官网链接：
[Front Matter Variables](https://gohugo.io/content-management/front-matter/#front-matter-variables)

## 模板

- **YAML**, identified by ‘`---`’.

```
---
author: 
title: 
description: 
date: 2020-00-00
slug: 
image: 
categories:
    - 
tags: 
    - 
---
（四个空格，不要用两个tab）
```

- **TOML**, identified by ‘`+++`’.

```
+++
author = ""
title = ""
description = ""
date = "2020-00-00"
slug = ""
image = ""
categories = [
    ""
]
tags = [
    ""
]
```

- **JSON**, a single JSON object which is surrounded by ‘`{`’ and ‘`}`’, each on their own line.

```
{
    "author": ""
    "title": ""
    "description": ""
    "date": "2020-00-00"
    "slug": ""
    "image": ""
    "categories": [
        ""
    ]
    "tags": [
        ""
    ]    
}
```



## 分类模板与图片示例

