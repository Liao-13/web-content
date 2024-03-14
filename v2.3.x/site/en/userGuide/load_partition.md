---
id: load_partition.md
related_key: load partition
summary: Learn how to load a partition into memory for search or query in Milvus.
---

# Load a Partition

This topic describes how to load a partition to memory. Loading partitions instead of the whole collection to memory can significantly reduce the memory usage. All search and query operations within Milvus are executed in memory.

Since version 2.3.0, Milvus has enhanced its partition operations and now supports cascading load and release operations. This means that you can perform any combination of the following operations:

- Load a collection that has already been loaded.
- Load a collection, and then load a specific partition in the collection.
- Load a partition, and then load the collection to which it belongs.
- Load a partition, and reload it before release.

Milvus allows users to load a partition as multiple replicas to utilize the CPU and memory resources of extra query nodes. This feature boosts the overall QPS and throughput with extra hardware. It is supported on PyMilvus in current release.

<div class="alert warning">
<ul>
<li>In current release, volume of the data to load must be under 90% of the total memory resources of all query nodes to reserve memory resources for execution engine.</li>
<li>In current release, all on-line query nodes will be divided into multiple replica groups according to the replica number specified by user. All replica groups shall have minimal memory resources to load one replica of the provided collection. Otherwise, an error will be returned.</li>
</ul>
</div>

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#go">GO</a>
  <a href="#javascript">Node.js</a>
</div>

```python
from pymilvus import Collection
collection = Collection("book")      # Get an existing collection.
collection.load(["novel"], replica_number=2)

# Or you can load a partition with the partition as an object
from pymilvus import Partition
partition = Partition("novel")       # Get an existing partition.
partition.load(replica_number=2)
```

```javascript
await milvusClient.loadPartitions({
  collection_name: "book",
  partition_names: ["novel"],
});
```

```go
err := milvusClient.LoadPartitions(
  context.Background(),   // ctx
  "book",                 // CollectionName
  []string{"novel"},      // partitionNames
  false                   // async
)
if err != nil {
  log.Fatal("failed to load partitions:", err.Error())
}
```

```java
milvusClient.loadPartitions(
  LoadPartitionsParam.newBuilder()
          .withCollectionName("book")
          .withPartitionNames(["novel"])
          .build()
);
```

<div style="display: none">

```shell
load -c book -p novel
```

``` curl
curl -X 'POST' \
  'http://localhost:9091/api/v1/partitions/load' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
    "collection_name": "book",
    "partition_names": ["novel"],
    "replica_number": 1
  }'
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
		<td><code>partition_name</code></td>
		<td>Name of the partition.</td>
	</tr>
    <tr>
		<td><code>replica_number</code> (optional)</td>
		<td>Number of the replica to load.</td>
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
		<td>Name of the collection to load partitions from.</td>
	</tr>
    <tr>
		<td><code>partition_names</code></td>
		<td>List of names of the partitions to load.</td>
	</tr>
	</tbody>
</table>

<table class="language-go">
	<thead>
        <tr>
            <th>Parameter</th>
            <th>Description</th>
        </tr>
	</thead>
	<tbody>
        <tr>
            <td><code>ctx</code></td>
            <td>Context to control API invocation process.</td>
        </tr>
        <tr>
            <td><code>CollectionName</code></td>
            <td>Name of the collection to load partitions from.</td>
        </tr>
        <tr>
            <td><code>partitionNames</code></td>
            <td>List of names of the partitions to load.</td>
        </tr>
        <tr>
            <td><code>async</code></td>
            <td>Switch to control sync/async behavior. The deadline of context is not applied in sync load.</td>
        </tr>
    </tbody>
</table>

<table class="language-java">
	<thead>
        <tr>
            <th>Parameter</th>
            <th>Description</th>
        </tr>
	</thead>
	<tbody>
        <tr>
            <td><code>CollectionName</code></td>
            <td>Name of the collection to load partitions from.</td>
        </tr>
        <tr>
            <td><code>PartitionNames</code></td>
            <td>List of names of the partitions to load.</td>
        </tr>
    </tbody>
</table>

<table class="language-shell" style="display: none">
    <thead>
        <tr>
            <th>Option</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>-c</td>
            <td>Name of the collection to load partitions from.</td>
        </tr>
        <tr>
            <td>-p (Multiple)</td>
            <td>The name of the partition to load.</td>
        </tr>
    </tbody>
</table>

<table class="language-curl" style="display: none">
	<thead>
	<tr>
		<th>Parameter</th>
		<th>Description</th>
	</tr>
	</thead>
	<tbody>
	<tr>
		<td><code>collection_name</code></td>
		<td>Name of the collection to load partitions from.</td>
	</tr>
    <tr>
		<td><code>partition_names</code></td>
		<td>List of names of the partitions to load.</td>
	</tr>
    <tr>
		<td><code>replica_number</code> (optional)</td>
		<td>Number of the replica to load.</td>
	</tr>
	</tbody>
</table>

## Get replica information

You can check the information of the loaded replicas.

```python
from pymilvus import Partition
partition = Partition("novel")       # Get an existing partition.
partition.load(replica_number=2)     # Load partition as 2 replicas
result = partition.get_replicas()
print(result)
```

## What's next

- Learn more basic operations of Milvus:
  - [Insert data into Milvus](insert_data.md)
  - [Build an index for vectors](build_index.md)
  - [Conduct a vector search](search.md)
  - [Conduct a hybrid search](hybridsearch.md)

