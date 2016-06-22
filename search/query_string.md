# _精簡_ 搜索

搜索的API分爲兩種：其一是通過參數來傳遞查詢的“精簡版”_查詢語句（query string）_，還有一種是通過JSON來傳達豐富的查詢的完整版_請求體（request body）_，這種搜索語言被稱爲查詢DSL。


查詢語句在行命令中運行點對點查詢的時候非常實用。比如我想要查詢所有`tweet`類型中，所有`tweet`字段爲`"elasticsearch"`的文檔：

```js
GET /_all/tweet/_search?q=tweet:elasticsearch
```

下一個查詢是想要尋找`name`字段爲`"john"`且`tweet`字段爲`"mary"`的文檔，實際的查詢就是：

    +name:john +tweet:mary

但是經過_百分號編碼（percent encoding）_處理後，會讓它看起來稍顯神祕:

```js
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```
前綴`"+"`表示**必須要**滿足我們的查詢匹配條件，而前綴`"-"`則表示**絕對不能**匹配條件。沒有`+`或者`-`的表示可選條件。匹配的越多，文檔的相關性就越大。



### 字段`_all`

下面這條簡單的搜索將會返回所有包含`"mary"`字符的文檔：

```js
GET /_search?q=mary
```

在之前的例子中，我們搜索`tweet`或者`name`中的文字。然而，搜索的結果顯示`"mary"`在三個不同的字段中：

* 用戶的名字爲"Mary"
* 6個"Mary"發送的推文
* 1個"@mary"

那麼Elasticsearch是如何找到三個不同字段中的內容呢？

當我們在索引一個文檔的時候，Elasticsearch會將所有字段的數值都彙總到一個大的字符串中，並將它索引成一個特殊的字段`_all`：

```js
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```
就好像我們已經添加了一個叫做`_all`的字段：

```js
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```
除非指定了字段名，不然查詢語句就會搜索字段`_all`。

TIP: 在你剛開始創建程序的時候你可能會經常使用`_all`這個字段。但是慢慢的，你可能就會在請求中指定字段。當字段`_all`已經沒有使用價值的時候，那就可以將它關掉。之後的《字段all》一節中將會有介紹


## 更加複雜的查詢

再實現一個查詢：

* 字段`name`包含`"mary"`或`"john"`
* `date`大於`2014-09-10`
* `_all`字段中包含`"aggregations"`或`"geo"`

```js
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```

最終處理完的語句可讀性可能很差：

```
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```
正如你所看到的，這個_簡明_查詢語句是出奇的強大。在[查詢語句語法](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current//query-dsl-query-string-query.html#query-string-syntax)中，有關於它詳細的介紹。藉助它我們就可以在開發的時候提高很多效率。

不過，你也會發現簡潔帶來的易讀性差和難以調試，以及它的脆弱：當其中出現`-`, `:`, `/` 或者 `"`時，它就會返回錯誤提示。

最後要提一句，任何用戶都可以通過查詢語句來訪問臃腫的查詢，或許會得到一些私人的信息，或許會通過大量的運算將你的集羣壓垮！


****
> ### TIP

出於以上原因，我們不建議你將查詢語句直接暴露給用戶，除非是你信任的可以訪問數據與集羣的權限用戶。

****

與此同時，在生產環境中，我們經常會使用到查詢語句。在瞭解更多關於搜索的知識前，我們先來看一下它是怎樣運作的。

