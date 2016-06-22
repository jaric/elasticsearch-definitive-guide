# 索引一個文檔

文檔通過`索引`API被_索引_——存儲並使其可搜索。但是最開始我們需要決定我們將文檔存儲在哪裏。正如之前提到的，一篇文檔通過`_index`, `_type`以及`_id`來確定它的唯一性。我們可以自己提供一個`_id`，或者也使用`index`API 幫我們生成一個。


## 使用自己的ID

如果你的文檔擁有天然的標示符（例如`user_account`字段或者文檔中其他的標識值），這時你就可以提供你自己的`_id`，這樣使用`index`API：

```js
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```
幾個例子。如果我們的索引叫做`"website"`，我們的類型叫做 `"blog"`，然後我們選擇`"123"`作爲ID的編號。這時，請求就是這樣的：
```js
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```

Elasticsearch返回內容：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
}
```
這個返回值意味着我們的索引請求已經被成功創建，其中還包含了`_index`, `_type`以及`_id`的元數據，以及一個新的元素`_version`。

在Elasticsearch中，每一個文檔都有一個版本號碼。每當文檔產生變化時（包括刪除），`_version`就會增大。在《版本控制》中，我們將會詳細講解如何使用`_version`的數字來確認你的程序不會隨意替換掉不想覆蓋的數據。

### 自增ID

如果我們的數據中沒有天然的標示符，我們可以讓Elasticsearch爲我們自動生成一個。請求的結構發生了變化：我們把`PUT`——“把文檔存儲在這個地址中”變量變成了`POST`——“把文檔存儲在這個**地址下**”。

這樣一來，請求中就只包含 `_index`和`_type`了：

```js
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

這次的反饋和之前基本一樣，只有`_id`改成了系統生成的自增值:

```
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "wM0OSFhDQXGZAWDf0-drSA",
   "_version":  1,
   "created":   true
}
```
自生成ID是由22個字母組成的，安全
_universally unique identifiers_ 或者被稱爲[UUIDs](http://baike.baidu.com/view/1052579.htm?fr=aladdin)。




