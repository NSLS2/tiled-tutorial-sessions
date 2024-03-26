# Tiled Tutorial Sessions

## This Week


- Using [HTTPie](https://httpie.io/) commandline tool to explore the Tiled HTTP API

Install and use HTTPie.

```bash
pip install httpie
http https://example.com
```

Start a Tiled server.

```bash
tiled serve catalog --temp --api-key secret
```

Public server landing page (HTML)

```bash
http :8000
```

Public server info (JSON)

```bash
http :8000/api/v1  # missing trailing /
http :8000/api/v1/
```

The `/metadata` and `/search` endpoints

```bash
http :8000/api/v1/metadata/   # missing authentication
http :8000/api/v1/metadata/ 'Authorization:Apikey secret'
http :8000/api/v1/search/ 'Authorization:Apikey secret'
```

Using the Python client to upload some data:

```python=
from tiled.client import from_uri
import dask.array
import dask.dataframe
import numpy
import pandas

c = from_uri('http://localhost:8000', api_key='secret')

arr = numpy.ones((5, 5))
c.write_array(arr, key="x")
df = pandas.DataFrame({"A": [1,2,3], "B": [4,5,6], "C": [7,8,9]})
c.write_dataframe(df, key="y")
z = c.create_container("z")
z.write_array([1], key="stuff")
z.write_array([10, 20, 30], key="things")
da = dask.array.from_array(numpy.mgrid[:5, :5][0] * 10 + numpy.mgrid[:5, :5][1], chunks=(3, 3))
c.write_array(da, key='da')
ddf = dask.dataframe.from_pandas(df, npartitions=2)
c.write_dataframe(ddf, key='ddf')
```

Use `/search` to list contents.

```bash
http :8000/api/v1/search/ 'Authorization:Apikey secret' 
```

```bash
pip install jq
http :8000/api/v1/search/ 'Authorization:Apikey secret' | jq .data[].id
http :8000/api/v1/search/ 'Authorization:Apikey secret' | jq .data[].links.self
http :8000/api/v1/search/z 'Authorization:Apikey secret' | jq .data[].id
```

Describe an array.

```
http :8000/api/v1/metadata/x 'Authorization:Apikey secret'
```

Fetch array data.

```bash
http :8000/api/v1/array/full/x 'Authorization:Apikey secret'
```

Request a non-default format.

```bash
http :8000/api/v1/array/full/x 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/array/full/x?format=text/csv' 'Authorization:Apikey secret'
http ':8000/api/v1/array/full/x?format=csv' 'Authorization:Apikey secret'
```

```bash
http :8000/api/v1/array/full/x 'Authorization:Apikey secret' 'Accept:nonsense'
```

Slice.

```bash
http ':8000/api/v1/array/full/x?slice=:3' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/array/full/x?slice=:,:3' 'Authorization:Apikey secret' 'Accept:text/csv'
```

Use the block endpoint.

```bash!
# See the full array in one request.
http ':8000/api/v1/array/full/da' 'Authorization:Apikey secret' 'Accept:text/csv'
# Request individual blocks.
http ':8000/api/v1/array/block/da?block=0,0' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/array/block/da?block=0,1' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/array/block/da?block=1,0' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/array/block/da?block=0,0' 'Authorization:Apikey secret' 'Accept:text/csv'
# Slice within a block.
http ':8000/api/v1/array/block/da?block=0,0&slice=1:,1:' 'Authorization:Apikey secret' 'Accept:text/csv'
```

Describe a table.

```bash
http ':8000/api/v1/metadata/y' 'Authorization:Apikey secret'
```

Fetch tabular data.

```bash=
http ':8000/api/v1/table/full/y' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/table/full/y?column=A' 'Authorization:Apikey secret' 'Accept:text/csv'
 http ':8000/api/v1/table/full/y?column=A&column=B' 'Authorization:Apikey secret' 'Accept:text/csv'
 ```

Fetch tabular data by partition:

```bash=
http ':8000/api/v1/table/partition/ddf?partition=0' 'Authorization:Apikey secret' 'Accept:text/csv'
http ':8000/api/v1/table/partition/ddf?partition=1' 'Authorization:Apikey secret' 'Accept:text/csv'
# Combine with column selection
 http ':8000/api/v1/table/partition/ddf?partition=1&column=A' 'Authorization:Apikey secret' 'Accept:text/csv'
```


## Previously on...

### 2024-03-15

- Registration
- Data Sources

### 2024-03-07 
