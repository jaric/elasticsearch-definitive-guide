# 面向文檔

程序中的對象很少是單純的鍵值與數值的列表。更多的時候它擁有一個複雜的結構，比如包含了日期、地理位置、對象、數組等。

遲早你會把這些對象存儲在數據庫中。你會試圖將這些豐富而又龐大的數據都放到一個由行與列組成的關係數據庫中，然後你不得不根據每個字段的格式來調整數據，然後每次重建它你都要檢索一遍數據。

Elasticsearch 是 _面向文檔型數據庫_，這意味着它存儲的是整個對象或者 _文檔_，它不但會存儲它們，還會爲他們建立**索引**，這樣你就可以搜索他們了。你可以在 Elasticsearch 中索引、搜索、排序和過濾這些文檔。不需要成行成列的數據。這將會是完全不同的一種面對數據的思考方式，這也是爲什麼 Elasticsearch 可以執行復雜的全文搜索的原因。


# JSON

Elasticsearch使用 [_JSON_](http://baike.baidu.com/view/136475.htm?fr=aladdin) (或稱作JavaScript
Object Notation ) 作爲文檔序列化的格式。JSON 已經被大多數語言支持，也成爲 NoSQL 領域的一個標準格式。它簡單、簡潔、易於閱讀。

把這個 JSON 想象成一個用戶對象:

```js
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "about": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01",
}
```

雖然 `user` 這個對象非常複雜，但是它的結構和含義都被保留到 JSON 中了。在 Elasticsearch 中，將對象轉換爲 JSON 並作爲索引要比在表結構中做相同的事情簡單多了。

***
>###將你的數據轉換爲 JSON

幾乎所有的語言都有將任意數據轉換、機構化成 JSON，或者將對象轉換爲JSON的模塊。查看 `serialization` 以及 `marshalling` 兩個 JSON 模塊。[The official Elasticsearch clients](http://www.elasticsearch.org/guide) 也可以幫你自動結構化 JSON。

***
