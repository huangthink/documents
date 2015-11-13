# 搜索逻辑

## 门票

### 门票广告位

搜索广告位产品，如果是第一页，搜索产品对应的商品，否则需统计广告位产品的数量。

#### 门票广告位产品搜索逻辑

1. 根据目的地信息获取adProductId、adProductSeq
2. 构建Query:广告产品ID,如果渠道是驴途要构建渠道,必须要有效可售的价格大于0才能查出来
3. seq升序排序

##### 门票广告位搜索相关接口

1. 搜索广告位产品的接口：`PageConfig<TicketIndexBean> searchAd(VstTicketSearchVO vstTicketSearchVO, DestIndexBean destIndexBean, int size);`


### 门票产品

#### 门票产品搜索逻辑

允许查询无渠道的产品，如果是主站搜索，主产品不按渠道搜索，而商品和打包商品产品继续按渠道搜

1. 通过经纬度搜索
2. 通过ID搜索（目前对于多个ID是或的关系）
3. 主题或标签（是否完全匹配）
4. 行政区（单个关键字）
5. 通过主题、标签、目的地、名称搜索
6. 通过游玩景点及活动主题搜索
7. 相关推荐搜索 （只针对以上所有都无结果或只有5有结果）

##### 门票产品搜索逻辑相关接口

1. `QueryResponse search(VstTicketSearchVO vo, String excludeIds, SORT... sorts);`
2. `QueryResponse relatedSearch(VstTicketSearchVO vo, String excludeIds, SORT... sorts);`

#### 门票搜索结果集处理

1. 如果有相关性搜索，执行 `ticketProductSearchService.relatedSearch(vo, excludeIds, sorts);`,并去重；
2. 过滤产品结果集和相关性搜索的结果集
3. 如果包含地标，按地标排序
4. 起价为空或0的产品和产品渠道不包含用户渠道的产品放到最后

### 门票商品

#### 门票商品搜索逻辑

`resultMap.put("product", ticketIndexBean);`

1. 必须当前产品为可售的情况下才查询他下面的单门票和其他票`resultMap.put("goods", goodsTableBeans);// 景点门票`
2. 供应商打包必须在产品品类是11(景区门票)的时候才查询`resultMap.put("suppgoods", suppGoodsTableBeans);// 供应商打包`
3. 自主打包必须在产品品类是11(景区门票)的时候才查询`resultMap.put("lvmamaProduct", listTicketIndexBeans);// 自主打包`

##### 门票商品搜索相关接口

1. 搜索商品的接口：`PageConfig<Map<String, Object>> searchGoods(PageConfig<TicketIndexBean> pageConfig, VstSearchVO sv);`

## 线路

### 线路广告位

线路广告位分为ALL和非ALL（route,local,freetour,around,group）

#### 线路广告位产品搜索逻辑

**ALL**:

1. 根据目的地信息获取adProductId、adProductSeq、adType
2. 构建Query:广告产品ID,如果渠道是驴途要构建渠道,必须要有效可售的价格大于0才能查出来
3. seq升序排序

**非ALL**:

1. 根据目的地信息获取adProductId、adProductSeq
2. 构建Query:广告产品ID,如果渠道是驴途要构建渠道,必须要有效可售的价格大于0才能查出来
3. seq升序排序

##### 线路广告位搜索相关接口

1. 搜索广告位产品的接口：`List<RouteIndexBean> searchAd(VstRouteSearchVO vstRouteSearchVO, DestIndexBean destIndexBean, String type, int size);`
2. 搜索广告位ALL产品的接口：`List<Map.Entry<String, Object[]>> searchAllAd(VstRouteSearchVO vstRouteSearchVO, DestIndexBean destIndexBean, String type, int size);`

### 线路产品

#### 线路产品搜索逻辑

允许查询无渠道的产品，如果是主站搜索，主产品不按渠道搜索，而商品和打包商品产品继续按渠道搜

1. 通过ID搜索（目前对于多个ID是或的关系）
2. 目的地跟团游(LOCAL)只搜索出发地

> 根据目的地类型获取对应的Query，目的地跟团游专用<br />
> 省和国家匹配对应的所属层级，其余默认匹配当前出发地（CITY）
	 
3. 主题或标签（是否完全匹配）
4. 行政区（单个关键字）
5. 通过主题、标签、目的地、名称搜索
7. 相关推荐搜索 （只针对以上所有都无结果或只有5有结果）

##### 线路产品搜索逻辑相关接口

1. `QueryResponse search(VstTicketSearchVO vo, String excludeIds, SORT... sorts);`
2. `QueryResponse relatedSearch(VstTicketSearchVO vo, String excludeIds, SORT... sorts);`

#### 线路搜索结果集处理

1. 如果有相关性搜索，执行 `ticketProductSearchService.relatedSearch(vo, excludeIds, sorts);`,并去重；
2. 过滤产品结果集和相关性搜索的结果集
3. 多出发地：对第一出发地排序块增加出发地排序影响因子，最终计算总分值进行排序（产品总分值 = 十日销量比 *5000 + （用户选择的）出发地 *3500 + 标签 *1500 + 主题 *500）；针对第一页设置自由行与跟团游的展示个数比例，从而保证第一页的自由行与跟团游产品都可以露出
4. 按产品总分值排序
5. 起价为空或0的产品和产品渠道不包含用户渠道的产品放到最后


# 搜索缓存

## 缓存对象

- 广告位产品ID集合（）
- 搜索结果产品ID集合
- 筛选结果

CacheConstant
CacheAnnotation
CacheAspect
CacheService
ThreadContext

为了解决产品ID集合和筛选结果保持一致，同时失效，将两个放入同一对象SearchCache：cacheKey，ids，selectMap，codeMap；

## 搜索缓存流程

#### 广告位缓存

1. 在需要缓存的方法上加上CacheAnnotation
2. 进入切面CacheAspect
3. 判断是否开启缓存
4. 根据prefixKey调用CacheService不同的方法
5. 拼接cacheKey，从redis读取缓存
6. 如果为空，调用原生方法，放入redis
7. 如果不为空，直接从缓存取

ticketAD:List<String> ids，routeAD:List<RouteIndexBean> routeList

#### 产品缓存

1. 在需要缓存的方法上加上CacheAnnotation
2. 进入切面CacheAspect
3. 判断是否开启缓存
4. 根据prefixKey调用CacheService不同的方法
5. 拼接cacheKey，从redis读取缓存
6. 如果为空，调用原生方法，将cacheKey，ids放入SearchCache，再放入ThreadContext
7. 如果不为空，取出当前页的id搜索产品，再把产品为空或0的放到后面
8. 在FilterSelector中通过ThreadContext.getSearchCache() 获取SearchCache对象
9. 如果searchCache不为空并且searchCache.getSelectMap()不为空，直接从缓存取
10. 否则searchCache为空，重新计算筛选结果，如果searchCache不为空，将筛选结果放入searchCache，再将searchCache放入redis
11. 从ThreadContext移除search_cache

**门票cacheKey**

	StringBuilder cacheKey = new StringBuilder();
	cacheKey.append(TICKET_KEY_PRIFIX);
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(vo.getKeyword());
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(vo.getDistributors());

**线路cacheKey**

	StringBuilder cacheKey = new StringBuilder();
	cacheKey.append(ROUTE_AD_KEY_PRIFIX);
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(vo.getKeyword());
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(type);
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(vo.getFromDestId() == null ? "0" : vo.getFromDestId());
	cacheKey.append(CACHE_KEY_SEPARATE);
	cacheKey.append(vo.getDistributors());
