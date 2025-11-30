---
title: Elasticsearch 结构化认知各种 Query
date: 2018-03-14
categories:
  - Data
tags:
  - Elasticsearch
  - ELK
excerpt: "系统梳理 Elasticsearch Query DSL 核心模型、两种上下文及复合查询的完整知识脉络，聚焦召回与排序的本质。"
aiSummary: "本文聚焦 Elasticsearch Query DSL 的结构化认知。首先区分两种上下文：Query 上下文计算相关性得分用于全文搜索，Filter 上下文不算分但自动缓存适合精确过滤。其次梳理两大核心查询模型——基于词项的精确匹配（term、terms、range 等）与基于全文的相关性匹配（match、match_phrase、multi_match 等）。最后详解复合查询：Bool 查询用于逻辑组合，dis_max 取最佳字段得分，multi_match 支持多字段搜索并可通过 type 参数选择最佳/最多/跨字段模式，function_score 则可在 BM25 基础上叠加自定义评分因素。"
---

Elasticsearch 作为强大的搜索引擎，提供玲琅满目的搜索方式。

- match, term, bool, multi_match blah blah...

我尝试阅读官方 [reference#quer-languanges](https://www.elastic.co/docs/reference/query-languages) 来梳理，但毕竟是文档型，很难梳理认知结构。

🔍 聚焦 **Query DSL** 核心查询类型，做一次 Query 的结构化认知，构建完整的知识脉络。

👉 Query 的本质只有两步：召回（Recall）+ 排序（Score）
- 召回：哪些文档能被找到
- 排序：谁排在前面 （ _score）

所以在梳理各种查询时，围绕这两点去思考。

<!-- more -->

## 两大上下文：Query 与 Filter

首先，Elasticsearch 中所有查询都运行在两种不同的上下文之下：

| 上下文                | 是否算分               | 是否缓存    | 典型场景                                                             |
| ------------------ | ------------------ | ------- | ---------------------------------------------------------------- |
| **Query Context**  | 是，计算相关性得分 `_score` | 否       | 全文搜索，需要按相关性排序的结果                                                 |
| **Filter Context** | 否，只判断“是否匹配”，得分恒为 0 | 是（自动缓存） | 精确过滤，如 `term`、`range`、`exists`，或 `bool` 中的 `filter` / `must_not` |
|                    |                    |         |                                                                  |

因此，在编写查询时，如果不需要算分（例如：限定 `status = "published"`），应优先使用 Filter 上下文，以获得更好的性能。

✅ 要排序 -> Query, 只过滤 -> Filter


---


## 两大核心查询模型：基于词项 VS 基于全文

**关键：是否经过分词器（Analyzer）处理**
- 基于词项 👉 精确匹配：不分词，直接查倒排索引
- 基于全文 👉 相关性匹配：会分词 + 计算相关性（BM25）



### 基于词项的查询（Term-level Queries）

“词项”（Term）是表达语意的最小单位，也是倒排索引中的最小单位。
- **特点**：词项查询不对输入内容分词，将整个输入作为一个词项在倒排索引中进行**精确匹配**
- **场景**：通常用于结构化数据（如 keyword、数字、状态、分类标签等）或精确匹配场景


**常用的词项查询类型**

| 查询类型       | 说明             | 示例场景                               |
| ---------- | -------------- | ---------------------------------- |
| `term`     | 单个精确值匹配        | 查询 `productID.keyword = "XYZ-123"` |
| `terms`    | 多值精确匹配（只要命中其一） | 查询状态为 `["active", "pending"]` 的订单  |
| `range`    | 范围匹配（数值、日期）    | 查询价格 `100~500` 的商品                 |
| `exists`   | 字段存在性检查        | 找出所有包含 `comment` 字段的文档             |
| `prefix`   | 前缀匹配（不分析）      | 查询以 `elast` 开头的索引名                 |
| `wildcard` | 通配符 `*` 和 `?`  | 查询 `"elas*"`                       |
| `fuzzy`    | 模糊匹配，允许编辑距离    | 用户输入拼写错误时容错                        |
| `regexp`   | 正则表达式匹配        | 复杂模式搜索（慎用）                         |

⚠️ 典型的非预期示例：term 查询
```json
{
  "query": {
    "term": {
      "desc": {
        "value": "Hello"
      }
    }
  }
}
```

例如 desc 字段是 text 类型，文档 1 存储的是 `"Hello World"`（索引词为 ["hello", "world"]）
- 查询结果：这条文档不会被匹配到
- 原因：text 字段会被分词，而 term 查询不会分词（ 'Hello' --🙅🏻‍♂️-> ['hello', 'world'] ）

同理：即使 query 改为 `"Hello World"`，也依然匹配不到。

如果你想精确匹配整个字段值 `"Hello World"`，那就指定 keyword 类型去匹配。
```json
{
  "query": {
    "term": {
      "desc.keyword": {
        "value": "Hello World"
      }
    }
  }
}
```



### 基于全文的查询（Full-Text Queries）

查询字符串会先经过分词器处理，生成词项列表后再进行底层查询，最终合并结果并计算相关性得分。
- **Match Query**：标准全文检索。默认情况下词项之间是 `OR` 关系，即匹配任意一个分词结果即可。
- **Match Phrase Query**：短语搜索。词项之间必须是 `AND` 关系，且**顺序和位置**必须一致。
- **Multi-match Query**：在多个字段上运行相同的 Match 查询。
- **Query String / Simple Query String**：支持复杂的 Lucene 语法（如 `+`、`-`、`AND`、`OR`），前者功能更强但易报错，后者更鲁棒，适合直接暴露给最终用户


**常用全文查询类型对比**

| 查询类型                  | 词项间关系                              | 位置/顺序敏感性        | 典型场景                                     |
| --------------------- | ---------------------------------- | --------------- | ---------------------------------------- |
| `match`               | 默认 **OR**，可改为 **AND**              | 无               | 普通全文搜索，如搜索 `"matrix reloaded"` 匹配任一单词    |
| `match_phrase`        | **AND**                            | 有（允许 `slop` 滑动） | 短语搜索，如搜 `"elasticsearch in action"` 要求连续 |
| `match_phrase_prefix` | **AND**，且最后一个词项做前缀匹配               | 有               | 实时搜索建议（如边输入边匹配）                          |
| `multi_match`         | 取决于 `type`                         | 视类型而定           | 同时在多个字段上搜索                               |
| `query_string`        | 支持 **AND / OR / NOT** 及复杂语法        | 有               | 高级用户输入复杂的组合查询（需谨慎处理语法错误）                 |
| `simple_query_string` | 默认 **OR**，可配置，支持 `+`、`-`、`\|` 简化逻辑 | 有               | 暴露给普通用户的搜索框，容错性强                         |
|                       |                                    |                 |                                          |

#### match VS match_phrase

- **`match`**：底层会被拆解为多个 `term` 查询（默认 OR 关系），只关心 **词项是否出现**，不关心顺序。
- **`match_phrase`**：底层使用短语查询（phrase query），在满足 AND 条件的同时，要求词项的位置符合原始顺序，可以通过 `slop` 参数允许间隔若干词。

e.g. 如果你搜的是“周杰伦”，你不希望搜出“周某和杰伦” --> `match_phrase`（要求词项必须相邻或者通过 `slop` 参数允许一定的跳词）

#### query_string VS simple_query_string

- **`query_string`**：支持 Lucene 查询语法（如 `+`、`-`、`*`、`()`、`AND`/`OR`/`NOT`），非常灵活，但若用户输入非法语法会抛出异常。适用于后台受控的查询构建。
- **`simple_query_string`**：对语法错误更宽容，且只支持部分功能（`+` 表示必须，`-` 表示禁止，`|` 表示 OR，不支持 `AND`/`OR`/`NOT` 关键字（会当成普通词项处理，而不是操作符） ）。推荐用于前端搜索框，减少异常风险

```json
// query_string 支持 AND
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Hello AND World"
    }
  }
}

// simple_query_string 中将 "AND" 视为普通词，除非用 + 或自定义运算符
{
  "query": {
    "simple_query_string": {
      "query": "Hello +World",
      "fields": ["name"],
      "default_operator": "AND"
    }
  }
}
```


---

## Compound Queries（复合查询）

👉 适合你需要 “既要，又要，还要” 的场景，可以进行逻辑构建与算分控制。

复合查询将多个子查询（基础查询或其他复合查询）组合成一个整体，实现复杂逻辑。


### Bool 查询

Bool 查询最常用的复合查询，包含 4 种子句（其中 2 种会影响算分，2 种不会）

| 子句         | 作用                                   | 是否算分    | Filter 缓存 | 典型用法             |
| ---------- | ------------------------------------ | ------- | --------- | ---------------- |
| `must`     | 必须匹配（逻辑 **AND**）                     | ✅ 贡献得分  | 否         | 核心关键词匹配          |
| `should`   | 选择性匹配（至少匹配 `minimum_should_match` 个） | ✅ 贡献得分  | 否         | 提升相关性的加分项        |
| `filter`   | 必须匹配                                 | ❌ 得分为 0 | ✅         | 精确过滤（状态、时间范围、类型） |
| `must_not` | 必须不匹配                                | ❌ 得分为 0 | ✅         | 排除某些值            |


**示例**：搜索标题或内容中包含“Elasticsearch”，且发布日期在 2015 年之后，优先展示点赞数高的文档。
```json
{
  "query": {
    "bool": {
      "must": [
        { "multi_match": {
            "query": "Elasticsearch",
            "fields": ["title", "content"]
        }}
      ],
      "filter": [
        { "range": { "publish_date": { "gte": "2015-01-01" } } }
      ],
      "should": [
        { "term": { "likes_count": { "value": 100 } } }
      ],
      "minimum_should_match": 1
    }
  }
}
```


### Disjunction Max 查询

当我们在多个字段上搜索同一关键词时，可能希望 **以最佳匹配字段的得分为主**，而不是简单叠加各个字段的得分（即 `bool` 的 `should` 行为）。

`dis_max` 正为此而生：它会取所有子查询中得分最高的那个作为最终得分。

```json
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "Apple pencil" } },
        { "match": { "body": "Apple pencil" } }
      ],
      "tie_breaker": 0.3   // 可选：将其他字段的得分乘以 tie_breaker 后累加
    }
  }
}
```

其中 `tie_breaker` 使得得分不再是单一的“最高分”，而是 `best_score + tie_breaker * other_scores`，让其他匹配字段也能微弱影响排序，但依然保持最佳字段的主导地位。


**dis_max VS bool should**
- dis_max: 适合多个字段中，只要有一个字段匹配得好，文档就应该排在前面（例如标题、关键词、摘要）。
- bool + should: 适合多个字段的内容需要互补，命中越多字段越好（例如全文检索，多个字段都是内容的有机部分）。

👉 dis_max 取“最佳匹配字段”的得分，而不是累加所有字段的得分，防止拼凑出来的文档排名靠前。



### Multi‑Match 查询

`multi_match` 是 `match` 查询在多字段上的一个封装，通过 `type` 参数支持三种常见场景：

- **best_fields**: 取多个字段中最佳匹配的得分 👉 哪个字段最匹配，用哪个
- **most_fields**: 将多个字段的得分相加 👉 多个字段匹配越多，分越高）
- **cross_fields**: 将多个字段视为一个大字段进行跨字段匹配 👉 把多个字段当成一个字段

| `type`             | 适用场景                                                                           | 等价底层            |
| ------------------ | ------------------------------------------------------------------------------ | --------------- |
| `best_fields` （默认） | 单个关键词搜索，希望匹配到任意一个字段即可（如标题或正文）                                                  | `dis_max`       |
| `most_fields`      | 字段间相互补充（如一个字段用不同的分词器 index）                                                    | `bool` `should` |
| `cross_fields`     | 人名、地址等自然分散在多个字段中，且需要词语关联（如 `"first_name"` 和 `"last_name"` 一同匹配 `"John Smith"`） | 特殊混合查询          |
```json
{
  "query": {
    "multi_match": {
      "query": "Apple pencil",
      "fields": ["title^2", "content"],   // title 字段权重为 2
      "type": "best_fields",
      "tie_breaker": 0.2,
      "minimum_should_match": "20%"
    }
  }
}
```


### Function Score 查询

当默认的 BM25 算法（词频相关性）不够用时，需要 **在相关性基础上叠加自定义因素**（如热度、用户点击量、更新时间、新鲜度等）来重新干预评分逻辑时，`function_score` 可以将多个函数计算出的分数与原始查询分数相结合。

> _常用函数_：`weight`（权重）、`field_value_factor`（利用字段值计算）、`gauss/linear/exp`（衰减函数，如距离越远分越低）



```json
{
  "query": {
    "function_score": {
      "query": {
        "match": { "content": "Elasticsearch" }
      },
      "field_value_factor": {
        "field": "popularity",
        "factor": 0.5,
        "modifier": "log1p"
      },
      "boost_mode": "multiply",   // 原始得分与函数得分的组合方式
      "max_boost": 10
    }
  }
}
```

除了 `field_value_factor`，还可以使用 `script_score`（任意脚本）、`random_score`（随机排序）、`decay`（衰减函数，用于地理位置、日期等）等。


## Expensive Queries

某些查询（如 `fuzzy` 模糊查询、`wildcard` 通配符查询、`script` 脚本查询）执行非常慢，使用不当甚至可能影响集群稳定性。

🙅🏻‍♂️ Very expensive ！




## 场景选择

从实际场景选择正确的查询

| 业务需求                 | 推荐查询                                                   | 原因                  |
| -------------------- | ------------------------------------------------------ | ------------------- |
| 精确匹配：订单号、用户名、状态码     | `term` 或 `terms` 结合 filter                             | 不需要算分，且要求精确         |
| 搜索框（普通用户）            | `match` 与 `multi_match`（best_fields）                   | 分词召回，相关性排序，简单易理解    |
| 短语搜索（如引用搜索）          | `match_phrase` 或 `match_phrase_prefix`                 | 保持词序和连续要求           |
| 高级搜索（管理员/分析师）        | `query_string`                                         | 支持复杂的 AND/OR/NOT 组合 |
| 搜索结果页 + 筛选（价格、类目、日期） | `bool`：`must` 用于关键词，`filter` 用于筛选                      | 筛选不参与算分，可缓存，性能好     |
| 同义词、多字段加权            | `multi_match` with `best_fields` and `fields` boosting | 简洁高效                |
| 地址、人名跨字段搜索           | `multi_match` 的 `cross_fields` 类型                      | 解决词语跨字段关联问题         |
| 个性化排序：热度 + 文本相关性     | `function_score`                                       | 灵活调整最终得分            |


## Q&A

| **Q**                                | **A**                                                                             |
| ------------------------------------ | --------------------------------------------------------------------------------- |
| `term` 与 `match` 最核心区别是什么？           | `term` 不分词（精确匹配），`match` 分词（全文搜索）。                                                |
| `match` 与 `match_phrase` 的得分差异？      | `match` 只需部分词命中，得分累加；`match_phrase` 必须所有词按序命中，且顺序错误则得分为 0。                        |
| 如何在 `bool` 中实现 **必须 AND 但不算分**？      | 使用 `filter` 子句，如 `{"bool": {"filter": [...]}}`。                                   |
| 什么时候用 `dis_max` 而不是 `bool` `should`？ | 当你希望**最佳字段**决定排序，而不是多个字段得分相加时。                                                    |
| `query_string` 何时会抛异常？               | 语法错误，如未闭合的括号、单独的 `+` 等。`simple_query_string` 则自动忽略非法部分。                           |
| 如何提升某个字段的权重？                         | 在 `multi_match` 的 `fields` 中用 `^` 符号：`"title^3"`，或使用 `bool` + `should` + `boost`。 |
