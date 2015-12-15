### 安装ES

`cd /tmp`
`wget -N http://192.168.0.55/downloads/elasticsearch-1.6.0.tar.gz`

`tar -zxf elasticsearch-1.6.0.tar.gz`

`mv ./elasticsearch-1.6.0 /usr/local/elasticsearch`

#### elasticsearch-servicewrapper

`cd /tmp`

`git clone https://github.com/elasticsearch/elasticsearch-servicewrapper.git`

`mv ./elasticsearch-servicewrapper/service /user/local/elasticsearch/bin`

#### 启动es

`cd /user/local/elasticsearch/bin/service`

`sh elasticsearch start`

`sh elasticsearch stop`

#### plugin

`cd /usr/local/elasticsearch/bin`

`./plugin -i mobz/elasticsearch-head`

`./plugin -i elasticsearch/marvel/latest`

#### 操作

插入数据

`curl -XPOST localhost:9200/lvmama/mint?pretty -d '{title: "小鸡炖蘑菇", source: "lvmama.com"}'`

`curl -XPOST localhost:9200/lvmama/mint?pretty -d '{title: "番茄炒鸡蛋", source: "ctrip.com"}'`

获取数据

`curl -XGET localhost:9200/lvmama/mint/AVGkq2V53H9cjxGpAgaz?pretty`

更新数据

`curl -XPUT localhost:9200/lvmama/mint/AVGkq2V53H9cjxGpAgaz?pretty -d '{title: "番茄炒菠萝", source: "lvmama.com"}'`

删除数据

`curl -XDELETE localhost:9200/lvmama/mint/AVGkq2V53H9cjxGpAgaz?pretty`

搜索数据

`curl -XGET localhost:9200/lvmama/mint/_search?pretty`

`curl -XGET localhost:9200/lvmama/mint/_search?pretty -d '{"query":{"match":{"source":"lvmama.com"}}}'`

