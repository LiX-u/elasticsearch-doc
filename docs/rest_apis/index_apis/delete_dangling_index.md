# 删除悬空索引 API

删除一个悬空索引。

## 请求

```bash
DELETE /_dangling/<index-uuid>?accept_data_loss=true
```

## 前置条件

- 如果 Elasticsearch 安全特性启用，你必须有 `manage` [集群权限](/secure_the_elastic_statck/user_authorization/security_privileges#集群权限)来使用此 API。

## 描述

如果 Elasticsearch 遇到当前集群状态中缺少的索引数据，则认为这些索引处于悬空状态。例如，如果在 Elasticsearch 节点脱机时删除多个 `cluster.index.tombstones.size` 索引，则可能会发生这种情况。

通过引用其 UUID 可以删除悬空索引。使用[列出悬空索引 API](/rest_apis/index_apis/list_dangling_indices) 定位索引的 UUID。

## 路径参数

- `<index-uuid>`

  （必需的，字符串）待删除索引的 UUID，你可以通过[列出悬空索引 API](/rest_apis/index_apis/list_dangling_indices) 找到它。

## 查询参数

- `accept_data_loss`

  （必需的，布尔值）此字段必须设置为 `true` 才能导入悬空索引。由于 Elasticsearch 无法知道悬空索引数据来自何处，也无法确定哪些分片副本是新的，哪些是旧的，因此它无法保证导入的数据代表索引在集群中最后一次出现时的最新状态。

- `master_timeout`

  （可选，[时间单位](/rest_apis/api_convention/common_options#时间单位)）等待连接到主节点的时间。如果在超时过期前没有收到响应，则请求失败并返回错误。默认为 `30s`。

- `timeout`

  （可选，[时间单位](/rest_apis/api_convention/common_options#时间单位)）等待响应的时间。如果在超时过期之前没有收到响应，则请求失败并返回错误。默认为 `30s`。

> [原文链接](https://www.elastic.co/guide/en/elasticsearch/reference/current/dangling-index-delete.html)
