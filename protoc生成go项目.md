# Protoc 生成 go 项目代码

**参考文档**

[Go Generated Code by Protocol Buffers ](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

[Protobuf 入门（大白话版） - 掘金](https://juejin.cn/post/6865126893063471112#heading-13)

---

首先需要通过以下指令下载 `protoc` 编译插件：

```bash
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

上面一个是用来生成对应 `message` 的代码，下面一个是专门针对 gPRC 使用。

下面会通过一个简单的例子来讲述代码生成的位置和对应指令。

```protobuf
syntax = "proto3";

package users;

option go_package = "protoc_gen/user_management";

service UserManagement {
  rpc CreateNewUser (NewUser) returns (User) {}
}

message NewUser {
  string name = 1;
  int32 age = 2;
}

message User {
  string name = 1;
  int32 age = 2;
  int64 id = 3;
}
```

下面是一些官方文档中的原文说明：

> You can add an optional `package` specifier to a `.proto` file to prevent name clashes between protocol message types.

&emsp;

> In order to generate Go code, the Go package's import path must be provided for every `.proto` file (including those transitively depended upon by the `.proto` files being generated). There are two ways to specify the Go import path:
> 
> - by declaring it within the `.proto` file, or
> - by declaring it on the command line when invoking `protoc`.

&emsp;

> The name of the output file is created by replacing the `.proto` extension with `.pb.go`.

&emsp;

* 在这个 `.proto` 文件中，`package` 代表这的是 `protobuf` 这个文件所在的作用域，从而避免和其他 `.proto` 文件中的命名产生冲突，原文是  当你在其他文件引用 `NewUser` 时，应该写成 `users.NewUser`。

* 除了这个之外，还需要指明生成的 go 代码文件所在的 `package`，每一个 `.proto` 文件都需要这样一个值，又被称为 `Go import path`，这个就是 `option go_package` 的作用。

* 最终 go 生成代码文件的地址由两部分决定，输出目录和相应的子目录。输出目录是由 `protoc` 指令中的 `--go_out` 参数决定，事先需要确保这个目录存在，一般是工程目录；子目录由 `go_package` 中声明的路径一直向下，其中最后一个目录就是 go 文件的包名，例如上面的 `NewUser` 结构体对应的 `package` 名称就是 management。

&emsp;

生成的指令如下：

```bash
protoc --proto_path=../idl --go_out=../ --go-grpc_out=../ user.proto
```

最终文件的结构如下：

```bash
${basedir}
|-- cmd
|    |-- protoc_gen.sh
|-- idl
|    |-- user.proto
|-- protoc_gen
|    |-- user_management
|         |-- user.pb.go
|         |-- user_grpc.pb.go 
|         |
```

一般在工程上也会采用通过 `Bash` 将各种指令直接写在项目中，便于后续直接使用。
