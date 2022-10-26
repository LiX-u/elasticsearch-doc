# 当节点离开时延迟分配

不论什么原因（有意还是无意）当一个节点离开集群时，主节点的反应是：

- 将副本分片升级为主分片，以替换节点上的任何主分片。

- 分配副本分片以替换丢失的副本（假设有足够的节点）。

- 在其余节点上均匀地重新平衡分片。

这些操作旨在通过确保每个分片尽快完全复制，以保护集群避免数据丢失。

即使我们在[节点级别](/set_up_elasticsearch/configuring_elasticsearchindex_recovery_settings)和[集群级别](/set_up_elasticsearch/configuring_elasticsearch/cluster_level_shard_allocation_and_routing_settings#集群级分片分配设置)限制并发恢复，这种“分片洗牌”仍然会给集群带来大量额外负载，如果丢失的节点可能会很快返回，这可能是不必要的。想象以下场景：

- 节点 5 丢失网络连接。
- 对于曾是节点 5 上的主分片的副本分片，主节点将把它们提升为主分片。
- 主节点将分配新的副本到集群中的其他节点。
- 每个新副本通过网络生成主分片的完整复制。
- 更多的分片移动到不同的节点以重平衡集群。
- 节点 5 几分钟后返回。
- 主节点通过分配分片到节点 5 以重平衡集群。

如果主节点仅等待了几分钟，那么丢失的分片就可以用最少的网络流量重新分配给节点 5。对于已自动[同步刷新](/rest_apis/index_apis/synced_flush)的空间分片（未接收索引请求的分片），此过程将更快。

由于一个节点已离开，副本分片分配变为未分配，可以使用 `index.unassigned.node_left.delayed_timeout` 动态设置，默认为 `1m`。

这个设置可以活动索引（或者所有索引）上更新：

```bash
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```

如果允许延迟分配，以上的场景会变成这样：

- 节点 5 丢失网络连接。
- 对于曾是节点 5 上的主分片的副本分片，主节点将把它们提升为主分片。
- 主节点记录日志消息，未分配的分片分配已延迟，以及延迟时长。
- 集群保持黄色状态，由于存在未分配的副本分片。
- 在 `timeout`（超时） 到期前，节点 5 在几分钟后返回。
- 丢失的副本重新分配给节点 5（且同步刷新的分片几乎可以立即恢复）。

::: tip 提示
此设置不会影响将副本升级为主分片，也不会影响以前未分配副本的分配。特别是，延迟分配在集群重启后不会生效。另外，在主节点故障切换的情况下，已消耗的延迟时长会被忽视（即，重置为完全初始的延迟）。
:::

## 分片迁移取消

如果延迟分配超时，主节点分配丢失的分片到另一个节点，此节点将开始恢复。如果丢失的节点重加入集群，且它的分片仍然有与主分片相同的同步id，那么分片迁移将取消，并使用同步的分片进行恢复。

同于这个原因，默认的 `timeout`（超时）只设置为 1 分钟：即使分片迁移开始了，取消恢复而采用同步的分片也是低成本的。

## 监控延迟的未分配分片

按照超时设置延迟分配的分片数量，可以通过[集群健康 API](/rest_apis/cluster_apis/cluster_health)查看：

```bash
GET _cluster/health
```

- 请求会返回 `delayed_unassigned_shards` 值。

## 永久移除节点

如果一个节点不打算返回，且你希望 Elasticsearch 立即分配丢失的分片，只需要将超时设置为 0 ：

```bash
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "0"
  }
}
```

一旦丢失的分片开始恢复，就可以重置超时。

> [原文链接](https://www.elastic.co/guide/en/elasticsearch/reference/current/delayed-allocation.html)
