## 分佈式文檔存儲

#### 本章將會在主要章節翻譯結束後再繼續翻譯

In the last chapter, we looked at all the ways to put data into your index and
then retrieve it.  But we glossed over many technical details surrounding how
the data is distributed and fetched from the cluster.  This separation is done
on purpose -- you don't really need to know how data is distributed to work
with Elasticsearch.  It just works.

In this chapter, we are going to dive into those internal, technical details
to help you understand how your data is stored in a distributed system.


****
> ### 內容警告

The information presented below is for your interest. You are not required to
understand and remember all the detail in order to use Elasticsearch. The
options that are discussed are for advanced users only.

Read the section to gain a taste for how things work, and to know where the
information is in case you need to refer to it in the future, but don't be
overwhelmed by the detail.

****
