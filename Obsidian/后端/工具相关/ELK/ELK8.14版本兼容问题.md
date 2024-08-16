## 1.报错# <Response stream not captured or already read to completion by serializer. Set DisableDirectStreaming() on ConnectionSettings to force it to be set on the response.> Exception: Elasticsearch.Net.ElasticsearchClientException: Request failed to execute. Call: Status code 400 from: POST /_bulk. ServerError: Type: illegal_argument_exception Reason: "Action/metadata line [1] contains an unknown parameter [_type]"

因为ES 8.x版本强制不能使用_type参数了，所以配置文件一定要明显的加上documentType=""。如果要账号密码的情况，要如下加上校验：
username="elastic" password="z-iiSyebBb2ppQmH-4vJ" requireAuth="true"

