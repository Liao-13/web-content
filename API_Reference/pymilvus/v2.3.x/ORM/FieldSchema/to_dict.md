
# to_dict()

This operation converts a FieldSchema object to a dictionary representation.

## Request Syntax

```python
to_dict()
```

__PARAMETERS:__

None

__RETURN TYPE:__

_dict_

__RETURNS:__

The dictionary representation of the field schema.

__EXCEPTIONS:__

- __MilvusException__

    This exception will be raised when any error occurs during this operation.

## Examples

```python
from pymilvus import CollectionSchema, FieldSchema, DataType

# Create field schemas  
primary_key = FieldSchema(
    name="id",
    dtype=DataType.INT64,
    is_primary=True,
)
 
vector = FieldSchema(
    name="vector", 
    dtype=DataType.FLOAT_VECTOR,
    dim=768
)

# Get dictionary representation
primary_key_dict = primary_key.to_dict()
vector_dict = vector.to_dict()

print(primary_key_dict)
print(vector_dict)

# Output
# {'name': 'id', 'description': '', 'type': <DataType.INT64: 5>, 'is_primary': True, 'auto_id': False}
# {'name': 'vector', 'description': '', 'type': <DataType.FLOAT_VECTOR: 101>, 'params': {'dim': 768}}
```

