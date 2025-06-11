# Go-GRPC Setup

## 1. Manual

| Tools              | Usage                                    | Ouput                                              |
|--------------------|------------------------------------------|----------------------------------------------------|
| protoc             | Protocol Buffers compiler                | Returns nothing, unless errors are found           |
| protoc-gen-go      | Go code generator for protobuf           | *.pb.go files (messages/structs and enums)         |
| protoc-gen-go-grpc | Go code generator for gRPC service stubs | *_grpc.pb.go files (service stubs and client APIs) |

`protoc` is the Protocol Buffers compiler, a command-line tool provided by Google. It has two core responsibilities

- Parse `.proto` definitions into an internal abstract syntax tree (AST)
- Delegate code generation to external plugins using this AST

Plugins are separate executables that follow a protocol to receive the `.proto` AST from `protoc` and return generated source code

```sh
protoc proto/service/greeter.proto

# Return generated code for both client & server
protoc --go_out=. --go-grpc_out=. proto/service/greeter.proto
```

## 2. Buf

| Tools           | Usage           | Ouput |
|-----------------|-----------------|-------|
| Buf             |                 |       |
