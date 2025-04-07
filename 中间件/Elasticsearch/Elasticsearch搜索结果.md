
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

> 内部分页时，必须先查询 0~1000条，然后截取其中的990 ~ 1000的这10条。

**Elasticsearch 默认最大分页深度为 `from + size <= 10000`，超过需使用：**

- `search_after`（推荐）：分页时需要排序，从上一次的排序值开始，查询下一页数据。
- `scroll`（适合大数据导出）：将排序后的文档id形成快照，保存在内存。

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)


## 高亮（Highlight）

### 基本语法

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

### 4.2 注意事项
- 使用 `match`、`multi_match` 等全文查询才可高亮
- 高亮字段必须和查询字段一致，若不同需设置 `"require_field_match": false`

---

## 5. 聚合（Aggregations）

聚合用于数据的分组统计，类似 SQL 中的 `GROUP BY` 和聚合函数。

### 5.1 桶（Bucket）聚合

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20,
        "order": {
          "_count": "desc"
        }
      }
    }
  }
}
```

### 5.2 度量（Metric）聚合
统计字段最大、最小、平均等：

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20,
        "order": {
          "score_stats.avg": "desc"
        }
      },
      "aggs": {
        "score_stats": {
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```

### 5.3 管道（Pipeline）聚合
可对已有聚合结果再聚合，例如对 avg 值求 max 等。

---

## 6. 建议补充

### 6.1 聚合 + 查询范围联动
聚合前加上查询条件可限制参与聚合的数据：

```json
"query": {
  "range": {
    "price": {
      "lte": 200
    }
  }
}
```

### 6.2 聚合字段类型限制
只支持 `keyword`、数值、布尔、日期字段聚合，不支持 `text` 字段。

---

如需更多示例（如聚合+分页混用、Java DSL 转换）可以继续补充。

