# 檢查文檔是否存在

如果確實想檢查一下文檔是否存在，你可以試用`HEAD`來替代`GET`方法，這樣就是會返回HTTP頭文件：

```js
curl -i -XHEAD /website/blog/123
```
如果文檔存在，Elasticsearch將會返回`200 OK`的狀態碼：

```js
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
如果不存在將會返回`404 Not Found`狀態碼：

```js
curl -i -XHEAD /website/blog/124
```

```js
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

當然，這個反饋只代表了你查詢的那一刻文檔不存在，但是不代表幾毫秒後它不存在，很可能與此同時，另一個進程正在創建文檔。
