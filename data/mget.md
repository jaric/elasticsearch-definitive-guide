# 獲取多個文檔

儘管Elasticsearch已經很快了，但是它依舊可以更快。你可以將多個請求合併到一個請求中以節省網絡開銷。如果你需要從Elasticsearch中獲取多個文檔，你可以使用_multi-get_ 或者 `mget` API來取代一篇又一篇文檔的獲取。

`mget`API需要一個`docs`數組，每一個元素包含你想要的文檔的`_index`, `_type`以及`_id`。你也可以指定`_source`參數來設定你所需要的字段：

```js
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```
返回值包含了一個`docs`數組，這個數組以請求中指定的順序每個文檔包含一個響應。每一個響應都和獨立的`get`請求返回的響應相同：


```js
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```
如果你所需要的文檔都在同一個`_index`或者同一個`_type`中，你就可以在URL中指定一個默認的`/_index`或是`/_index/_type`。

你也可以在單獨的請求中重寫這個參數：

```js
GET /website/blog/_mget
{
   "docs" : [
      { "_id" : 2 },
      { "_type" : "pageviews", "_id" :   1 }
   ]
}
```
事實上，如果所有的文檔擁有相同的`_index` 以及 `_type`，直接在請求中添加`ids`的數組即可：

```js
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```
請注意，我們所請求的第二篇文檔不存在，這是就會返回如下內容：

```js
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  <1>
    }
  ]
}
```
1. 文檔沒有被找到。

當第二篇文檔沒有被找到的時候也不會影響到其它文檔的獲取結果。每一個文檔都會被獨立展示。

注意：上方請求的HTTP狀態碼依舊是`200`，儘管有個文檔沒有找到。事實上，即使**所有的**文檔都沒有被找到，響應碼也依舊是`200`。這是因爲`mget`這個請求本身已經成功完成。要確定獨立的文檔是否被成功找到，你需要檢查`found`標識。
