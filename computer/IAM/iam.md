# 使用keycloak服务授权
## 授权中的术语与概念
### Resource Server / 资源服务
存储受保护资源的服务，能够接受对这些受保护资源的请求并响应。
这些服务需要一类信息，用于判断是否接受对受保护资源的访问。基于RESTFul的资源服务通常通过bearer令牌携带这类信息，web应用则通常把信息存储在session中。
在keycloak中，*confidential*类型的注册服务(`client application`)都可以作为资源服务。这些注册服务以及他们自己的资源都通过一系列的授权策略保护。

### Resource / 资源
资源是应用或组织的资产的一部分。资源可以是端点，一类网络资源，比如HTML页面等等。在授权策略中，所有的资源都被抽象成对象。
资源对象可以指代单一的资源或一组资源。比如我们既可以把所有的银行账户定义成一种资源，并给这种资源设置一组统一的授权策略；也可以把小李的账户设置为另一种资源，这种资源独属于一位用户，并给这种资源设置一组独立的授权策略。

### Scope / 作用域
资源的作用域是指对资源执可能执行的的操作。比如`view`、`edit`、`delete`等。

### Permission / 权限
权限通过一组策略和受保护资源相关联，策略定义了对资源的访问是否可以通过。

### Policy / 策略
策略定义了访问一种对象时必须满足的条件。定义策略时不需要指定对象，只需要定义访问对象时要满足的条件。
通过策略，我们可以实现基于属性的访问机制、基于角色的访问机制或基于上下文的访问机制以及其他授权机制。
keycloak中策略和聚合策略的概念，是我们可以定义策略组的策略，通过分治技术，定义一组独立的策略，再把他们复用于不同的权限，这样可以通过组合不同的策略构造更复杂的策略。

### Policy Provider / 策略提供程序
策略提供程序是指特定策略类型的实现。Keycloak提供了服务接口，我们可以使用该接口添加自定义的策略提供程序。

### Permission Ticket / 权限票据
权限票据是一种token，在UMA(User-Managed Access / 用户管理授权)规范中定义。权限票据提供一种不透明的数据结构，其中包含了客户端请求的资源和作用域、请求上下文以及必须应用于请求的策略。
权限票据在用户向用户分享资源或用户向组织分享资源时非常重要。
在UMA流程中，授权服务发布授权票据给资源服务，资源服务把授权票据发给尝试访问受保护资源的客户端，客户端收到票据后把票据发给授权服务请求RPT（请求方令牌）。

## 资源服务管理
### 资源服务设置项
#### 策略执行模式
* Enforcing
默认的执行模式，如果请求的资源没有关联的策略，则拒绝请求。
* Permissive
如果请求的资源没有相关联策略，则接受请求。
* Disable
取消所有的策略匹配，允许访问所有的资源。

#### 授权模式
授权策略评估引擎会根据授权模式对所有的权限评估，并根据所有的评估结果决定是否授予对请求资源和作用域的访问。
* Affirmative
只要任一权限满足则接受访问
* Unanimous
所有的权限都满足才可以接受访问

#### 远程资源管理
如果设置为false，则只能通过管理控制台设置资源

### 默认配置
keycloak会在用户创建资源服务时，创建默认授权配置，这些配置包括：
* 一条默认的受保护资源，这条资源代表应用中的全部资源
* 一条策略，使用这条策略会授予所有相关资源访问权限
* 一条权限，这条权限把默认策略和所有资源相关联

默认资源的类型定义为`urn:{clientID}:resources:default`，URI定义为`/*`。这个URI使用通配符，代表应用的所有路径。所以允许鉴权时，所有的访问都必须经过鉴权。在创建和默认资源相关的权限时需要使用类型值。

## 资源与作用域管理
### 类型/ Type
type是一条唯一的字符串，代表一种或多种资源。使用type可以把资源分组，并次啊用相同的策略保护这些资源。

### 资源拥有者
默认情况下，资源的拥有者是应用，但是也可以设置为用户，这样可以指定基于用户的策略，
比如，只有用户可以删除或更新指定的资源。

### 远程资源管理
资源服务可以通过API远程管理资源

## 管理策略
### 基于用户的策略
此策略用于一组用户可以访问某对象的情况。
#### 配置
* 名称
* 描述
* 用户，指定可以使用此策略授权的用户
* 逻辑，授权或拒绝

### 基于角色的策略
此策略用于一组用户可以访问某对象的情况。默认情况下，添加到策略中的角色不是必须全部满足的，如用用户满足其中任一角色，即可授予权限。但是可以设置某些角色为必须满足的。
在需要严格的基于角色授权的场景下，基于角色策略很有用，这时必须满足指定的角色才能访问特定的资源。比如，你可以强制要求用户必须同意客户端应用访问他们的资源。
#### 配置
* 名称
* 描述
* 域角色
* 应用角色，使用应用角色前必须先选择角色
* 授权逻辑

#### 必须角色
如果指定了必须角色，那么只有用户的角色满足所有的必须角色时，请求才会被授权。

### 基于JS的角色

### 基于时间的角色
#### 配置
* 名称
* 描述
* Not Before
在这个字段定义的时间之前访问不会被授权
* Not On of After
在这个字段定义的时间之后，访问不会被授权
* Day of Month
* Month
* Year
* Hour
* Minute
* Logic

只有所有的条件都满足时，请求才会被授权。

### 聚合策略
keycloak中可以定义策略的策略，即把策略聚合使用，这样可以服用定义好的策略创造更复杂的策略，实现权限和策略的解耦。
在创建聚合策略时需要选择授权模式。除了上文的`Unanimous`、`Affirmative`外，还包括`Consensus`，当通过的策略数高于未通过的策略数时，授予权限。

### 基于客户应用的策略
允许一组客户端访问对象。
#### 配置
* 名称
* 描述
* 客户应用
* 逻辑

### 基于群组的策略
决定是否对某一群组的用户授权。
#### 配置
* 名称
* 描述
* 群组申明： 通常授权是基于令牌，通过此配置项指定令牌中的哪一字段包含群组名称。
* 群组
* 逻辑

#### 向子群组拓展访问权限
默认情况下，权限仅对严格属于选定的群组有效。但是可以延申给子群组。

### 应用作用域策略

### 基于正则表达式的授权策略

## 权限管理
权限把受保护的资源和必须满足的策略关联起来。
权限可以保护两种对象：
* 资源
* 作用域

### 基于资源的权限
基于资源的权限使用一组授权策略保护一组选定的资源。
#### 配置
* 名称
* 描述
* 应用的资源类型，定义这个字段后，所有匹配这一类型的资源都被保护
* 资源
* 应用策略
* 决策模式
  * Unanimous：必须所有的策略都满足才授权
  * Affirmative：任一策略满足即授权
  * Consensus：满足授权的策略数量大于不满足授权的策略数量就授权
#### 分类资源权限
资源权限可以应用给有相同类型的资源。应用中的资源可以根据他们涉及的数据或提供的功能分类。比如财务应用可以管理不同的账户，每个账户都属于特定的用户。这些不同的账户都有相同的安全需求。使用分类资源权限可以仅让账户拥有者可以管理账户；只允许账户拥有着访问账户；实行特定的认证方法。

### 基于作用域的权限
基于资源的权限使用一组授权策略保护一组选定的作用域。使用基于作用域的权限，既可以选择资源也可以选择作用域。
#### 配置
* 名称
* 描述
* 资源
* 作用域
* 应用策略
* 决策模式
  * Unanimous：必须所有的策略都满足才授权
  * Affirmative：任一策略满足即授权
  * Consensus：满足授权的策略数量大于不满足授权的策略数量就授

## 服务鉴权
Keycloak的鉴权服务基于OAuth2和UMA规范。
OAuth2客户端，比如前端应用，通过token端点从服务获取访问令牌，再使用访问令牌向资源服务，比如后端服务，获取资源。Keycloak以同样的流程拓展了OAuth2，根据被请求的资源和作用域找出相关联的的策略，再根据策略的授权评估结果发布访问令牌。资源服务器可以基于服务授予的权限和令牌中包含的权限决定是否接受对资源的访问。
在Keycloak中，携带权限的访问令牌称为请求方令牌，简称为RPT。
除了发布RPT，KeyCloak授权服务还提供了一组RESTful端点，允许资源服务器管理其受保护的资源、作用域、权限和策略，帮助开发人员将这些功能扩展或集成到其应用程序中，以支持细粒度授权。

### 授权服务端点与元信息发现
客户端可以通过Keycloak提供的发现端点，获取和Keycloak授权服务交互需要的所有必要信息，包括端点地址以及功能。
发现文档可以通过：
```bash
curl -X GET http://${host}:${port}/realms/${realm}/.well-known/uma2-configuration
```
可以收到如下响应：
```json
{
    // some claims are expected here

    // these are the main claims in the discovery document about Authorization Services endpoints location
    "token_endpoint"："http://${host}:${port}/realms/${realm}/protocol/openid-connect/token",
    "token_introspection_endpoint"："http://${host}:${port}/realms/${realm}/protocol/openid-connect/token/introspect",
    "resource_registration_endpoint"："http://${host}:${port}/realms/${realm}/authz/protection/resource_set",
    "permission_endpoint"："http://${host}:${port}/realms/${realm}/authz/protection/permission",
    "policy_endpoint"："http://${host}:${port}/realms/${realm}/authz/protection/uma-policy"
}
```
不同的端点提供不同的功能
* token_endpoint：符合OAuth2的令牌端点，支持`urn:ietf:params:oauth:grant-type:uma`票证授予类型。通过这个端点，客户端可以发送授权请求，并获得一个RPT，该RPT包含Keycloak授予的所有权限。
* token_introspection_endpoint：符合OAuth2的令牌检查端点，客户端可以使用该端点检查服务器，以确定RPT的活动状态，并确定与令牌相关的任何其他信息，例如KeyCloak授予的权限。
* resource_registration_endpoint：符合UMA的资源注册端点，资源服务器可以通过这个端点管理资源和作用域。包括创建、查询、更新和删除。
* permission_endpoint：符合UMA的权限端点，资源服务器可以通过此端点管理授权票据。此端点提供权限的创建、查询、更新和删除操作。

### 获取授权
要从keycloak获取授权需要向令牌端点发送授权请求。keycloak会根据请求的资源和作用域评估所有关联的授权策略，然后授予一个含有权限的RPT。
客户端可以使用以下参数请求授权：
* grant_type：必须参数。形如`urn:ietf:params:oauth:grant-type:uma-ticket`
* ticket：可选参数。客户端收到的最新的授权票据。
* claim_token：可选参数。字符串类型，表示服务端做授权判断时需要额外考虑的其他声明。客户端可以通过此参数把生命推送给服务端。有关支持的令牌格式，参考`claim_token_format`参数。
* claim_token_format：可选参数。字符串类型，指明`claim_token`参数所用的令牌格式。keycloak支持两种令牌格式：`urn:ietf:params:oauth:token-type:jwt`和`https://openid.net/specs/openid-connect-core-1_0.html#Token`。`urn:ietf:params:oauth:token-type:jwt`表示`claim_token`中使用的使`access_token`，`https://openid.net/specs/openid-connect-core-1_0.html#Token`表示`claim_token`使用的是`OIDC token`。
* rpt：可选参数。以前颁发的RPT，其中的权限应该被评估并添加到新颁发的令牌中。已经获得RPT的客户端可以使用这个参数发起增量授权请求，按需求获取额外的授权。
* permission：可选参数。字符串类型。代表客户端请求的自组权限，每个权限表示一个或多个资源与作用域。这个参数是对`urn:ietf:params:oauth:grant-type:uma-ticket`授权类型的拓展，客户端使用此参数可以没有权限票据。字符串的格式必须是：`RESOURCE_ID#SCOPE_ID`。比如`Resource A#Scope A`, `Resource A#Scope A`, `Scope B`, `Scope C`, `Resource A`, `#Scope A`。
* audience：可选参数。标记当前客户端试图访问的资源服务器的在keycloak中的客户端id。如果定义了`permission`参数，那么必须定义此参数，用于告知keycloak评估授权的上下文。
* response_include_resource_name：可选参数。布尔类型，告诉服务器是否需要把资源的名称也包含在RPT的权限中。如果是false，那么只会包含资源的标识符。
* response_permission_limit：可选参数。整数类型，定义RPT中可以包含的资源总量。当和`RPT`参数一起使用时，只用最后N个请求的权限会被包含在RPT中。
* submit_request：可选参数。布尔类型，告诉服务器是否需要根据权限票据关联的资源和作用域，创建权限请求。仅在和`ticket`参数一起使用时生效。
* response_mode：可选参数。字符类型。指明服务器如何响应授权请求。当你关注服务器的授权决定而不标准OAuth2响应时，这个参数很有用。可选值包括：
  * decision
  告知服务器，用JSON格式返回完整的授权决策：
  ```json
  {
      "result": true
  }
  ```
  如果授权请求和权限不匹配，则会返回 403状态码。
  * permissions 
  告知服务器用JSON格式返回授予的权限
  ```json
  [
    {
        "rsid": "My Resource"
        "scopes": ["view", "update"]
    },

    ...
  ]
  ```
  如果授权请求和权限不匹配，则会返回 403状态码。

下面展示客户端请求两种资源的授权：
```bash
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  --data "audience={resource_server_client_id}" \
  --data "permission=Resource A#Scope A" \
  --data "permission=Resource B#Scope B"
```
下例展示客户端请求资源服务器的所有资源和作用域
```bash
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  --data "audience={resource_server_client_id}"
```
下例展示客户端获取授权票据后，请求受UMA保护的资源：
```bash
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  --data "ticket=${permission_ticket}
```
如果keycloak的鉴权结果是授予权限，那么会返回含有权限的RPT。
keycloak向客户端响应RPT：
```json
HTTP/1.1 200 OK
Content-Type: application/json
...
{
    "access_token": "${rpt}",
}
```
授权响应和使用其他grant type返回token端点的响应格式是一样的。RPT通过access token字段获取。如果客户端没有获得授权，那么keyclak会响应403状态码。
```bash
HTTP/1.1 403 Forbidden
Content-Type: application/json
...
{
    "error": "access_denied",
    "error_description": "request_denied"
}
```
#### 客户端认证方法
客户端需要向令牌端点认证才能获取RPT。当使用`urn:ietf:params:oauth:grant-type:uma-ticket`授权类型时，客户端可以使用下面的认证方法：
* Bearer Token
  客户端在http请求头的`Authorization`字段使用`Bearer`凭证携带access token。例
  ```bash
  curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket"
  ```
  但客户端代表用户时使用此方法比较合适。这时bearer令牌就是在请求之前keycloak向用户代理授予的accesss token。授权会给予access token锁关联的请求上下文进行。比如access token是向代表用户A的应用A授予的，那么基于用户A可以访问的权限和作用域进行授权决策。
* Client Credentials
  客户端可以使用keycloak支持的任意客户端认证方法，比如client_id和client_secret。例：
  ```bash
  curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Basic cGhvdGg6L7Jl13RmfWgtkk==pOnNlY3JldA==" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket"
  ```

#### 推送声明
从服务器获取权限时，您可以推送任意声明，以便在评估权限时，您的策略可以使用这些声明。
如果获取授权时没有使用权限票据，即没有使用UMA流程，那么可以发送一个授权请求给令牌端点：
```bash
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  --data "claim_token=ewogICAib3JnYW5pemF0aW9uIjogWyJhY21lIl0KfQ==" \
  --data "claim_token_format=urn:ietf:params:oauth:token-type:jwt" \
  --data "client_id={resource_server_client_id}" \
  --data "client_secret={resource_server_client_secret}" \
  --data "audience={resource_server_client_id}"
```
其中`claim_token`参数是BASE64编码的JSON字符串，JSON的格式如下：
```json
{
  "organization": ["acme"]
}
```

### User-Managed Access
Keycloak授权服务基于UMA，UMA规范通过以下方面加强了Oauth2的功能：
* Privacy
* Party-to-Party Authorization
  资源拥有者，通常是终端用户，可以管理对他们资源的访问，并且授权第三方，通常也是终端用户，访问这些资源。 Ouath2把访问权限授予代表用户的客户端应用，而UMA可以使用异步的方式把权限授予其他用户。
* Resource Sharing
  资源所有者可以管理其资源的权限，并决定谁可以访问特定资源以及如何访问。keycloak可以充当一个共享管理服务，资源所有者可以从中管理他们的资源。

Keycloak实现了UMA2.0授权规范。

比如，用户Alice使用网络银行服务管理的她的银行账户，现在她想向会计师Bob开放她的账户，但是Bob只能查看（作用域）她的账户。
网络银行服务作为资源服务器，必须有能力保护Alice的账户安全，所以这个服务使用Kaycloak资源注册端点创建代表Alice银行账户的资源。
这时，如果Bob访问Alice的银行账户，会被拒绝。网络银行服务定义了关于银行账户的默认策略，其中一条是只有账户的拥有者可以访问她的账户。
但是，网络银行服务允许Alice更改她的账户的授权策略。其中一条就是她可以定义允许谁查看她的账户。因此网络银行服务借助keycloak向Alice提供一项服务，Alice通过这项服务可以选择哪些个人以及操作可以被允许访问。Alice可以随时收回权限或授权其他权限给Bob。

#### 授权流程
