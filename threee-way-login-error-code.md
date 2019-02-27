## Weibo Login OAuth2.0 ##  
| 错误码(error) | 错误编号(error_code) | 错误描述(error_description) |
| --- | ---- | ---- |
|redirect_uri_mismatch|21322|重定向地址不匹配 
|invalid_request|21323|请求不合法 
|invalid_client|21324|client_id或client_secret参数无效
|invalid_grant|21325|提供的Access Grant是无效的、过期的或已撤销的
|unauthorized_client|21326|客户端没有权限
|expired_token|21327|token过期
|unsupported_grant_type|21328|不支持的 GrantType
|unsupported_response_type|21329|不支持的 ResponseType
|access_denied|21330|用户或授权服务器拒绝授予数据访问权限
|temporarily_unavailable|21331|服务暂时无法访问
|appkey permission denied|21337|应用权限不足
## QQ Login OAuth2.0 ## 
| 错误码 | 含义说明|
| --- | ---- |
|0|成功。
|100000|缺少参数response_type或response_type非法。
|100001|缺少参数client_id。100002缺少参数client_secret。
|100003|http head中缺少Authorization。
|100004|缺少参数grant_type或grant_type非法。
|100005|缺少参数code。
|100006|缺少refresh token。
|100007|缺少access token。
|100008|该appid不存在。
|100009|client_secret（即appkey）非法。
|100010|回调地址不合法，常见原因请见：回调地址常见问题及修改方法100011APP不处于上线状态。
|100012|HTTP请求非post方式。
|100013|access token非法。
|100014|access token过期。 token过期时间为3个月。如果存储的access token过期，请重新走登录流程，根据使用Authorization_Code获取Access_Token或使用Implicit_Grant方式获取Access_Token获取新的access token值。
|100015|access token废除。 token被回收，或者被用户删除。请重新走登录流程，根据使用Authorization_Code获取Access_Token或使用Implicit_Grant方式获取Access_Token获取新的access token值。
|100016|access token验证失败。
|100017|获取appid失败。
|100018|获取code值失败。
|100019|用code换取access token值失败。
|100020|code被重复使用。
|100021|获取access token值失败。
|100022|获取refresh token值失败。
|100023|获取app具有的权限列表失败。
|100024|获取某OpenID对某appid的权限列表失败。
|100025|获取全量api信息、全量分组信息。
|100026|设置用户对某app授权api列表失败。
|100027|设置用户对某app授权时间失败。
|100028|缺少参数which。
|100029|错误的http请求。
|100030|用户没有对该api进行授权，或用户在腾讯侧删除了该api的权限。请用户重新走登录、授权流程，对该api进行授权。
|100031|第三方应用没有对该api操作的权限。请发送邮件进行OpenAPI权限申请。
|100032|过载，一开始未细分时可以用。
|100033|缺少UIN参数。
|100034|缺少skey参数
|100035|用户未登陆。
|100036|RefreshToken失效。
|100037|RefreshToken已过期
|100038|RefreshToken已废除
|100039|RefreshToken到达调用上限。
|100040|RefreshToken的AppKey非法。
|100041|RefreshToken,AppID非法。
|100042|RefreshToken非法。
|100043|APP处于暂停状态。
|100044|错误的sign，Md5校验失败，请求签名与官网填写的签名不一致。
|100045|用户改密token失效。
|100046|g_tk校验失败。
|100048|没有设置companyID。
|100049|APPID没有权限(get_unionid)。
|100050|OPENID解密失败，一般是openid和appid不匹配。
|100051|调试模式无权限。
|110401|请求的应用不存在。
|110404|请求参数缺少appid。
|110405|登录请求被限制，请稍后在登录。
|110406|应用没有通过审核。
|110500|获取用户授权信息失败。
|110501|获取应用的授权信息失败
|110502|设置用户授权失败
|110503|获取token失败
|110504|系统内部错误
|110505|参数错误
|110506|获取APP info信息失败
|110506|校验APP info 签名信息失败
|110508|获取code失败
|110509|SKEY校验失败
|110510|Disable
