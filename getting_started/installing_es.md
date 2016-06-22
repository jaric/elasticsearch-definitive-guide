# 安裝 JAVA
```js
yum install java-1.7.0-openjdk -y
```

# 安裝 Elasticsearch

瞭解 Elasticsearch 最簡單的方法就是去盡情的玩兒它（汗），準備好了我們就開始吧。

安裝 Elasticsearch 只有一個要求，就是要安裝最新版本的JAVA。你可以到官方網站下載它：[www.java.com](http://www.java.com).

你可以在這裏下載到最新版本的 Elasticsearch：
[elasticsearch.org/download](http://www.elasticsearch.org/download/).

```js
curl -L -O http://download.elasticsearch.org/PATH/TO/LATEST/$VERSION.zip
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```

提示: 當你安裝 Elasticsearch 時，你可以到 [下載頁面](http://www.elasticsearch.org/downloads) 選擇Debian或者RP安裝包。或者你也可以使用官方提供的 [Puppet module](https://github.com/elasticsearch/puppet-elasticsearch) 或者 [Chef cookbook](https://github.com/elasticsearch/cookbook-elasticsearch).



# 安裝 Marvel
###### 這是個付費的監控插件 暫時先不翻譯
[Marvel](http://www.elasticsearch.com/marvel) is a management and monitoring
tool for Elasticsearch which is free for development use. It comes with an
interactive console called Sense which makes it very easy to talk to
Elasticsearch directly from your browser.

Many of the code examples in this book include a ``View in Sense'' link. When
clicked, it will open up a working example of the code in the Sense console.
You do not have to install Marvel, but it will make this book much more
interactive by allowing you to  experiment with the code samples on your local
Elasticsearch cluster.

Marvel is available as a plugin. To download and install it, run this command
in the Elasticsearch directory:

```js
./bin/plugin -i elasticsearch/marvel/latest
```

You probably don't want Marvel to monitor your local cluster, so you can
disable data collection with this command:

```js
echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml
```

# 運行 Elasticsearch

Elasticsearch 已經蓄勢待發，現在你便可以運行它了：

```js
./bin/elasticsearch
```
如果你想讓它在後臺保持運行的話可以在命令後面再加一個 `-d`

開啓後你就可以使用另一個終端窗口來進行測試了:

```js
curl 'http://localhost:9200/?pretty'
```


你應該看到如下提示：

```js
{
   "status": 200,
   "name": "Shrunken Bones",
   "version": {
      "number": "1.4.0",
      "lucene_version": "4.10"
   },
   "tagline": "You Know, for Search"
}
```

這就說明你的 Elasticsearch _集羣_ 已經上線運行了，這時我們就可以進行各種實驗了。

****
> ###集羣和節點

_節點_ 是 Elasticsearch 運行的實例。_集羣_ 是一組有着同樣`cluster.name`的節點，它們協同工作，互相分享數據，提供了故障轉移和擴展的功能。當然一個節點也可以是一個集羣。

****
