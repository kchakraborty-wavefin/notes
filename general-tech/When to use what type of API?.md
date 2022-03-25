##### GraphQL
GraphQL is a query language used to interact with APIs. It allows users to fetch only the data their apps require from the server and nothing more. So every time you send a GraphQL query to your server API endpoint, you're guaranteed to get a predictable data response. GraphQL works as an abstraction layer over the API, requiring only a single endpoint to access data.

Use when:
- **applications are bandwitdth-sensitive**: critical to manage the requested data
- **applications that use data from multiple data stores**: graphQL can build responses from multiple data sources. All the data can then be nested into a single response, making it useful for applications such as reporting dashboards.

##### gRPC
gRPC or Google Remote Procedure Call, an implementation of remote procedure calls developed by Google, is used to run operations on remote servers. Unlike GraphQL, where client requests rely on predefined schemas to receive data in a specific format, gRPC functions through contracts defined by the client and server.

Use when:
- **real-time streaming communication services**
- **micro-service architecture**: protobufs allow for multiple services to communicate with each other independently using their preferred programming language

##### REST
Use When:
- **developing public APIs**: standardized so better to be consumed by large number of clients
- **simple applications**: quick to implement, good option for simple apps

##### Webhooks
Use when:
- **developing automation services**: easy to develop automation services that can be _triggered_ by servers when required
- **notification services**: post data to clients when _triggered_ to updae notifis/messages

#### References
- https://fauna.com/blog/when-to-use-graphql-grpc-rest-webhooks