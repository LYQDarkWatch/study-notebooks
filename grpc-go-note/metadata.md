# gRPC 的 metadata 使用

在 http 请求中我们可以设置 header 来传递数据， gRPC 底层采用 http2 协议也是支持数据传递的， 采用的是 metadata。 Metadata 对于 gRPC 本身来说透明，他使得 client 和 server 能为对方提供本次调用的信息。就像一次 http 请求的 RequestHeader 和 REsponseHeader，http header 的生命周期是一次 http 请求， Metadata 的生命周期则是一次 RPC 调用。

## 简析

### 创建 metadata

Metadata 实际上是 map，key 是 string，value 是 string 类型的 slice。

```go
type MD map[string][]string
```

创建的时候可以像普通的 map 类型一样使用 new 关键字创建：

```go
md := metadata.New(map[string]string{"key1": "value1", "key2": "value2"})
```

或者使用 Paris 创建，相同的 key 值会被组合成 slice。

```go
md := metadata.Pairs(
    "key1", "value1",
    "key1", "value2",   // "key1" will have map value []string{"val1", "val1-2"}
    "key2", "value3",
)
```

key 不区分大小写，会被统一转成小写。

### 发送 matadata

```go
md := metadata.Pairs("key", "val")

// 新建一个有 metadata 的 context
ctx := metadata.NewOutgoingContext(context.Background(), md)

// 单向 RPC
response, err := client.SomeRPC(ctx, someRequest)
```

### 接收 metadata

利用函数 FromIncomingContext 从 context 中获取 metadata:

```go
func (s *server) SomeRPC(ctx context.Context, in *pb.SomeRequest) (*pb.SomeResponse, err) {
    md, ok := metadata.FromIncomingContext(ctx)
    // do something with metadata
}
```
