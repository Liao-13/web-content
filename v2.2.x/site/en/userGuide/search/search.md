---
id: search.md
related_key: search
summary: Conduct a vector similarity search with Milvus.
---

# Conduct a Vector Similarity Search

This topic describes how to search entities with Milvus.

A vector similarity search in Milvus calculates the distance between query vector(s) and vectors in the collection with specified similarity metrics, and returns the most similar results. By specifying a [boolean expression](boolean.md) that filters the scalar field or the primary key field, you can perform a [hybrid search](hybridsearch.md) or even a search with [Time Travel](timetravel.md).

The following example shows how to perform a vector similarity search on a 2000-row dataset of book ID (primary key), word count (scalar field), and book introduction (vector field), simulating the situation that you search for certain books based on their vectorized introductions. Milvus will return the most similar results according to the query vector and search parameters you have defined. 


## Load collection

All search and query operations within Milvus are executed in memory. Load the collection to memory before conducting a vector similarity search.

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
  <a href="#csharp">C#</a>
</div>

```python
from pymilvus import Collection
collection = Collection("book")      # Get an existing collection.
collection.load()
```

```javascript
await milvusClient.loadCollection({
  collection_name: "book",
});
```

```go
err := milvusClient.LoadCollection(
  context.Background(),   // ctx
  "book",                 // CollectionName
  false                   // async
)
if err != nil {
  log.Fatal("failed to load collection:", err.Error())
}
```

```java
milvusClient.loadCollection(
  LoadCollectionParam.newBuilder()
          .withCollectionName("book")
          .build()
);
```

```csharp
var collection = milvusClient.GetCollection("book").LoadAsync();
```

## Prepare search parameters

Prepare the parameters that suit your search scenario. The following example defines that the search will calculate the distance with Euclidean distance, and retrieve vectors from ten closest clusters built by the IVF_FLAT index.

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
  <a href="#csharp">C#</a>
  <a href="#curl">Curl</a>
</div>

```python
search_params = {
    "metric_type": "L2", 
    "offset": 5, 
    "ignore_growing": False, 
    "params": {"nprobe": 10}
}
```

```javascript
const searchParams = {
    params: { nprobe: 1024 }
};
```

```go
sp, _ := entity.NewIndexIvfFlatSearchParam( // NewIndex*SearchParam func
    10,                                  // searchParam
)

opt := client.SearchQueryOptionFunc(func(option *client.SearchQueryOption) {
    option.Limit = 3
    option.Offset = 0
    option.ConsistencyLevel = entity.ClStrong
    option.IgnoreGrowing = false
})
```

```java
final Integer SEARCH_K = 2;                       // TopK
final String SEARCH_PARAM = "{\"nprobe\":10, \"offset\":5}";    // Params
```

```csharp
var parameters = new SearchParameters
{
    OutputFields = { "title" },
    ConsistencyLevel = ConsistencyLevel.Strong,
    Offset = 5,
    ExtraParameters = { ["nprobe"] = "1024" }
};
```

```curl
# Search entities based on a given vector.
curl --request POST \
     --url '${MILVUS_HOST}:${MILVUS_PORT}/v1/vector/search' \
     --header 'Authorization: Bearer <TOKEN>' \
     --header 'accept: application/json' \
     --header 'content-type: application/json'
     -d '{
        "collectionName": "collection1",
        "vector": [0.0128121, 0.029119, .... , 0.09121]
      }'

# Search entities and return specific fields.
curl --request POST \
     --url '${MILVUS_HOST}:${MILVUS_PORT}/v1/vector/search' \
     --header 'Authorization: Bearer <TOKEN>' \
     --header 'accept: application/json' \
     --header 'content-type: application/json'
     -d '{
       "collectionName": "collection1",
       "outputFields": ["id", "name", "feature", "distance"],
       "vector": [0.0128121, 0.029119, .... , 0.09121],
       "filter": "id in (1, 2, 3)",
       "limit": 100,
       "offset": 0
     }'
```

<div class="language-curl">

Output:

```json
{
    "code": 200,
    "data": {}
}
```

</div>

<table class="language-python">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>metric_type</code></td>
        <td>Method used to measure the distance between vectors during search. It should be the same as the one specified for the index-building process. See <a href="metric.md">Simlarity Metrics</a> for more information.</td>
    </tr>
    <tr>
        <td><code>offset</code></td>
        <td>Number of entities to skip during the search. The sum of this parameter and <code>limit</code> of the <code>search</code> method should be less than <code>16384</code>.</td>
    </tr>
    <tr>
        <td><code>ignore_growing</code></td>
        <td>Whether to ignore growing segments during similarity searches. The value defaults to <code>False</code>, indicating that searches involve growing segments.</td>
    </tr>    
    <tr>
        <td><code>params</code></td>
        <td>Search parameter(s) specific to the specified index type. See <a href="index.md">Vector Index</a> for more information. Possible options are as follows: <ul><li><code>nprobe</code> Indicates the number of cluster units to search. This parameter is available only when <code>index_type</code> is set to <code>IVF_FLAT</code>, <code>IVF_SQ8</code>, or <code>IVF_PQ</code>. The value should be less than <code>nlist</code> specified for the index-building process.</li>
            <li><code>ef</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>HNSW</code>. The value should be within the range from <code>top_k</code> to <code>32768</code>.</li>
            <li><code>search_k</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>ANNOY</code>. The value should be greater than or equal to the top K. </li>
        </ul></td>
    </tr>
    </tbody>
</table>

<table class="language-javascript">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>params</code></td>
        <td>Search parameter(s) specific to the index. See <a href="index.md">Vector Index</a> for more information. Possible options are as follows:<ul>
            <li><code>nprobe</code> Indicates the number of cluster units to search. This parameter is available only when <code>index_type</code> is set to <code>IVF_FLAT</code>, <code>IVF_SQ8</code>, or <code>IVF_PQ</code>. The value should be less than <code>nlist</code> specified for the index-building process.</li>
            <li><code>ef</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>HNSW</code>. The value should be within the range from <code>top_k</code> to <code>32768</code>.</li>
            <li><code>search_k</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>ANNOY</code>. The value should be greater than or equal to the top K. </li>
        </ul></td>
    </tr>
    </tbody>
</table>

<table class="language-go">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    <th>Options</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>NewIndex*SearchParam func</code></td>
        <td>Function to create <code>entity.SearchParam</code> according to different index types.</td>
        <td>For floating point vectors:
            <ul>
                <li><code>NewIndexFlatSearchParam()</code> (FLAT)</li>
                <li><code>NewIndexIvfFlatSearchParam(nprobe int)</code> (IVF_FLAT)</li>
                <li><code>NewIndexIvfSQ8SearchParam(nprobe int)</code> (IVF_SQ8)</li>
                <li><code>NewIndexIvfPQSearchParam(nprobe int)</code> (RNSG)</li>
                <li><code>NewIndexHNSWSearchParam(ef int)</code> (HNSW)</li>
                <li><code>NewIndexANNOYSearchParam(search_k int)</code> (ANNOY)</li>
            </ul>
            For binary vectors:
            <ul>
                <li><code>NewIndexBinFlatSearchParam(nprobe int)</code> (BIN_FLAT)</li>
                <li><code>NewIndexBinIvfFlatSearchParam(nprobe int)</code> (BIN_IVF_FLAT)</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><code>sp</code></td>
        <td>Search parameter(s) specific to the index returned by the preceding functions.</td>
        <td>See <a href="index.md">Vector Index</a> for more information. </td>
    </tr>
    <tr>
        <td><code>opt<code></td>
        <td>Options for ANN searches.</td>
        <td><ul>
            <li><code>Limit</code> Indicates the number of entities to return.</li>
            <li><code>Offset</code> Indicates the number of entities to skip during the search. The sum of this parameter and <code>Limit</code> should be less than <code>16384</code>.</li>
            <li><code>ConsistencyLevel</code> Indicates the consistency level applied during the search.</li>
            <li><code>Ignore Growing</code> Indicates whether to ignore growing segments during similarity searches. The value defaults to <code>False</code>, indicating that searches involve growing segments. </li>
        </ul></td>
    </tr>
    </tbody>
</table>

<table class="language-java">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Options</th>
    </tr>
    </thead>
    <tbody>
  <tr>
        <td><code>SEARCH_K</code></td>
        <td>Number of the most similar results to return.</td>
    <td>N/A</td>
    </tr>
  <tr>
        <td><code>SEARCH_PARAM</code></td>
        <td>Search parameter(s) specific to the index.</td>
    <td>See <a href="index.md">Vector Index</a> for more information. Possible options are as follows:<ul>
        <li><code>nprobe</code> Indicates the number of cluster units to search. This parameter is available only when <code>index_type</code> is set to <code>IVF_FLAT</code>, <code>IVF_SQ8</code>, or <code>IVF_PQ</code>. The value should be less than <code>nlist</code> specified for the index-building process.</li>
        <li><code>ef</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>HNSW</code>. The value should be within the range from <code>top_k</code> to <code>32768</code>.</li>
        <li><code>search_k</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>ANNOY</code>. The value should be greater than or equal to the top K.</li>
        <li><code>metric_type</code> Indicates the metric type used in the search. It should be the same as the one specified when you index the collection.</li>
        <li><code>offset</code> Indicates the number of entities to skip during the search. The sum of this parameter and <code>topK</code> of the <code>withTopK()</code> method should be less than <code>16384</code>.</li>
    </ul></td>
    </tr>
    </tbody>
</table>


<table class="language-csharp">
    <thead>
        <tr>
            <th>Parameter</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>OutputFields</td>
            <td>A dictionary of the fields in the search results.</td>
        </tr>
        <tr>
            <td>ConsistencyLevel</td>
            <td>Consistency level for the search. Possible values are: <ul>
                <li>ConsistencyLevel.Strong</li>
                <li>ConsistencyLevel.Session</li>
                <li>ConsistencyLevel.BoundedStaleness</li>
                <li>ConsistencyLevel.Eventually</li>
                <li>ConsistencyLevel.Customized</li>
            </ul></td>
        </tr>
        <tr>
            <td>Offset</td>
            <td>Number of records to skip before return. The sum of this value and <code>limit</code> in the search request should be less than <code>16384</code>.</td>
        </tr>
        <tr>
            <td>Expression</td>
            <td>Boolean expression used to filter attribute. See <a href="boolean.md">Boolean Expression Rules</a> for more information.</td>
        </tr>
        <tr>
            <td>ExtraParameters</td>
            <td>Other applicable parameters. See <a href="index.md">Vector Index</a> for more information. Possible options are as follows:<ul>
        <li><code>nprobe</code> Indicates the number of cluster units to search. This parameter is available only when <code>index_type</code> is set to <code>IVF_FLAT</code>, <code>IVF_SQ8</code>, or <code>IVF_PQ</code>. The value should be less than <code>nlist</code> specified for the index-building process.</li>
        <li><code>ef</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>HNSW</code>. The value should be within the range from <code>top_k</code> to <code>32768</code>.</li>
        <li><code>search_k</code> Indicates the search scope. This parameter is available only when <code>index_type</code> is set to <code>ANNOY</code>. The value should be greater than or equal to the top K.</li>
    </ul></td>
        </tr>
    </tbody>
</table>

<table class="language-curl">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>collectionName</code></td>
        <td>(Required) The name of the collection to which this operation applies.</td>
    </tr>
    <tr>
        <td><code>filter</code></td>
        <td>The filter used to find matches for the search</td>
    </tr>
    <tr>
        <td><code>limit</code></td>
        <td>The maximum number of entities to return.<br>The sum of this value of that of `offset` should be less than **1024**.<br>The value defaults to <code>100</code>.<br>The value ranges from <code>1</code> to <code>100</code></td>
    </tr>
    <tr>
        <td><code>offset</code></td>
        <td>The number of entities to skip in the search results.<br>The sum of this value and that of `limit` should not be greater than <code>1024</code>.<br>The maximum value is <code>1024</code>.</td>
    </tr>
    <tr>
        <td><code>outputFields</code></td>
        <td>An array of fields to return along with the search results.</td>
    </tr>
    <tr>
        <td><code>vector</code></td>
        <td>The query vector in the form of a list of floating numbers.</td>
    </tr>
    </tbody>
</table>

## Conduct a vector search

Search vectors with Milvus. To search in a specific [partition](glossary.md#Partition), specify the list of partition names. 

Milvus supports setting consistency level specifically for a search. The example in this topic sets the consistency level as `Strong`. You can also set the consistency level as `Bounded`, `Session` or `Eventually`. See [Consistency](consistency.md) for more information about the four consistency levels in Milvus.

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
  <a href="#csharp">C#</a>
  <a href="#curl">Curl</a>
</div>

```python
results = collection.search(
    data=[[0.1, 0.2]], 
    anns_field="book_intro", 
    # the sum of `offset` in `param` and `limit` 
    # should be less than 16384.
    param=search_params,
    limit=10,
    expr=None,
    # set the names of the fields you want to 
    # retrieve from the search result.
    output_fields=['title'],
    consistency_level="Strong"
)

# get the IDs of all returned hits
results[0].ids

# get the distances to the query vector from all returned hits
results[0].distances

# get the value of an output field specified in the search request.
hit = results[0][0]
hit.entity.get('title')
```

```javascript
const results = await milvusClient.search({
    collection_name: "book",
    vector: [0.1, 0.2],
    filter: null,
    // the sum of `limit` and `offset` should be less than 16384.
    limit: 10,
    offset: 2,
    metric_type: MetricType.L2,
    param: searchParams,
    consistency_level: ConsistencyLevelEnum.Strong,
});
```

```go
searchResult, err := milvusClient.Search(
    context.Background(),                    // ctx
    "book",                                  // CollectionName
    []string{},                              // partitionNames
    "",                                      // expr
    []string{"book_id"},                     // outputFields
    []entity.Vector{entity.FloatVector([]float32{0.1, 0.2})}, // vectors
    "book_intro",                            // vectorField
    entity.L2,                               // metricType
    10,                                      // topK
    sp,                                      // sp
    opt,
)
if err != nil {
    log.Fatal("fail to search collection:", err.Error())
}
```

```java
List<String> search_output_fields = Arrays.asList("book_id");
List<List<Float>> search_vectors = Arrays.asList(Arrays.asList(0.1f, 0.2f));

SearchParam searchParam = SearchParam.newBuilder()
        .withCollectionName("book")
        .withConsistencyLevel(ConsistencyLevelEnum.STRONG)
        .withMetricType(MetricType.L2)
        .withOutFields(search_output_fields)
        .withTopK(SEARCH_K)
        .withVectors(search_vectors)
        .withVectorFieldName("book_intro")
        .withParams(SEARCH_PARAM)
        .build();
R<SearchResults> respSearch = milvusClient.search(searchParam);
```

```csharp
var results = await milvusClient.GetCollection("book").SearchAsync(
    vectorFieldName: "book_intro",
    vectors: new ReadOnlyMemory<float>[] { new[] { 0.1f, 0.2f } },
    SimilarityMetricType.L2,
    // the sum of `offset` in `parameters` and `limit` should be less than 16384.
    limit: 10,
    parameters);
```

```curl
# Follow the previous step.
```

<table class="language-python">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>data</code></td>
        <td>Vectors to search with.</td>
    </tr>
    <tr>
        <td><code>anns_field</code></td>
        <td>Name of the field to search on.</td>
    </tr>
    <tr>
        <td><code>param</code></td>
        <td>Search parameter(s) specific to the index. See <a href="index.md">Vector Index</a> for more information.</td>
    </tr>
    <tr>
        <td><code>limit</code></td>
        <td>Number of the most similar results to return.  The sum of this value and <code>offset</code> in <code>param</code> should be less than 16384.</td>
    </tr>
  <tr>
        <td><code>expr</code></td>
        <td>Boolean expression used to filter attribute. See <a href="boolean.md">Boolean Expression Rules</a> for more information.</td>
    </tr>
  <tr>
        <td><code>output_fields</code> (optional)</td>
        <td>Name of the field to return. Vector field is not supported in current release.</td>
    </tr>
    <tr>
        <td><code>consistency_level</code> (optional)</td>
        <td>Consistency level of the search.</td>
    </tr>
    </tbody>
</table>

<table class="language-javascript">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>collection_name</code></td>
        <td>Name of the collection to search in.</td>
    </tr>
    <tr>
    <td><code>search_params</code></td>
    <td>Parameters (as an object) used for search.</td>
  </tr>
    <tr>
    <td><code>vector</code> / <code>vectors</code></td>
    <td>Vectors to search with. Note that you should provide a list of floats if you choose to use <code>vector</code>. Otherwise, you should provide a list of float lists.</td>
  </tr>
  <tr>
        <td><code>vector_type</code></td>
        <td>Pre-check of binary or float vectors. <code>100</code> for binary vectors and <code>101</code> for float vectors.</td>
    </tr>
    <tr>
        <td><code>limit</code> (optional)</td>
        <td>Number of the most similar results to return. The sum of this value and <code>offset</code> should be less than 16384.</td>
    </tr>
    <tr>
        <td><code>offset</code> (optional)</td>
        <td>Number of entities to skip. The sum of this value of <code>limit</code> should be less than 16384.</td>
    </tr>
    <tr>
        <td><code>filter</code> (optional)</td>
        <td>Boolean expression used to filter attribute. See <a href="boolean.md">Boolean Expression Rules</a> for more information.</td>
    </tr>
  <tr>
        <td><code>output_fields</code> (optional)</td>
        <td>Name of the field to return. Vector field is not supported in current release.</td>
    </tr>
    </tbody>
</table>

<table class="language-go">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    <th>Options</th>
    </tr>
    </thead>
    <tbody>
  <tr>
    <td><code>ctx</code></td>
    <td>Context to control API invocation process.</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td><code>CollectionName</code></td>
    <td>Name of the collection to load.</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td><code>partitionNames</code></td>
    <td>List of names of the partitions to load. All partitions will be searched if it is left empty.</td>
    <td>N/A</td>
  </tr>
  <tr>
        <td><code>expr</code></td>
        <td>Boolean expression used to filter attribute.</td>
    <td>See <a href="boolean.md">Boolean Expression Rules</a> for more information.</td>
    </tr>
  <tr>
        <td><code>output_fields</code></td>
        <td>Name of the field to return.</td>
    <td>Vector field is not supported in current release.</td>
    </tr>
  <tr>
    <td><code>vectors</code></td>
    <td>Vectors to search with.</td>
    <td>N/A</td>
  </tr>
  <tr>
        <td><code>vectorField</code></td>
        <td>Name of the field to search on.</td>
    <td>N/A</td>
    </tr>
  <tr>
        <td><code>metricType</code></td>
        <td>Metric type used for search.</td>
    <td>This parameter must be set identical to the metric type used for index building.</td>
    </tr>
  <tr>
        <td><code>topK</code></td>
        <td>Number of the most similar results to return. The sum of this value and that of <code>offset</code> in <code>WithOffset</code> of <code>opts</code> should be less than 16384.</td>
    <td>N/A</td>
    </tr>
  <tr>
        <td><code>sp</code></td>
        <td><code>entity.SearchParam<code> specific to the index.</td>
    <td>N/A</td>
    </tr>
    </tbody>
</table>

<table class="language-java">
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    <th>Options</th>
    </tr>
    </thead>
    <tbody>
    <tr>
    <td><code>CollectionName</code></td>
    <td>Name of the collection to load.</td>
    <td>N/A</td>
  </tr>
  <tr>
        <td><code>MetricType</code></td>
        <td>Metric type used for search.</td>
    <td>This parameter must be set identical to the metric type used for index building.</td>
    </tr>
  <tr>
        <td><code>OutFields</code></td>
        <td>Name of the field to return.</td>
    <td>Vector field is not supported in current release.</td>
    </tr>
  <tr>
    <td><code>Vectors</code></td>
    <td>Vectors to search with.</td>
    <td>N/A</td>
  </tr>
<tr>
        <td><code>VectorFieldName</code></td>
        <td>Name of the field to search on.</td>
    <td>N/A</td>
    </tr>
  <tr>
        <td><code>Expr</code></td>
        <td>Boolean expression used to filter attribute.</td>
    <td>See <a href="boolean.md">Boolean Expression Rules</a> for more information.</td>
    </tr>
  <tr>
        <td><code>ConsistencyLevel</code></td>
        <td>The consistency level used in the query.</td>
      <td><code>STRONG</code>, <code>BOUNDED</code>, and<code>EVENTUALLY</code>.</td>
    </tr>
    </tbody>
</table>

<table class="language-csharp">
    <thead>
        <tr>
            <th>Parameter</th>
            <th>Description</th>
            <th>Options</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>collectionName</code></td>
            <td>Name of the collection to load.</td>
            <td>N/A</td>
        </tr>
        <tr>
            <td><code>metricType</code></td>
            <td>Metric type used for search.</td>
            <td>This parameter must be set identical to the metric type used for index building.</td>
        </tr>
        <tr>
            <td><code>vectors</code></td>
            <td>Vectors to search with.</td>
            <td>N/A</td>
        </tr>
        <tr>
            <td><code>vectorFieldName</code></td>
            <td>Name of the field to search on.</td>
            <td>N/A</td>
        </tr>
        <tr>
            <td><code>limit</code></td>
            <td>Number of records to return.</td>
            <td>N/A</td>
        </tr>
    </tbody>
</table>

Check the primary key values of the most similar vectors and their distances.

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
  <a href="#csharp">C#</a>
</div>

```python
results[0].ids
results[0].distances
```

```javascript
console.log(results.results)
```

```go
fmt.Printf("%#v\n", searchResult)
for _, sr := range searchResult {
    fmt.Println(sr.IDs)
    fmt.Println(sr.Scores)
}
```

```java
SearchResultsWrapper wrapperSearch = new SearchResultsWrapper(respSearch.getData().getResults());
System.out.println(wrapperSearch.getIDScore(0));
System.out.println(wrapperSearch.getFieldData("book_id", 0));
```

```csharp
// # get the IDs of all returned hits
Console.WriteLine(results.Ids.LongIds)
// alternative Console.WriteLine(results.Ids.StringIds)

// get the scores to the query vector from all returned hits
foreach (var score in results.Scores.ToList()) {
    Console.WriteLine(score);
};
```

Release the collection loaded in Milvus to reduce memory consumption when the search is completed.

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
  <a href="#csharp">C#</a>
</div>

```python
collection.release()
```

```javascript
await milvusClient.releaseCollection({  collection_name: "book",});
```

```go
err := milvusClient.ReleaseCollection(
    context.Background(),                            // ctx
    "book",                                          // CollectionName
)
if err != nil {
    log.Fatal("failed to release collection:", err.Error())
}
```

```java
milvusClient.releaseCollection(
        ReleaseCollectionParam.newBuilder()
                .withCollectionName("book")
                .build());
```

```csharp
var collection = milvusClient.GetCollection("book").ReleaseAsync();
```

## Limits
|Feature|Maximum limit|
|---|---|
|Length of a collection name|255 characters|
|Number of partitions in a collection|4,096|
|Number of fields in a collection|256|
|Number of shards in a collection|256|
|Dimensions of a vector|32,768|
|Top K|16,384|
|Target input vectors|16,384|


## What's next

- Learn more basic operations of Milvus:
  - [Query vectors](query.md)
  - [Conduct a hybrid search](hybridsearch.md)
  - [Search with Time Travel](timetravel.md)

- Explore API references for Milvus SDKs:

  - [PyMilvus API reference](/api-reference/pymilvus/v2.2.x/About.md)
  - [Node.js API reference](/api-reference/node/v2.2.x/About.md)
  - [Go API reference](/api-reference/go/v2.2.x/About.md)
  - [Java API reference](/api-reference/java/v2.2.x/About.md)


