# 與 Elasticsearch 通信

如何與 Elasticsearch 通信要取決於你是否使用 JAVA。

## Java API

如果你使用的是 JAVA，Elasticsearch 內置了兩個客戶端，你可以在你的代碼中使用：

節點客戶端:
    節點客戶端以一個 _無數據節點_ 的身份加入了一個集羣。換句話說，它自身是沒有任何數據的，但是他知道什麼數據在集羣中的哪一個節點上，然後就可以請求轉發到正確的節點上並進行連接。

傳輸客戶端:
    更加輕量的傳輸客戶端可以被用來向遠程集羣發送請求。他並不加入集羣本身，而是把請求轉發到集羣中的節點。

這兩個客戶端都使用 Elasticsearch 的 _傳輸_ 協議，通過**9300端口**與 java 客戶端進行通信。集羣中的各個節點也是通過9300端口進行通信。如果這個端口被禁止了，那麼你的節點們將不能組成一個集羣。

**************************************************
> ###TIP

Java 的客戶端的版本號必須要與 Elasticsearch 節點所用的版本號一樣，不然他們之間可能無法識別。
**************************************************
更多關於 Java API 的說明可以在這裏找到 [Guide](http://www.elasticsearch.org/guide/).


## 通過 HTTP 向 RESTful API 傳送 json

其他的語言可以通過*9200端口*與 Elasticsearch 的 RESTful API 進行通信。事實上，如你所見，你甚至可以使用行命令 `curl` 來與 Elasticsearch 通信。

**************************************************

Elasticsearch 官方提供了很多種編程語言的客戶端，也有和許多社區化軟件的集成插件，這些都可以在 [Guide](http://www.elasticsearch.org/guide/) 裏面找到。

**************************************************

向 Elasticsearch 發出的請求和其他所有的 HTTP 請求的組成部分是一致的。例如，計算集羣中文件的數量，我們就可以使用：

```js
      <1>     <2>                   <3>    <4>
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{  <5>
    "query": {
        "match_all": {}
    }
}
'
```
1. 相應的 HTTP _請求方法_ 或者 _變量_ : `GET`, `POST`, `PUT`, `HEAD` 或者 `DELETE`。
2. 集羣中任意一個節點的訪問協議、主機名以及端口。
3. 請求的路徑。
4. 任意一個查詢後再加上 `?pretty` 就可以生成 _更加美觀_ 的JSON反饋，以增強可讀性。
5. 一個 JSON 編碼的請求主體（如果需要的話）。

Elasticsearch 將會返回一個 HTTP 狀態碼類似於 '200 OK'，以及一個 JSON 格式的主體（除了單純的 'HEAD' 請求），上面的請求會得到下方的 JSON 主體：

```js
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

在反饋中，我們並沒有看見 HTTP 的頭部信息，因爲我們沒有告知 `curl` 顯示這些內容。如果你想看到頭部信息，可以在使用 `curl` 命令的時候再加上 `-i` 這個參數：

```js
curl -i -XGET 'localhost:9200/'
```

從現在開始，本書裏所有涉及 `curl`  命令的部分我們都會進行簡寫，因爲主機、端口等信息都是相同的，縮減前的樣子:

```js
curl -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```

我們將會簡寫成這樣:

```js
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```


