# 集羣健康

在 Elasticsearch 集羣中可以監控統計很多信息，其中最重要的就是：**集羣健康(cluster health)**。它的 `status` 有 `green`、`yellow`、`red` 三種；

```
GET /_cluster/health
```

在一個沒有索引的空集羣中，它將返回如下信息：

```Js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
1.  `status` 是我們最應該關注的字段。

`status` 可以告訴我們當前集羣是否處於一個可用的狀態。三種顏色分別代表：

| 狀態     | 意義                                     |
| -------- | ---------------------------------------- |
| `green`  | 所有主分片和從分片都可用               |
| `yellow` | 所有主分片可用，但存在不可用的從分片 |
| `red`    | 存在不可用的主要分片                 |

在接下來的章節，我們將學習一下什麼是**主要分片(primary shard)** 和 **從分片(replica shard)**，並說明這些狀態在實際環境中的意義。
