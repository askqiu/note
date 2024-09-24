# 寻找
首先找到graphql提供服务的那条url
## 通用查询
把`query{__typename}`发到任何graphql的endpoint，会在返回的某个位置返回`{"data": {"__typename": "query"}}`           这是一个探测url是否对应graphql服务的方法

查询之所以有效，是因为每个 GraphQL 终端节点都有一个名为 `__typename` 的保留字段，该字段以字符串形式返回查询对象的类型。
## 路径特征
GraphQL 服务通常使用类似的终端节点后缀。在测试 GraphQL 终端节点时，您应该考虑将通用查询发送到以下位置：

- /graphql
- /api
- /api/graphql
- /graphql/api
- /graphql/graphql
如果这些常见终端节点未返回 GraphQL 响应，您还可以尝试将 **/v1** 附加到路径。
GraphQL services will often respond to any non-GraphQL request with a "query not present" or similar error. You should bear this in mind when testing for GraphQL endpoints.
GraphQL 服务通常会使用“查询不存在”或类似错误来响应任何非 GraphQL 请求。在测试 GraphQL 终端节点时，您应该牢记这一点。
# 请求方法
一般是只接受content type 为application/json的post请求（可以防御csrf）
但是也有可能接受 ct为x-www-form-urlencoded的get或者post请求
如果无法通过向常见终端节点发送 POST 请求来找到 GraphQL 终端节点，尝试使用替代 HTTP 方法重新发送通用查询。
发现终端节点后，您可以发送一些测试请求，以进一步了解其工作原理。如果端点正在为网站提供支持，请尝试在 Burp 的浏览器中浏览 Web 界面，并使用 HTTP 历史记录来检查发送的查询。

## intrpspection
您可以使用以下简单查询来探测[[01.开发/架构知识/graphql|graphql]]  introspection。如果启用了自省，则响应将返回所有可用查询的名称。


    #Introspection probe request

    {
        "query": "{__schema{queryType{name}}}"
    }
下一步是对终端节点运行完整的自省查询，以便您可以获取有关底层架构的尽可能多的信息。
以下示例查询返回有关所有查询、更改、订阅、类型和片段的完整详细信息。


    #Full introspection query

    query IntrospectionQuery {
        __schema {
            queryType {
                name
            }
            mutationType {
                name
            }
            subscriptionType {
                name
            }
            types {
             ...FullType
            }
            directives {
                name
                description
                args {
                    ...InputValue
            }
            onOperation  #Often needs to be deleted to run query
            onFragment   #Often needs to be deleted to run query
            onField      #Often needs to be deleted to run query
            }
        }
    }

    fragment FullType on __Type {
        kind
        name
        description
        fields(includeDeprecated: true) {
            name
            description
            args {
                ...InputValue
            }
            type {
                ...TypeRef
            }
            isDeprecated
            deprecationReason
        }
        inputFields {
            ...InputValue
        }
        interfaces {
            ...TypeRef
        }
        enumValues(includeDeprecated: true) {
            name
            description
            isDeprecated
            deprecationReason
        }
        possibleTypes {
            ...TypeRef
        }
    }

    fragment InputValue on __InputValue {
        name
        description
        type {
            ...TypeRef
        }
        defaultValue
    }

    fragment TypeRef on __Type {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
                ofType {
                    kind
                    name
                }
            }
        }
    }

如果启用了内省，但上述查询未运行，请尝试从查询结构中删除 onOperation、onFragment 和 onField 指令。许多终端节点不接受这些指令作为内省查询的一部分，您通常可以通过删除它们来获得更大的内省成功。
查询成功的话数据很很大很难看，可以用工具看，graphql可视化工具[http://nathanrandal.com/graphql-visualizer/](http://nathanrandal.com/graphql-visualizer/)

## 建议查询
建议是 Apollo GraphQL 平台的一项功能，服务器可以在错误消息中建议查询修正。这些通常用于查询略微不正确但仍可识别的情况（例如， There is no entry for 'productInfo'. Did you mean 'productInformation' instead? ）。
您可能会从中收集有用的信息，因为响应实际上泄露了架构的有效部分。
Clairvoyance 是一种工具，它使用建议自动恢复全部或部分 GraphQL 架构，即使禁用了内省也是如此。这使得从建议响应中拼凑信息的时间大大减少。

You cannot disable suggestions directly in Apollo. See this GitHub thread for a workaround.
您不能直接在 Apollo 中禁用建议。有关解决方法，请参阅此 [GitHub](https://github.com/apollographql/apollo-server/issues/3919#issuecomment-836503305) 帖子。
# 绕过 GraphQL 内省防御
如果您无法为正在测试的 API 运行自省查询，请尝试在 `__schema 关键字后插入特殊字符`。
当开发人员禁用自省时，他们可以使用正则表达式在查询中排除 `__schema` 关键字。您应该尝试使用空格、换行符和逗号等字符，因为它们会被 GraphQL 忽略，但不会被有缺陷的正则表达式忽略。
因此，如果开发人员仅排除了 `__schema{`，则不会排除以下自省查询。


    #Introspection query with newline

    {
        "query": "query{__schema
        {queryType{name}}}"
    }

如果这不起作用，请尝试通过替代请求方法运行探测，因为内省可能只能通过 POST 禁用。尝试 GET 请求，或 content-type 为 x-www-form-urlencoded 的 POST 请求。
以下示例显示了通过 GET 发送的内省探测，其中包含 URL 编码的参数。


    # Introspection probe as GET request

    GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D

# 别名绕过速率限制
GraphQL 允许通过别名在单个请求中执行多个不同的查询操作。攻击者利用这一点，可以一次性请求大量的数据，而不必分多次请求发起。例如，如果 API 服务器对请求频率有限制（如每秒最多10个请求），攻击者可以在一个请求中模拟几十甚至上百个查询操作，快速获取大量数据，绕过速率限制。
绕过速率限制的攻击方式
速率限制通常是基于请求数来实现的，比如限制客户端在某段时间内只能发送 100 次 API 请求。然而，在 GraphQL 中，由于别名的存在，攻击者可以在单个请求中伪装出多个不同的请求，从而绕过速率限制。

举个例子，如果一个 API 对单次请求进行速率限制，攻击者可以在一次 GraphQL 请求中通过别名模拟多个请求：

```
{
  user1: user(id: 1) { name }
  user2: user(id: 2) { name }
  user3: user(id: 3) { name }
  user4: user(id: 4) { name }
  ...
  user100: user(id: 100) { name }
}
```
虽然这看起来像是一个请求，但实际上它让服务器在同一个请求中执行了 100 次不同的数据查询。因为这些查询都在同一请求中，API 的速率限制（比如每秒 10 个请求）可能无法检测到这种情况，导致服务器负载增加，甚至引发拒绝服务（DoS）攻击。
# graphql csrf
当 GraphQL 端点不验证发送给它的请求的内容类型并且未实施 CSRF 令牌时，可能会出现 CSRF 漏洞。
只要内容类型经过验证，使用 application/json 内容类型的 POST 请求就可以防止伪造。在这种情况下，即使受害者访问恶意站点，攻击者也无法让受害者的浏览器发送此请求。
但是，浏览器可以发送其他方法（如 GET）或任何内容类型为 x-www-form-urlencoded 的请求，因此，如果终端节点接受这些请求，则可能会使用户容易受到攻击。在这种情况下，攻击者可能能够精心设计漏洞以向 API 发送恶意请求。
使用 application/json 限制 CSRF 攻击
## 为什么json防止csrf
使用 application/json 作为 Content-Type 的主要优势是：浏览器在表单提交和 URL 重定向时不会自动生成或发送这种类型的请求。

Form 表单限制：HTML 表单的 Content-Type 只能是 application/x-www-form-urlencoded、multipart/form-data 或 text/plain，而不能直接发送 application/json 类型的数据。

XMLHttpRequest 和 Fetch 限制：在跨站点请求中，浏览器出于安全考虑，对于 application/json 类型的请求会自动执行 CORS（跨源资源共享）策略检查。如果攻击者试图通过 JavaScript 使用 XMLHttpRequest 或 fetch 来发送 application/json 类型的请求，浏览器会首先发起预检请求（OPTIONS 请求），以确认目标服务器是否允许跨域请求。

因此，即使攻击者通过某些方式诱导用户访问恶意网站，浏览器也不会自动发送 application/json 格式的请求。这使得攻击者无法构造符合服务器预期的请求，从而有效地防止了 CSRF 攻击。
# 防止graphql攻击
t is sometimes possible to bypass standard rate limiting when using GraphQL APIs. For an example of this, see the Bypassing rate limiting using aliases section.

With this in mind, there are design steps that you can take to defend your API against brute force attacks. This generally involves restricting the complexity of queries accepted by the API, and reducing the opportunity for attackers to execute denial-of-service (DoS) attacks.

To defend against brute force attacks:

Limit the query depth of your API's queries. The term "query depth" refers to the number of levels of nesting within a query. Heavily-nested queries can have significant performance implications, and can potentially provide an opportunity for DoS attacks if they are accepted. By limiting the query depth your API accepts, you can reduce the chances of this happening.

Configure operation limits. Operation limits enable you to configure the maximum number of unique fields, aliases, and root fields that your API can accept.

Configure the maximum amount of bytes a query can contain.

Consider implementing cost analysis on your API. Cost analysis is a process whereby a library application identifies the resource cost associated with running queries as they are received. If a query would be too computationally complex to run, the API drops it.
专门防御 GraphQL CSRF 漏洞，请在设计 API 时确保满足以下条件：

Your API only accepts queries over JSON-encoded POST.
您的 API 仅接受通过 JSON 编码的 POST 进行的查询。

The API validates that content provided matches the supplied content type.
API 验证提供的内容是否与提供的内容类型匹配。

The API has a secure CSRF token mechanism.
API 具有安全的 CSRF 令牌机制。

# graphql美化
`http://nathanrandal.com/graphql-visualizer`