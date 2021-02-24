# es 更新延迟问题

[es 近实时搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/near-real-time.html)

## 瞎解释

详情还是看上面的说明吧，总而言之，es在更新之后并不是立即可见(可查询)的，会有1s的延迟，可以通过设置`refresh_interval`参数来修改刷新的间隔。

但是在实际应用中，1s的延迟已经算是很久了，在java high level client中，为`index`、`insert`、`update`、`bulk` 提供了`setRefreshPolicy`方法，用于设置数据更改后的刷新策略。

主要是三个参数`IMMEDIATE`、`NONE`、`WAIT_UNTIL`：

1. `NONE`:
   **Don’t refresh after this request. The default.**
   这是默认的一种方式，调用request修改以后，并不进行强制刷新，刷新的时间间隔为refresh_interval设置的参数。设置方式有以下两种。

```
// 1.
request.setRefreshPolicy(WriteRequest.RefreshPolicy.NONE);
// 2.
request.setRefreshPolicy("false");
1234
```

1. `IMMEDIATE`：
   强制刷新相关的主分片和副分片（而不是整个索引），使更新的分片状态变为可搜索。在使用之前，一定要仔细考虑使用该参数会不会导致性能不佳。
   **Force a refresh as part of this request. This refresh policy does not scale for high indexing or search throughput but is useful to present a consistent view to for indices with very low traffic. And it is wonderful for tests!**
   强制刷新作为请求的一部分。这种方式并不适用于索引和查询高吞吐量的场景，但是作为流量小时提供一致性的视图的确是很使用的。一般用来测试…?使用方式：

```
// 1.
request.setRefreshPolicy(WriteRequest.RefreshPolicy.IMMEDIATE);
// 2.
request.setRefreshPolicy("true");
1234
```

1. `WAIT_UNTIL`:
   在返回请求结果之前，会等待刷新请求所做的更改（类似于阻塞…?）。并不是强制立即刷新，而是等待刷新发生。Elasticsearch会自动刷新已更改每个index.refresh_interval的分片，默认为一秒。该设置是动态的。
   **Leave this request open until a refresh has made the contents of this request visible to search. This refresh policy is compatible with high indexing and search throughput but it causes the request to wait to reply until a refresh occurs.**
   请求持续为打开状态，知道修改的内容变为可搜索为止。此刷新策略与高索引和搜索吞吐量兼容，但它会导致请求等待回复，直到刷新发生。使用方式：

```
// 1.
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
// 2.
request.setRefreshPolicy("wait_for");
```