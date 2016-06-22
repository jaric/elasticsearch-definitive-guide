# 映射與統計

當我們在進行搜索的事情，我們會發現有一些奇怪的事情。比如有一些內容似乎是被打破了：在我們的索引中有12條推文，中有一個包含了`2014-09-15`這個日期，但是看看下面的查詢結果中的總數量：

``` js
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```
爲什麼我們使用字段`_all`搜索全年就會返回所有推文，而使用字段`date`搜索年份卻沒有結果呢？爲什麼使用兩者所得到的結果是不同的？

推測大概是因爲我們的數據在`_all`和`date`在索引時沒有被相同處理。我們來看看Elasticsearch是如何處理我們的文檔結構的。我們可以對`gb`的`tweet`使用_mapping_請求：

```js
GET /gb/_mapping/tweet
```
我們得到：

```js
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```
Elasticsearch會根據系統自動判斷字段類型並生成一個映射。返回結果告訴我們`date`字段被識別成了`date`類型。`_all`沒有出現是因爲他是默認字段，但是我們知道字段`_all`實際上是`string`類型的。

所以類型爲`date`的字段和類型爲`string`的字段的索引方式是不同的。

So fields of type `date` and fields of type `string` are indexed differently,
and can thus be searched differently.  That's not entirely surprising.
You might expect that each of the core data types -- strings, numbers, booleans
and dates -- might be indexed slightly differently. And this is true:
there are slight differences.

But by far the biggest difference is actually between fields that represent
_exact values_ (which can include `string` fields) and fields that
represent _full text_. This distinction is really important -- it's the thing
that separates a search engine from all other databases.

