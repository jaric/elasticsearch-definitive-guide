# 統計

最後，我們還有一個需求需要完成：可以讓老闆在職工目錄中進行統計。Elasticsearch 把這項功能稱作 _彙總 (aggregations)_，通過這個功能，我們可以針對你的數據進行復雜的統計。這個功能有些類似於 SQL 中的 `GROUP BY`，但是要比它更加強大。

例如，讓我們找一下員工中最受歡迎的興趣是什麼：

```js
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

請忽略語法，讓我們先來看一下結果：

```js
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```
我們可以發現有兩個員工喜歡音樂，還有一個喜歡森林，還有一個喜歡運動。這些數據並沒有被預先計算好，它們是在文檔被查詢的同時實時計算得出的。如果你想要查詢姓 Smith 的員工的興趣彙總情況，你就可以執行如下查詢：

```js
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```
這樣，`all_interests` 的統計結果就只會包含滿足查詢的文檔了：

```js
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```
彙總還允許多個層面的統計。比如我們還可以統計每一個興趣下的平均年齡：

```js
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```
雖然這次返回的彙總結果變得更加複雜了，但是它依舊很容易理解：

```js
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```
在這個豐富的結果中，我們不但可以看到興趣的統計數據，還能針對不同的興趣來分析喜歡這個興趣的`平均年齡`。

即使你現在還不能很好地理解語法，但是相信你還是能發現，用這個功能來實現如此複雜的統計工作是這樣的簡單。你的極限取決於你存入了什麼樣的數據喲！
