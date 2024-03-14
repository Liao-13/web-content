# Query

Conducts a vector query in a collection.

<div>
    <div style="display: inline-block; background: #026aca; font-size: 0.6em; border-radius: 10px; color: #ffffff; padding: 0.3em 1em;">
        <span>POST</span>
    </div>
    <span style="font-weight: bold;">  https://${MILVUS_HOST}:${MILVUS_PORT}/v1/vector/query</span>
</div>

## Example


Query entities that meet specific conditions:

```shell
curl --request POST \
     --url "${MILVUS_HOST}:${MILVUS_PORT}/v1/vector/query" \
     --header "Authorization: Bearer ${TOKEN}" \
     --header "accept: application/json" \
     --header "content-type: application/json" \
     -d '{
       "collectionName": "medium_articles",
       "outputFields": ["id", "title", "link"],
       "filter": "id in [443300716234671427, 443300716234671426]",
       "limit": 100,
       "offset": 0
     }'
```



## Request

### Parameters

- No query parameters required

- No path parameters required

### Request Body

```json
{
    "collectionName": "string",
    "dbName": "string",
    "filter": "string",
    "limit": "integer",
    "offset": "integer",
    "outputFields": []
}
```

| Parameter        | Description                                                                               |
|------------------|-------------------------------------------------------------------------------------------|
| `dbName`  | **string**<br>The name of the database.|
| `collectionName`  | **string**(required)<br>The name of the collection to which this operation applies.|
| `filter`  | **string**(required)<br>The filter used to find matches for the search.|
| `limit`  | **integer**<br>The maximum number of entities to return.<br/>The sum of this value and that of `offset` should be less than **16384**.<br>The value defaults to **100**.<br>The value ranges from **1** to **100**.|
| `offset`  | **integer**<br>The number of entities to skip in the search results.<br/>The sum of this value and that of `limit` should be less than **16384**.<br>The maximum value is **16384**.|
| `outputFields`  | **array**<br>An array of fields to return along with the search results.|

## Response

Returns the search results.

### Response Bodies

- Response body if we process your request successfully

```json
{
    "code": 200,
    "data": {}
}
```

- Response body if we failed to process your request

```json
{
    "code": integer,
    "message": string
}
```

### Properties

The properties in the returned response are listed in the following table.

| Property | Description                                                                                                                                 |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `code`   | **integer**<br>Indicates whether the request succeeds.<br><ul><li>`200`: The request succeeds.</li><li>Others: Some error occurs.</li></ul> |
| `data`  | **array**<br>A data array of objects. |
| `message`  | **string**<br>Indicates the possible reason for the reported error. |

## Possible Errors

| Error Code | Description |
| --- | --- |
| 800 | database not found |
| 1800 | user hasn't authenticate |
| 1801 | can only accept json format request |
| 1802 | missing required parameters |
| 1805 | fail to parse search result |
