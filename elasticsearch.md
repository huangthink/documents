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
