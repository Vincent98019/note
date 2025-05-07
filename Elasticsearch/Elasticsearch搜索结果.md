
Elasticsearch 查询结果结构由多个关键字段组成：

- `query`：查询条件
- `from` 与 `size`：分页控制
- `sort`：排序条件
- `highlight`：高亮设置
- `aggs`：聚合条件

---

## 排序（Sort）

>在使用排序后就不会再进行算分，根据排序设置的规则排列

### 普通字段排序

排序字段可以为 `keyword`、数值、日期类型等。

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "score": "desc"
    },
    {
      "price": "asc"
    }
  ]
}
```


```java
public void sortQuery() throws IOException {
    SearchResponse<Product> response = client.search(s -> s
        .index("indexName")
        .query(q -> q.matchAll(m -> m)) // match_all 查询
        .sort(so -> so
            .field(f -> f
                .field("_score") // 排序字段为score
                .order(SortOrder.Desc) // 降序
            )
        )
        .sort(so -> so
            .field(f -> f
                .field("price") // 第二排序字段为price
                .order(SortOrder.Asc) // 升序
            )
        ),
        Product.class
    );

    response.hits().hits().forEach(hit -> System.out.println(hit.source()));
}
```

### 地理坐标排序

根据与目标点之间的距离升序或降序排列：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance": {
        "location": "31.034661,121.612282",
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```

```java
public void geoDistanceSortQuery() throws IOException {
    SearchResponse<Product> response = client.search(s -> s
        .index("indexName")
        .query(q -> q.matchAll(m -> m)) // match_all 查询
        .sort(so -> so
            .geoDistance(g -> g
                .field("location") // geo_point 字段名
                .location(l -> l
                    .lat(31.034661)
                    .lon(121.612282)
                )
                .order(SortOrder.Asc) // 升序
                .unit(DistanceUnit.Kilometers) // 单位 km
            )
        ),
        Product.class
    );

    response.hits().hits().forEach(hit -> System.out.println(hit.source()));
}
```

## 分页（Pagination）

默认情况下只返回top10的数据。而如果要查询更多数据就需要修改分页参数。

### 基本分页

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 10
}
```

```java
public void basicPaginationQuery() throws IOException {
    SearchResponse<Product> response = client.search(s -> s
        .index("hotel")
        .query(q -> q.matchAll(m -> m)) // 匹配所有文档
        .from(0) // 从第0条开始
        .size(10), // 查询10条
        Product.class
    );

    response.hits().hits().forEach(hit -> System.out.println(hit.source()));
}
```

### 深度分页限制

要查询990~1000的数据，查询逻辑要这么写：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "from": 990, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```

> - 内部分页时，必须先查询 0~1000条，然后截取其中的990 ~ 1000的这10条。
> - 如果要查询第1000条数据，会将集群中每个机器的前1000条数据查出，再进行排序获取结果。

**Elasticsearch 默认最大分页深度为 `from + size <= 10000`，超过需使用：**

- `search_after`（推荐）：分页时需要排序，从上一次的排序值开始，查询下一页数据。
- `scroll`（适合大数据导出）：将排序后的文档id形成快照，保存在内存。

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)


## 高亮（Highlight）

**基本语法：**

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "name": "测试"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```

> **注意事项：**
> 1. 使用 `match`、`multi_match` 等全文查询才可高亮
> 2. 高亮字段必须和查询字段一致，若不同需设置 `"require_field_match": false`

```java
public void highlightQuery() throws IOException {
    SearchResponse<Product> response = client.search(s -> s
        .index("hotel")
        .query(q -> q.match(m -> m.field("name").query("测试")))
        .highlight(h -> h
            .fields("name", f -> f
                .preTags("<em>")
                .postTags("</em>")
            )
        ),
        Product.class
    );

    response.hits().hits().forEach(hit -> {
        Product product = hit.source();
        System.out.println("Product: " + product);
        if (hit.highlight() != null) {
            System.out.println("Highlight: " + hit.highlight().get("name"));
        }
    });
}
```

## 聚合（Aggregations）

聚合用于数据的分组统计，类似 SQL 中的 `GROUP BY` 和聚合函数。

### 桶（Bucket）聚合

用来对文档做分组：

- **TermAggregation**：按照文档字段值分组，例如按照品牌值分组、按照国家分组
- **Date Histogram**：按照日期阶梯分组，例如一周为一组，或者一月为一组

```json
GET /hotel/_search
{
  // 搜索条件
  "query": { 
    "range": {
      "price": {
        "lte": 200
      }
    }
  }, 
  // 设置size为0，结果中不包含查询结果文档，只包含聚合结果
  "size": 0,
  
  // 定义聚合
  "aggs": {
    // 聚合名，随便起
    "brandAgg": {
      // 聚合的类型，按照品牌值聚合，所以选择terms
      "terms": {
        // 参与聚合的字段
        "field": "brand",
        "order": {
          // 排序
          "doc_count": "asc"
        },
        // 希望获取的聚合结果数量
        "size": 20 
      }
    }
  }
}
```

```java
public void bucketAggregation() throws IOException {
    SearchResponse<Void> response = client.search(s -> s
        .index("hotel")
        .size(0) // 不返回文档，只要聚合结果
        .query(q -> q
            .range(r -> r
                .field("price")
                .lte(JsonData.of(200))
            )
        )
        .aggregations("brandAgg", a -> a
            .terms(t -> t
                .field("brand")
                .order(o -> o
                    .key("doc_count")
                    .order(SortOrder.Asc)
                )
                .size(20)
            )
        ),
        Void.class // 不需要文档内容
    );

    // 打印聚合结果
    Aggregate brandAgg = response.aggregations().get("brandAgg");
    if (brandAgg.isSterms()) {
        brandAgg.sterms().buckets().array().forEach(bucket -> {
            System.out.println("品牌: " + bucket.key().stringValue());
            System.out.println("数量: " + bucket.docCount());
        });
    }
}
```

### 度量（Metric）聚合

> 度量聚合很少单独使用，一般是和桶聚合一并结合使用

统计字段最大、最小、平均等：

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",        // 按照 brand 字段分组（即按品牌分组）
        "size": 20,              // 最多返回前 20 个品牌
        "order": {
          "score_stats.avg": "desc"  // 按每个品牌的平均评分降序排序
        }
      },
      "aggs": {
        "score_stats": {
          "stats": {
            // 对每个品牌分组下的 score 字段做统计（最大、最小、平均、总和、数量）
            "field": "score"
          }
        }
      }
    }
  }
}
```

```java
public void brandScoreStatsAggregation() throws IOException {
    SearchResponse<Void> response = client.search(s -> s
        .index("hotel")
        .size(0) // 不返回文档，只聚合结果
        .aggregations("brandAgg", a -> a
            .terms(t -> t
                .field("brand")
                .size(20)
                .order(o -> o
                    .key("score_stats.avg")
                    .order(SortOrder.Desc)
                )
            )
            .aggregations("score_stats", subAgg -> subAgg
                .stats(st -> st
                    .field("score")
                )
            )
        ), Void.class);

    // 解析结果
    TermsAggregate brandAgg = response.aggregations().get("brandAgg").terms();
    for (TermsBucket bucket : brandAgg.buckets().array()) {
        String brand = bucket.key().stringValue();
        long docCount = bucket.docCount();

        StatsAggregate stats = bucket.aggregations().get("score_stats").stats();
        double avg = stats.avg();
        double min = stats.min();
        double max = stats.max();
        double sum = stats.sum();
        long count = stats.count();

        System.out.printf("品牌: %s, 数量: %d, 平均分: %.2f, 最低: %.2f, 最高: %.2f, 总分: %.2f\n",
                brand, docCount, avg, min, max, sum);
    }
}
```

### 管道（Pipeline）聚合

参考上述代码，可对已有聚合结果再聚合，例如对 avg 值求 max 等。
