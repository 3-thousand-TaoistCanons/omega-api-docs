![logo.png](https://github.com/Dataman-Cloud/omega-api-docs/blob/master/apiv3/logo-shurenyun.png)

# 数人云RESTful API文档规范(v0.0.1)

## CHANGES
- [x] 文档初次提交（cmxu 2016-2-5)

- [x] 修改了版本约定，HTTP STATUS CODE，返回结果结构（cmxu 2016-2-22)

- [x] 规范HTTP STATUS CODE的使用（Zheng Liu 2016-6-28)

## 一，文档目的

**为了给数人云客户（开发者）提供简单通俗，容易理解的数人云API文档， 特提出此规范，
目的是约束大家API开发过程中使用统一的命名规范，
命名原则，参数命名规范，HTTP headers使用， 返回结果集，
出错错误码等内容。**

### API样例

	GET api.shurenyun.com/v1/clusters?page=1&per_page=20

	PUT api.shurenyun.com/v1/cluster/:cluster_id

	DELETE api.shurenyun.com/v1/cluster/:cluster_id

## 二，命名规范

### 1，API HOST地址

	api.shurenyun.com/

* 数人云API 使用二级域名访问， 方便后期维护和负载均衡。
* API文档由swagger自动生成和部署，各位开发同事维护自己熟悉部分API， 由运维同事统一生成和部署。
* 数人云schema用https还是http需要商量之后决定。

### 2，版本说明

	api.shurenyun.com/v1/
	api.shurenyun.com/v2/

* 数人云API版本信息由产品同事配合决定，重要版本发布应该集体讨论，合适场合发布给客户。

### 3，数人云API响应的 HTTP Method
数人云API响应的HTTP包括 GET,POST,PUT,PATCH,DELETE 五种，其中

* GET 标识资源获取和列表
* POST 新生成资源
* PUT 对原有资源的整体更新
* PATCH 对原有资源部分修改， 比如状态修改
* DELETE 删除资源，包括隐藏资源

### 4，URL以及命名

* URL path部分完全符合RESTful原则，资源名称尽量简洁恰当，资源命名前应集体商讨决定，避免出现生僻单词，避免不够native的命名。
* 资源名称应该为一个单词的名词。
* 对资源的操作比如（deactive， activate）应为动词， 如果需要两个单词才能表达的动作应为两个单词以下划线分隔，比如get_status而非getStatus。
* URL的字符集应以小写字母， 数字，?, _, &, /组合而成， 不应出现大写字母以及其他特殊字符，下面列出误例几则：

	- api.shurenyun.com/v1/CLUSTERS     (不能出现大写)
	- api.shurenyun.com/v1/clusters/clusterid/getStatus (用get_status而非getStatus)
	- api.shurenyun.com/v1/clusters?page=1&perPage=20 (per_page而非perPage)
	- api.shurenyun.com/v1/clusters/?page=1&per_page=20 (cluster后面不应出现**/**)
        - api.shurenyun.com/v1/clusters/{cluster_id}/get_status(正确示例)

### 5，参数命名
API请求的参数可能有三个来源，Header, Query和Body中。其中GET请求的参数出现在Query中， PUT和POST请求参数出现在BODY当中，HEADER中参数对应AUTH等信息。


## 三，返回结果
数人云返回结果由两部分组成， 第一部分为HTTP status code， 第二部分为response body本身。所有API的状态返回应符合常规的HTTP status code的定义。
而详细的自定义的错误状态码, 应采用JSON格式包含在response body中。response body本身也为json格式。

### HTTP STATUS CODE

* 200 OK 正常返回，场景例如：GET请求列表正常返回,删除资源成功等
* 201 Created 资源创建成功，服务器接受POST请求，并成功创建资源时返回。
* 400 Bad Request 用在客户端请求错误上，如非法参数，数据错误等,此时需返回状态码400，同时将详细错误信息码和错误信息包含在response body中。
* 401 Unauthorized 场景例如:访问需要授权的资源，比如未登录情况下访问cluster，Token过期，Token不正确
* 403 Forbidden 没有权限进行此次操作。
* 404 Not Found. 多用于GET, DELETE请求返回。
* 409 Conflict 要创建的资源已经存在，需要客户端更新请求的数据。
* 50x 多为Nginx提供，当server端发生未知错误时使用。

更详细的HTTP STATUS使用规范请参考 [这个地址](https://github.com/kubernetes/kubernetes/blob/release-1.2/docs/devel/api-conventions.md#http-status-codes)


### RESPONSE BODY

 * 返回结果本身为application/json格式，
 * json reponse的key如果是多个单词组成应该是camelCase，例如
   responseBody而不是response_body。
 * 返回结果的key不应该是单词缩写，应该是完整的清晰的单词。例如
  response而不是resp。
 * 返回结果分code和data两个部分，
   code表明结果是否符合预期，data包含返回数据
 * 所有返回结果中都应该有code表明请求是否成功，code值有0和非0的数字组成
 * data可以为一个object或者数组

成功结果返回

	{
		"code": 0
		"data": ...
	}

失败返回结果

	{
		"code": 10002
		"data": {
			"message": "token not valid",
      ...
		}

	}

<b>请注意</b>：当API源码有所更改时，API doc上也需要做相应的更新。

## 四，错误码规约
错误码   |  描述   |  备注
------- | ------ | ------
0       |  正常
10000   |  未知错误
10001   |  数据错误 | data为对象类型，key为错误字段，value为错误信息，如果{"name": "集群名重复"}
10002   |  token验证失败
10003   |  数据库操作错误
10008   |  JSON格式错误
10009   |  资源不存在
10010   |  没有权限进行此操作
10011   |  请求参数per_page出错
10012   |  请求参数page出错
10013   |  请求参数sort_by出错
10014   |  请求参数order出错
10015   |  操作不被允许
11005   |  非激活用户
11006   |  用户已加入用户组
11007   |  该用户组有其它用户存在或有集群存在
11008   |  没有验证手机号
11009   |  已发送，请等待
11010   |  验证码错误
11011   |  授权错误
13001   |  Job不存在
13002   |  query_by条件不支持
14001   |  Application不存在
14002   |  application历史版本id不存在
14003   |  应用名称冲突
14004   |  应用端口冲突
14005   |  应用版本冲突
14006   |  应用被锁定
14007   |  应用扩展失败
14008   |  应用更新已完成，回滚失败
14009   |  环境变量命名错误
14010   |  非法的clusterId
14011   |  请求错误
14012   |  非法的appId
14013   |  应用组件不可用
14014   |  CheckPermissionError
14015   |  系统保留端口
15001   |  数据库操作错误
15002   |  json 操作错误
15003   |  读取header错误
15004   |  获取路径参数错误
15005   |  获取body中的参数错误
15006   |  类型转换错误
15007   |  构建Entry错误
15008   |  project不存在
15009   |  image不存在
15010   |  drone激活错误
15011   |  获取public key 错误
15012   |  hook错误
15013   |  解析url参数错误
15014   |  获取build错误 | app连接drone报错, 查看app能否连接drone的端口
15015   |  获取log错误
15016   |  向harbor注册错误
15017   |  获取最新build错误
15018   |  更新drone失败
15019   |  删除drone错误
15020   |  Build Stream不存在
15022   |  drone获取代码失败
15023   |  yaml文件不存在或获取失败
15024   |  项目名称冲突
15025   |  镜像名称冲突
16000   |  repo配置文件没找到
16001   |  编排应用部署有问题
16002   |  编排服务名称冲突
16003   |  编排应用删除出错
16004   |  问题模板解析错误
16005   |  Docker Compose解析错误
16006   |  Marathon Config解析错误
16007   |  当前状态应用不能更新
16008   |  MarathonConfig和DockerCompose不匹配
17001   |  metric请求参数错误
18001   |  告警策略参数解析失败
18010   |  删除告警策略失败
18011   |  查询告警策略失败
18012   |  查询告警策略失败
18020   |  告警事件参数解析失败
18021   |  查询告警事件失败


## 五，SWAGGER使用
[参考SWAGGER](./README.md)

## 六，其他
