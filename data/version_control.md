# 處理衝突

當你使用`索引`API來更新一個文檔時，我們先看到了原始文檔，然後修改它，最後一次性地將**整個新文檔**進行再次索引處理。Elasticsearch會根據請求發出的順序來選擇出最新的一個文檔進行保存。但是，如果在你修改文檔的同時其他人也發出了指令，那麼他們的修改將會丟失。

很長時間以來，這其實都不是什麼大問題。或許我們的主要數據還是存儲在一個關係數據庫中，而我們只是將爲了可以搜索，纔將這些數據拷貝到Elasticsearch中。或許發生多個人同時修改一個文件的概率很小，又或者這些偶然的數據丟失並不會影響到我們的正常使用。

但是有些時候如果我們丟失了數據就會出**大問題**。想象一下，如果我們使用Elasticsearch來存儲一個網店的商品數量。每當我們賣出一件，我們就會將這個數量減少一個。

突然有一天，老闆決定來個大促銷。瞬間，每秒就產生了多筆交易。並行處理，多個進程來處理交易：

![無併發控制的後果](/images/03-01_concurrency.png "無併發控制的後果")

`web_1`中`庫存量`的變化丟失的原因是`web_2`並不知道它所得到的`庫存量`數據是是過期的。這樣就會導致我們誤認爲還有很多貨存，最終顧客就會對我們的行爲感到失望。

當我們對數據修改得越頻繁，或者在讀取和更新數據間有越長的空閒時間，我們就越容易丟失掉我們的數據。

以下是兩種能避免在併發更新時丟失數據的方法：

### 悲觀併發控制（PCC）

這一點在關係數據庫中被廣泛使用。假設這種情況很容易發生，我們就可以阻止對這一資源的訪問。典型的例子就是當我們在讀取一個數據前先鎖定這一行，然後確保只有讀取到數據的這個線程可以修改這一行數據。

### 樂觀併發控制（OCC）

Elasticsearch所使用的。假設這種情況並不會經常發生，也不會去阻止某一數據的訪問。然而，如果基礎數據在我們讀取和寫入的間隔中發生了變化，更新就會失敗。這時候就由程序來決定如何處理這個衝突。例如，它可以重新讀取新數據來進行更新，又或者它可以將這一情況直接反饋給用戶。

## 樂觀併發控制

Elasticsearch是分佈式的。當文檔被創建、更新或者刪除時，新版本的文檔就會被複制到集羣中的其他節點上。Elasticsearch即是同步的又是異步的，也就是說複製的請求被平行發送出去，然後可能會**混亂地**到達目的地。這就需要一種方法能夠保證新的數據不會被舊數據所覆蓋。

我們在上文提到每當有`索引`、`put`和`刪除`的操作時，無論文檔有沒有變化，它的`_version`都會增加。Elasticsearch使用`_version`來確保所有的改變操作都被正確排序。如果一箇舊的版本出現在新版本之後，它就會被忽略掉。

我們可以利用`_version`的優點來確保我們程序修改的數據衝突不會造成數據丟失。我們可以按照我們的想法來指定`_version`的數字。如果數字錯誤，請求就是失敗。

我們來創建一個新的博文:

```js
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```
反饋告訴我們這是一個新建的文檔，它的`_version`是`1`。假設我們要編輯它，把這個數據加載到網頁表單中，修改完畢然後保存新版本。

首先我們先要得到文檔：

```js
GET /website/blog/1
```


返回結果顯示`_version`爲`1`：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```
現在，我們試着重新索引文檔以保存變化，我們這樣指定了`version`的數字：

```js
PUT /website/blog/1?version=1 <1>
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
1. 我們只希望當索引中文檔的`_version`是`1`時，更新才生效。

請求成功相應，返回內容告訴我們`_version`已經變成了`2`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```
然而，當我們再執行同樣的索引請求，並依舊指定`version=1`時，Elasticsearch就會返回一個`409 Conflict`的響應碼，返回內容如下：

```js
{
  "error" : "VersionConflictEngineException[[website][2] [blog][1]:
             version conflict, current [2], provided [1]]",
  "status" : 409
}
```
這裏面指出了文檔當前的`_version`數字是`2`，而我們要求的數字是`1`。

我們需要做什麼取決於我們程序的需求。比如我們可以告知用戶已經有其它人修改了這個文檔，你應該再保存之前看一下變化。而對於上文提到的`庫存量`問題，我們可能需要重新讀取一下最新的文檔，然後顯示新的數據。

所有的有關於更新或者刪除文檔的API都支持`version`這個參數，有了它你就通過修改你的程序來使用樂觀併發控制。


### 使用外部系統的版本

還有一種常見的情況就是我們還是使用其他的數據庫來存儲數據，而Elasticsearch只是幫我們檢索數據。這也就意味着主數據庫只要發生的變更，就需要將其拷貝到Elasticsearch中。如果多個進程同時發生，就會產生上文提到的那些併發問題。

如果你的數據庫已經存在了版本號碼，或者也可以代表版本的`時間戳`。這是你就可以在Elasticsearch的查詢字符串後面添加`version_type=external`來使用這些號碼。版本號碼必須要是大於零小於`9.2e+18`（Java中long的最大正值）的整數。

Elasticsearch在處理外部版本號時會與對內部版本號的處理有些不同。它不再是檢查`_version`是否與請求中指定的數值_相同_,而是檢查當前的`_version`是否比指定的數值小。如果請求成功，那麼外部的版本號就會被存儲到文檔中的`_version`中。

外部版本號不僅可以在索引和刪除請求時使用，還可以在_創建_時使用。

例如，創建一篇使用外部版本號爲`5`的博文，我們可以這樣操作：


```js
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

在返回結果中，我們可以發現`_version`是`5`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```

現在我們更新這個文檔，並指定`version`爲`10`：

```js
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

請求被成功執行並且`version`也變成了`10`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```
如果你再次執行這個命令，你會得到之前的錯誤提示信息，因爲你所指定的版本號並沒有大於當前Elasticsearch中的版本號。
