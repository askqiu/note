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