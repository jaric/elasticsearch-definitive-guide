# 創建一個文檔

當我們索引一個文檔時，如何確定我們是創建了一個新的文檔還是覆蓋了一個已經存在的文檔呢？


請牢記`_index`,`_type`以及`_id`組成了唯一的文檔標記，所以爲了確定我們創建的是全新的內容，最簡單的方法就是使用`POST`方法，讓Elasticsearch自動創建不同的`_id`：

```js
POST /website/blog/
{ ... }
```

然而，我們可能已經決定好了`_id`，所以需要告訴Elasticsearch只有當`_index`，`_type`以及`_id`這3個屬性全部相同的文檔不存在時才接受我們的請求。實現這個目的有兩種方法，他們實質上是一樣的，你可以選擇你認爲方便的那種：

第一種是在查詢中添加`op_type`參數：

```js
PUT /website/blog/123?op_type=create
{ ... }
```

或者在請求最後添加 `/_create`:

```js
PUT /website/blog/123/_create
{ ... }
```

如果成功創建了新的文檔，Elasticsearch將會返回常見的元數據以及`201 Created`的HTTP反饋碼。

而如果存在同名文件，Elasticsearch將會返回一個`409 Conflict`的HTTP反饋碼，以及如下方的錯誤信息：

```js
{
  "error" : "DocumentAlreadyExistsException[[website][4] [blog][123]:
             document already exists]",
  "status" : 409
}
```

