ES官方提供了各种不同语言的客户端，用来操作ES。这些客户端的本质就是组装DSL语句，通过http请求发送给ES。官方文档地址：[https://www.elastic.co/guide/en/elasticsearch/client/index.html](https://www.elastic.co/guide/en/elasticsearch/client/index.html)

其中的Java Rest Client又包括两种：

- Java Low Level Rest Client
- Java High Level Rest Client


## 配置Java客户端

这里使用es的版本是8.15.5

引入依赖：

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

依赖的版本要与es的版本保持一致。

SpringBoot直接引入：

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>  
</dependency>
```


创建连接对象：
```java
@Bean  
public RestClient restClient() {  
    return RestClient.builder(HttpHost.create("https://127.0.0.1:9200"))  
            .setHttpClientConfigCallback(httpClientBuilder ->  
                    httpClientBuilder.setDefaultCredentialsProvider(  
                            new BasicCredentialsProvider() {{  
                                setCredentials(AuthScope.ANY,  
                                        new UsernamePasswordCredentials("elastic", "es1019"));  
                            }}  
                    )  
            ).build();  
}
```

> 定义了连接地址与账号密码，也可以使用API key进行连接，低版本的es不需要认证。


RestClient对象可以直接使用Restful风格进行调用，而ElasticsearchClient对象可以直接调取API操作es。

```java
@Bean  
public ElasticsearchClient elasticsearchClient() {  
    ElasticsearchTransport transport = new RestClientTransport(restClient(), new JacksonJsonpMapper());  
    return new ElasticsearchClient(transport);  
}
```

如果不是SpringBoot项目的话，还需要引入Jackson的依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.0</version>
</dependency>
```

### 创建索引

```java
public void createIndex() throws IOException {
    CreateIndexRequest request = new CreateIndexRequest.Builder()
            .index("test_index")
            .mappings(new TypeMapping.Builder()
                    .properties("id", p -> p.keyword(k -> k))
                    .properties("name", p -> p.text(t -> t))
                    .properties("category", p -> p.keyword(k -> k))
                    .build())
            .build();

    client.indices().create(request);
}
```

- `index("test_index")`：指定索引名称。
- `mappings(...)`：定义字段类型，例如 `keyword`（适用于精确匹配）和 `text`（适用于全文搜索）。

### 删除索引

```java
public void deleteIndex() throws IOException {
    client.indices().delete(d -> d.index("test_index"));
}
```

- `delete(d -> d.index("test_index"))`：删除指定索引。

### 新增文档

```java
public void addDocument() throws IOException {
    Product product = new Product("1", "测试商品", "电子产品");
    client.index(i -> i.index("test_index").id(product.getId())
	    .document(product)
	);
}
```

- `index("test_index")`：指定索引名称。
- `id(product.getId())`：指定文档 ID。
- `document(product)`：存储 Java 对象。

### 批量新增文档

```java
public void bulkAddDocuments() throws IOException {
    List<BulkOperation> operations = List.of(
        new BulkOperation.Builder()
            .index(i -> i.index("test_index").id("1").document(new Product("1", "商品A", "分类1")))
            .build(),
        new BulkOperation.Builder()
            .index(i -> i.index("test_index").id("2").document(new Product("2", "商品B", "分类2")))
            .build()
    );

    BulkRequest request = new BulkRequest.Builder().operations(operations).build();
    client.bulk(request);
}
```

- `BulkOperation`：用于构造批量操作。
- `bulk(request)`：批量执行新增操作。

### 查询文档

```java
public void getDocument() throws IOException {
    Product product = client.get(g -> g.index("test_index").id("1"), Product.class).source();
    System.out.println(product);
}
```

- `get(g -> g.index("test_index").id("1"), Product.class)`：根据 ID 查询文档。
- `source()`：获取文档数据。

### 更新文档

```java
public void updateDocument() throws IOException {
    Product updatedProduct = new Product("1", "更新后的商品名称", "电子产品");
    client.index(i -> i.index("test_index").id(updatedProduct.getId()).document(updatedProduct));
}
```

- `index(...)` 方法用于插入或更新文档，若 ID 已存在，则会覆盖原有文档。

### 删除文档

```java
public void deleteDocument() throws IOException {
    client.delete(d -> d.index("test_index").id("1"));
}
```

- `delete(d -> d.index("test_index").id("1"))`：根据 ID 删除文档。


### 查询所有（Match All）

```java
public void matchAllQuery() throws IOException {
    SearchResponse<Product> response = client.search(s -> s
            .index("test_index")
            .query(q -> q.matchAll(m -> m)), Product.class);
    response.hits().hits().forEach(hit -> System.out.println(hit.source()));
}
```

- `matchAll(m -> m)`：返回索引中的所有文档。

