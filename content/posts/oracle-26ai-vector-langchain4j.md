+++
title = 'Oracle 26ai 文本与图片向量存储与查询'
date = 2025-11-17T15:24:00+08:00
categories = ['Java', 'Oracle', 'AI']
tags = ['oracle', 'langchain4j', 'vector-database', 'embedding', 'dashscope']
summary = 'Oracle 26ai、 LangChain4j 框架结合阿里魔搭平台的向量化模型，实现文本和图片的向量存储与相似性查询。'
+++

Oracle Database 26ai 版本引入了强大的向量存储和查询功能，为 AI 应用提供了原生的向量数据库支持。本文将介绍如何使用 LangChain4j 框架结合阿里魔搭平台的向量化模型，实现文本和图片的向量存储与相似性查询。

## 技术栈

- **Oracle Database 26ai**: 提供原生向量存储能力
- **LangChain4j**: Java 生态的 LLM 应用框架
- **阿里魔搭平台**: 提供向量化模型服务（DashScope API）
- **Hutool**: Java 工具库，用于 HTTP 请求和 JSON 处理

## 环境准备

### Maven 依赖

首先在 `pom.xml` 中添加必要的依赖：

```xml
<dependencies>
    <!-- LangChain4j 核心 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-core</artifactId>
        <version>1.8.0</version>
    </dependency>
    
    <!-- LangChain4j Oracle 集成 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-oracle</artifactId>
        <version>1.8.0-beta15</version>
    </dependency>
    
    <!-- LangChain4j OpenAI 集成（用于文本向量化） -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai</artifactId>
        <version>1.8.0</version>
    </dependency>
    
    <!-- Oracle JDBC 驱动 -->
    <dependency>
        <groupId>com.oracle.database.jdbc</groupId>
        <artifactId>ojdbc11</artifactId>
        <version>23.4.0.24.05</version>
    </dependency>
    
    <!-- Oracle UCP 连接池 -->
    <dependency>
        <groupId>com.oracle.database.jdbc</groupId>
        <artifactId>ucp</artifactId>
        <version>23.4.0.24.05</version>
    </dependency>
    
    <!-- Hutool 工具库 -->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.25</version>
    </dependency>
</dependencies>
```

### 数据库连接配置

使用 Oracle UCP 连接池配置数据库连接：

```java
PoolDataSource dataSource = PoolDataSourceFactory.getPoolDataSource();
dataSource.setConnectionFactoryClassName(
        "oracle.jdbc.datasource.impl.OracleDataSource");
dataSource.setURL("jdbc:oracle:thin:@//host:1521/FREEPDB1");
dataSource.setUser("your_username");
dataSource.setPassword("your_password");
```

## 一、文本向量存储与查询

### 1.1 创建 EmbeddingStore

使用 LangChain4j 的 `OracleEmbeddingStore` 创建向量存储：

```java
EmbeddingStore<TextSegment> embeddingStore = OracleEmbeddingStore.builder()
        .dataSource(dataSource)
        .embeddingTable("test_content_retriever", 
                CreateOption.CREATE_IF_NOT_EXISTS)
        .build();
```

`CreateOption.CREATE_IF_NOT_EXISTS` 参数表示如果表不存在则自动创建。Oracle 26ai 会自动创建包含向量列的表结构。

### 1.2 配置向量化模型

本示例使用兼容 OpenAI 接口的向量化服务（也可以使用本地部署的 Ollama）：

```java
EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
        .baseUrl("http://your-embedding-service:8100/v1/")
        .apiKey("demo")
        .modelName("nomic-embed-text:latest")
        .build();
```

### 1.3 存储文本向量

将文本转换为向量并存储到 Oracle 数据库：

```java
List<String> textList = List.of(
    "我叫张三", 
    "我今年18岁", 
    "我来自山东临沂", 
    "我喜欢打篮球", 
    "我最喜欢吃的水果是西瓜"
);

textList.forEach(text -> {
    TextSegment segment = TextSegment.from(text);
    Embedding embedding = embeddingModel.embed(segment).content();
    embeddingStore.add(embedding, segment);
});
```

### 1.4 相似性查询

使用 `ContentRetriever` 进行语义相似性查询：

```java
ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
        .embeddingStore(embeddingStore)
        .embeddingModel(embeddingModel)
        .maxResults(2)        // 最多返回2条结果
        .minScore(0.5)        // 最低相似度阈值
        .build();

// 查询示例
List<Content> results = retriever.retrieve(Query.from("你今年多大？"));
results.forEach(content -> {
    System.out.println(content.textSegment().text() + " - " + content.metadata());
});
```

查询 "你今年多大？" 将返回语义最相近的文本，即 "我今年18岁"。

## 二、图片向量存储与查询

图片向量化需要使用支持多模态的向量化模型。本文使用阿里魔搭平台的 `tongyi-embedding-vision-plus` 模型。

### 2.1 自定义 QwenEmbeddingModel

由于 LangChain4j 官方暂未提供对阿里魔搭多模态向量化的直接支持，我们需要自定义一个 `EmbeddingModel`：

```java
package com.wzw;

import cn.hutool.http.HttpRequest;
import cn.hutool.http.HttpResponse;
import cn.hutool.json.JSONArray;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import dev.langchain4j.data.embedding.Embedding;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.embedding.DimensionAwareEmbeddingModel;
import dev.langchain4j.model.output.Response;

import java.util.List;

public class QwenEmbeddingModel extends DimensionAwareEmbeddingModel {

    private final String apiKey;
    private final String modelName;

    public QwenEmbeddingModel(QwenEmbeddingModelBuilder builder) {
        this.apiKey = builder.apiKey;
        this.modelName = builder.modelName;
    }

    @Override
    public Response<Embedding> embed(String text) {
        List<Float> floats = embeddingByQwen(text);
        if (floats == null) {
            throw new RuntimeException("Embedding failed");
        }
        return Response.from(Embedding.from(floats));
    }

    @Override
    public Response<Embedding> embed(TextSegment textSegment) {
        List<Float> floats = embeddingByQwen(textSegment.text());
        if (floats == null) {
            throw new RuntimeException("Embedding failed");
        }
        return Response.from(Embedding.from(floats));
    }

    @Override
    public Response<List<Embedding>> embedAll(List<TextSegment> list) {
        List<Embedding> embeddings = list.stream().map(textSegment -> {
            List<Float> floats = embeddingByQwen(textSegment.text());
            if (floats == null) {
                throw new RuntimeException("Embedding failed");
            }
            return Embedding.from(floats);
        }).toList();
        return Response.from(embeddings);
    }

    public static QwenEmbeddingModelBuilder builder() {
        return new QwenEmbeddingModelBuilder();
    }

    public static class QwenEmbeddingModelBuilder {
        private String apiKey;
        private String modelName;

        public QwenEmbeddingModelBuilder apiKey(String apiKey) {
            this.apiKey = apiKey;
            return this;
        }

        public QwenEmbeddingModelBuilder modelName(String modelName) {
            this.modelName = modelName;
            return this;
        }

        public QwenEmbeddingModel build() {
            return new QwenEmbeddingModel(this);
        }
    }

    private List<Float> embeddingByQwen(String imageUrl) {
        String url = "https://dashscope.aliyuncs.com/api/v1/services/" + 
                "embeddings/multimodal-embedding/multimodal-embedding";

        // 构建请求体
        JSONArray contentsArray = new JSONArray();
        contentsArray.add(new JSONObject().set("image", imageUrl));
        JSONObject input = new JSONObject().set("contents", contentsArray);
        JSONObject requestBody = new JSONObject()
                .set("model", this.modelName)
                .set("input", input);

        try {
            HttpResponse httpResponse = HttpRequest.post(url)
                    .header("Authorization", "Bearer " + this.apiKey)
                    .header("Content-Type", "application/json")
                    .body(requestBody.toString())
                    .execute();

            String result = httpResponse.body();
            JSONObject jsonObject = JSONUtil.parseObj(result);
            JSONArray embeddings = jsonObject.getJSONObject("output")
                    .getJSONArray("embeddings")
                    .getJSONObject(0)
                    .getJSONArray("embedding");
            
            httpResponse.close();
            return embeddings.toList(Float.class);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

### 2.2 API 请求格式说明

阿里魔搭的多模态向量化 API 请求格式如下：

**请求示例：**

```json
{
    "model": "tongyi-embedding-vision-plus",
    "input": {
        "contents": [
            {"image": "https://example.com/image.png"}
        ]
    }
}
```

**响应示例：**

```json
{
    "output": {
        "embeddings": [
            {
                "embedding": [0.0712890625, -0.04071044921875, ...],
                "index": 0,
                "type": "image"
            }
        ]
    },
    "usage": {
        "input_tokens": 66,
        "output_tokens": 1,
        "total_tokens": 67
    },
    "request_id": "xxx"
}
```

### 2.3 图片向量存储

使用自定义的 `QwenEmbeddingModel` 存储图片向量：

```java
EmbeddingModel embeddingModel = QwenEmbeddingModel.builder()
        .apiKey("your-dashscope-api-key")
        .modelName("tongyi-embedding-vision-plus")
        .build();

EmbeddingStore<TextSegment> embeddingStore = OracleEmbeddingStore.builder()
        .dataSource(dataSource)
        .embeddingTable("test_image_retriever", 
                CreateOption.CREATE_IF_NOT_EXISTS)
        .build();

// 存储图片向量
List<String> imageUrls = Arrays.asList(
    "https://img.meituan.net/csc/880064b6b368ddcd97d4a72e5a9880881368981.png",
    "https://m.360buyimg.com/i/jfs/t1/358400/9/12129/52487/691c192aFb513334c/e92c9f73541be44c.png",
    "https://m.360buyimg.com/i/jfs/t1/367859/36/1853/54479/691c1929F33481b1e/04734cc422c03c18.png",
    "https://img.meituan.net/csc/2fb05a555a07ec85dff6d92d643d495d1114989.png"
);

imageUrls.forEach(url -> {
    TextSegment segment = TextSegment.from(url);  // 将图片URL作为文本内容存储
    Embedding embedding = embeddingModel.embed(segment).content();
    embeddingStore.add(embedding, segment);
    System.out.println("Stored: " + url);
});
```

> **注意**：这里我们将图片 URL 作为 `TextSegment` 的内容存储，这样在查询结果中可以直接获取到原始图片的 URL。

### 2.4 图片相似性查询

```java
ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
        .embeddingStore(embeddingStore)
        .embeddingModel(embeddingModel)
        .maxResults(2)
        .minScore(0.5)
        .build();

// 使用一张苹果图片查询相似图片
String testImageUrl = "https://m.360buyimg.com/i/jfs/t1/364672/27/1937/73599/" + 
        "691c2d9eF7f235643/54613de88f5c4138.png";
List<Content> results = retriever.retrieve(Query.from(testImageUrl));

results.forEach(content -> {
    System.out.println("相似图片: " + content.textSegment().text());
    System.out.println("元数据: " + content.metadata());
});
```

## 三、Oracle 26ai 向量功能特性

### 3.1 自动表结构管理

使用 `OracleEmbeddingStore` 时，Oracle 会自动创建包含向量列的表结构：

```sql
CREATE TABLE test_content_retriever (
    id RAW(16) DEFAULT SYS_GUID() PRIMARY KEY,
    embedding VECTOR,
    content CLOB,
    metadata JSON
);
```

### 3.2 向量索引

Oracle 26ai 支持多种向量索引类型，可以根据数据规模和查询需求选择：

- **HNSW（Hierarchical Navigable Small World）**: 适合高维向量的近似最近邻搜索
- **IVF（Inverted File Index）**: 适合大规模数据集

### 3.3 相似度计算

Oracle 26ai 支持多种向量距离度量方式：

- **余弦相似度（Cosine Similarity）**: 测量向量方向的相似性
- **欧氏距离（Euclidean Distance）**: 测量向量间的直线距离
- **点积（Dot Product）**: 测量向量的投影长度

## 四、完整示例代码

### 4.1 文本向量检索完整示例

```java
@Test
public void textEmbeddingTest() throws SQLException {
    // 1. 配置数据源
    PoolDataSource dataSource = PoolDataSourceFactory.getPoolDataSource();
    dataSource.setConnectionFactoryClassName(
            "oracle.jdbc.datasource.impl.OracleDataSource");
    dataSource.setURL("jdbc:oracle:thin:@//host:1521/FREEPDB1");
    dataSource.setUser("your_username");
    dataSource.setPassword("your_password");

    // 2. 配置向量化模型
    EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
            .baseUrl("http://your-embedding-service:8100/v1/")
            .apiKey("demo")
            .modelName("nomic-embed-text:latest")
            .build();

    // 3. 创建向量存储
    EmbeddingStore<TextSegment> embeddingStore = OracleEmbeddingStore.builder()
            .dataSource(dataSource)
            .embeddingTable("test_content_retriever", 
                    CreateOption.CREATE_IF_NOT_EXISTS)
            .build();

    // 4. 创建内容检索器
    ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
            .embeddingStore(embeddingStore)
            .embeddingModel(embeddingModel)
            .maxResults(2)
            .minScore(0.5)
            .build();

    // 5. 执行查询
    List<Content> results = retriever.retrieve(Query.from("你今年多大？"));
    results.forEach(content -> {
        System.out.println(content.textSegment().text() + 
                " - " + content.metadata());
    });
}
```

### 4.2 图片向量检索完整示例

```java
@Test
public void imageEmbeddingTest() throws SQLException {
    // 1. 配置数据源
    PoolDataSource dataSource = PoolDataSourceFactory.getPoolDataSource();
    dataSource.setConnectionFactoryClassName(
            "oracle.jdbc.datasource.impl.OracleDataSource");
    dataSource.setURL("jdbc:oracle:thin:@//host:1521/FREEPDB1");
    dataSource.setUser("your_username");
    dataSource.setPassword("your_password");

    // 2. 配置多模态向量化模型
    EmbeddingModel embeddingModel = QwenEmbeddingModel.builder()
            .apiKey("your-dashscope-api-key")
            .modelName("tongyi-embedding-vision-plus")
            .build();

    // 3. 创建向量存储
    EmbeddingStore<TextSegment> embeddingStore = OracleEmbeddingStore.builder()
            .dataSource(dataSource)
            .embeddingTable("test_image_retriever", 
                    CreateOption.CREATE_IF_NOT_EXISTS)
            .build();

    // 4. 创建内容检索器
    ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
            .embeddingStore(embeddingStore)
            .embeddingModel(embeddingModel)
            .maxResults(2)
            .minScore(0.5)
            .build();

    // 5. 使用测试图片查询
    String testUrl = "https://img.meituan.net/csc/" + 
            "57148221577fe872de8c11ad400631811993996.png";
    List<Content> results = retriever.retrieve(Query.from(testUrl));
    results.forEach(content -> {
        System.out.println(content.textSegment().text() + 
                " - " + content.metadata());
    });
}
```

## 五、注意事项

1. **API Key 安全**：生产环境中请勿将 API Key 硬编码在代码中，建议使用环境变量或配置中心管理。

2. **向量维度一致性**：同一个向量表中的所有向量维度必须一致。不同的向量化模型可能产生不同维度的向量，需要分开存储。

3. **网络延迟**：调用外部向量化服务时需要考虑网络延迟，建议在生产环境中实现重试机制和超时处理。

4. **批量处理**：对于大量数据的向量化，建议实现批量处理逻辑，避免频繁的 API 调用。

5. **索引优化**：当数据量较大时，建议在向量列上创建合适的索引以提升查询性能。

## 总结

Oracle Database 26ai 的向量存储功能为企业级 AI 应用提供了强大的基础设施支持。结合 LangChain4j 框架，我们可以轻松实现文本和图片的向量化存储与语义检索。阿里魔搭平台的多模态向量化模型更是为图片搜索等场景提供了便捷的解决方案。

**相关链接：**

- [Oracle Database 26ai 官方文档](https://docs.oracle.com/en/database/)
- [LangChain4j GitHub](https://github.com/langchain4j/langchain4j)
- [阿里魔搭平台](https://dashscope.aliyun.com/)