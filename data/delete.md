# 刪除一個文檔

刪除文檔的基本模式和之前的基本一樣，只不過是需要更換成`DELETE`方法：

```js
DELETE /website/blog/123
```
如果文檔存在，那麼Elasticsearch就會返回一個`200 OK`的HTTP響應碼，返回的結果就會像下面展示的一樣。請注意`_version`的數字已經增加了。

```js
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```
如果文檔不存在，那麼我們就會得到一個`404 Not Found`的響應碼，返回的內容就會是這樣的：

```js
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```
儘管文檔並不存在（`"found"`值爲`false`），但是`_version`的數值仍然增加了。這個就是內部管理的一部分，它保證了我們在多個節點間的不同操作的順序都被正確標記了。

****

正如我在《更新》一章中提到的，刪除一個文檔也不會立即生效，它只是被標記成已刪除。Elasticsearch將會在你之後添加更多索引的時候纔會在後臺進行刪除內容的清理。

****

