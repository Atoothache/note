# Elasticsearch 入门: _bulk 批量导入数据

#### 批量导入数据

使用 Elasticsearch Bulk API `/_bulk`批量 update

##### 步骤：

1. 需求：我希望批量导入一个 `movie` type 的名词列表到 `wordbank` index 索引。

2. 准备数据：

   根据[官方文档](https://link.jianshu.com?t=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fen%2Felasticsearch%2Freference%2Fcurrent%2Fdocs-bulk.html)，Json 数据要准备成这个格式的：

   

   ```json
   action_and_meta_data\n
   optional_source\n
   action_and_meta_data\n
   optional_source\n
   ....
   action_and_meta_data\n
   optional_source\n
   ```

   其中 action 需要是 `index`, `create`, `delete` and `update` 中的一个。

   接下来准备这样的数据：

   

   ```json
   {"index": {"_index": "wordbank", "_type": "movie", "_id": 1}}
   {"doc": {"name": "权力的游戏"}}
   {"index": {"_index": "wordbank", "_type": "movie", "_id": 2}}
   {"doc": {"name": "熊出没"}}
   ```

3. POST bulk

   

   ```bash
   curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary @movie_names
   ```

4. 批量 update 成功

   

   ```json
   {"took":50,"errors":false,"items":[{"index":{"_index":"wordbank","_type":"movie","_id":"1","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1,"status":201}},{"index":{"_index":"wordbank","_type":"movie","_id":"2","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1,"status":201}}]}
   ```

##### 遇到过的坑：

1. illegal_argument_exception:

   

   ```json
   {"error":{"root_cause":[{"type":"illegal_argument_exception","reason":"The bulk request must be terminated by a newline [\n]"}],"type":"illegal_argument_exception","reason":"The bulk request must be terminated by a newline [\n]"},"status":400}
   ```

   - 原因：批量导入的 json 文件最后必须要以`\n`结尾，也就是需要一个空行。
   - 解决：在 json 文件末尾加多一个回车。

2. header 问题：

   

   ```json
   {"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}
   ```

   - 原因：Elasticsearch 6.x 之后 curl 的 content-type 更严格了。
   - 解决：在 curl 命令后多加一条 `-H 'Content-Type: application/json'`

3. action_request_validation_exception：

   

   ```json
   {"error":{"root_cause":[{"type":"action_request_validation_exception","reason":"Validation Failed: 1: script or doc is missing;2: script or doc is missing;"}],"type":"action_request_validation_exception","reason":"Validation Failed: 1: script or doc is missing;2: script or doc is missing;"},"status":400}
   ```

   - 原因：bulk update 时，更新的文本需要放到 `"doc"` 字典下，另外 update 在这里就只是 update，如果文档不存在会报错。
   - 解决：

   

   ```json
   { "field1" : "value1", "field2" : "value2" }
   --> { "doc" : { "field1" : "value1", "field2" : "value2" } }
   ```

4. 不要直接在 terminal 把 curl 的结果显示出来

   - 原因：因为 curl 返回的结果是个单行 json 当批量处理条目多的时候，这个单行的 json 很长。而且`-s` 也silent 模式是不会把这个结果去掉的，因为 `-s` 是 curl 的参数，会屏蔽掉 curl 的 log，但 Elasticsearch 的返回 json 是不会被屏蔽掉的。

   - 解决：把输出结果导到文件

     

     ```bash
     curl -s 'http://example.com' > /dev/null
     ```

5. 据说不要重复指定 index 和 type：[来源](https://link.jianshu.com?t=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fcn%2Felasticsearch%2Fguide%2Fcurrent%2Fbulk.html)，可能是我数据量比较小，2w条，差距不大。不过前者确实省文档空间。

   推荐使用这种：

   

   ```bash
   POST /website/log/_bulk
   { "index": {}}
   { "event": "User logged in" }
   ```

   而不是这种：

   

   ```bash
   POST /_bulk
   { "index": { "_index": "website" , "_type": "blog" , }}
   { "title": "Overriding the default type" }
   ```

