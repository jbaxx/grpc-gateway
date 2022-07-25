Add to `grpc-gateway/examples/internal/proto/examplepb/a_bit_of_everything.proto`:
```
service CamelvsSnakeEnum {
    rpc CamelEnum(CamelEnumRequest) returns (CamelEnumResponse) {
        option (google.api.http) = {
            get: "/v1/example/camelenum/{what}"
        };
    }

    rpc SnakeEnum(SnakeEnumRequest) returns (SnakeEnumResponse) {
        option (google.api.http) = {
            get: "/v1/example/snake/{what}"
        };
    }
}

enum CamelCaseEnum {
    value_a = 0;
    value_b = 1;
}

message CamelEnumRequest {
    CamelCaseEnum what = 1;
}

message CamelEnumResponse {}

enum snake_case_enum {
    value_c = 0;
    value_d = 1;
}

message SnakeEnumRequest {
    snake_case_enum what = 1;
}

message SnakeEnumResponse {}
```

Then run `buf generate` at root.

Then run a main like:
```
go run examples/internal/cmd/example-gateway-server/main.go
```
The same with:
```
go run examples/internal/cmd/example-grpc-server/main.go
```

Output
```
# github.com/grpc-ecosystem/grpc-gateway/v2/examples/internal/proto/examplepb
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2697:29: undefined: snake_case_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2702:18: undefined: snake_case_enum
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2726:29: undefined: snake_case_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2731:18: undefined: snake_case_enum
```


