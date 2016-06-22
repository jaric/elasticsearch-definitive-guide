# 批量更高效

與`mget`能同時允許幫助我們獲取多個文檔相同，`bulk` API可以幫助我們同時完成執行多個請求，比如：`create`，`index`, `update`以及`delete`。當你在處理類似於log等海量數據的時候，你就可以一下處理成百上千的請求，這個操作將會極大提高效率。

`bulk`的請求主體的格式稍微有些不同：

```js
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```
這種格式就類似於一個用`"\n"`字符來連接的單行json一樣。下面是兩點注意事項：

* 每一行都結尾處都必須有換行字符`"\n"`，**最後一行也要有**。這些標記可以有效地分隔每行。


* 這些行裏不能包含非轉義字符，以免干擾數據的分析 — — 這也意味着JSON**不能**是pretty-printed樣式。

**************************************************
> ###TIP

在《bulk格式》一章中，我們將解釋爲何`bulk` API要使用這種格式。

**************************************************

_action/metadata_ 行指定了將要在**哪個文檔**中執行**什麼操作**。

其中_action_必須是`index`, `create`, `update`或者`delete`。_metadata_ 需要指明需要被操作文檔的`_index`, `_type`以及`_id`，例如刪除命令就可以這樣填寫：

```js
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```
在你進行`index`以及`create`操作時，_request body_ 行必須要包含文檔的`_source`數據——也就是文檔的所有內容。

同樣，在執行`update` API: `doc`, `upsert`,`script`的時候，也需要包含相關數據。而在刪除的時候就不需要_request body_行。

```js
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```

如果沒有指定`_id`，那麼系統就會自動生成一個ID：

```js
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```

完成以上所有請求的`bulk`如下：

```js
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} <1>
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } <2>
```

1. 注意`delete`操作是如何處理_request body_的,你可以在它之後直接執行新的操作。

2. 請記住最後有換行符

Elasticsearch會返回含有`items`的列表、它的順序和我們請求的順序是相同的：


```js
{
   "took": 4,
   "errors": false, <1>
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}}
```
1. 所有的請求都被成功執行。

每一個子請求都會被單獨執行，所以一旦有一個子請求失敗了，並不會影響到其他請求的成功執行。如果一旦出現失敗的請求，`error`就會變爲`true`，詳細的錯誤信息也會出現在返回內容的下方：


```js
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```
請求中的`create`操作失敗，因爲`123`已經存在，但是之後針對文檔`123`的`index`操作依舊被成功執行：

```js
{
   "took": 3,
   "errors": true, <1>
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, <2>
            "error":    "DocumentAlreadyExistsException <3>
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 <4>
      }}
   ]
}
```
1. 至少有一個請求錯誤發生。
2. 這條請求的狀態碼爲`409 CONFLICT`。
3. 錯誤信息解釋了導致錯誤的原因。
4. 第二條請求的狀態碼爲`200 OK`。

這也更好地解釋了`bulk`請求是獨立的，每一條的失敗與否 都不會影響到其他的請求。

### 能省就省

或許你在批量導入大量的數據到相同的`index`以及`type`中。每次都去指定每個文檔的metadata是完全沒有必要的。在`mget` API中，`bulk`請求可以在URL中聲明`/_index` 或者`/_index/_type`：

```js
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```
你依舊可以在metadata行中使用`_index`以及`_type`來重寫數據，未聲明的將會使用URL中的配置作爲默認值：

```js
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```

### 最大有多大？

整個數據將會被處理它的節點載入內存中，所以如果請求量很大的話，留給其他請求的內存空間將會很少。`bulk`應該有一個最佳的限度。超過這個限制後，性能不但不會提升反而可能會造成宕機。

最佳的容量並不是一個確定的數值，它取決於你的硬件，你的文檔大小以及複雜性，你的索引以及搜索的負載。幸運的是，這個_平衡點_ 很容易確定：

試着去批量索引越來越多的文檔。當性能開始下降的時候，就說明你的數據量太大了。一般比較好初始數量級是1000到5000個文檔，或者你的文檔很大，你就可以試着減小隊列。
有的時候看看批量請求的物理大小是很有幫助的。1000個1KB的文檔和1000個1MB的文檔的差距將會是天差地別的。比較好的初始批量容量是5-15MB。
