## 🔁 Milvus Data & Query Flows

### High-Level Flow (Insert + Index)

```text
Client
  ↓
Proxy
  ↓
DataCoord
  ↓
DataNode
  ↓
S3 / MinIO (flushed segments)
  ↓
IndexCoord
  ↓
IndexNode
  ↓
S3 / MinIO (index files)
```

### Query Flow
```text
Client
  ↓
Proxy
  ↓
QueryCoord
  ↓
QueryNode
  ↓
S3 / MinIO (loads segments if needed)
  ↓
ANN Search
  ↓
Results → Proxy → Client
```

### Index Building Flow
```text
Client inserts vectors
  ↓
Proxy
  ↓
DataCoord
  ↓
DataNode (flushes to storage)
  ↓
IndexCoord
  ↓
IndexNode builds index
  ↓
Stores index in S3/MinIO
```

### DataNode Flow
```text
Client
  ↓
Proxy
  ↓
DataCoord
  ↓
DataNode
  ↓
S3 / MinIO
  ↓
IndexCoord → IndexNode
```

### Messaging Flow
```text
Milvus Component (e.g. DataNode)
  ↓
Publishes event
  ↓
Pulsar Broker
  ↓
Stores in BookKeeper (WAL)
```


> Note: `mixCoord` is only used in development. In production, each `Coord` (e.g. `DataCoord`, `QueryCoord`) runs as a separate pod.


