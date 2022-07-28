I'm interested in trying to fix this.

## TL; DR
I've been able to reproduce the issue, looks like identifiers for enums in the `*.pb.gw.go` file doesn't match those in the `*.pb.go` file when enums are snake-cased. Also, I'd appreciate some guidance on the path to test and fix as mentioned in the fix proposal below.

## Reproducing the issue

I was able to reproduce the issue as well by:
1. Adding snake-cased enums to `a_bit_of_everything.proto`
2. Re-generating the `.pb.*` files with `buf`.
3. Trying to `go build` or `go run` any `main` package that references the `examplepb` package generated from `a_bit_of_everythingpb.proto` like `examples/internal/cmd/example-gateway-server/main.go`

Enums and service added to `a_bit_of_everything.proto`:

```
service SnakeEnum {

    rpc SnakeEnum(SnakeEnumRequest) returns (SnakeEnumResponse) {
        option (google.api.http) = {
            get: "/v1/example/snake/{who}/{what}"
        };
    }
}

enum snake_case_enum {
    value_c = 0;
    value_d = 1;
}

enum snake_case_0_enum {
    value_e = 0;
    value_f = 1;
}

message SnakeEnumRequest {
    snake_case_enum what = 1;
    snake_case_0_enum who = 2;
}

message SnakeEnumResponse {}
```

`go build` and/or `go run` output:
```
# github.com/grpc-ecosystem/grpc-gateway/v2/examples/internal/proto/examplepb
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2639:29: undefined: snake_case_0_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2644:17: undefined: snake_case_0_enum
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2651:29: undefined: snake_case_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2656:18: undefined: snake_case_enum
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2680:29: undefined: snake_case_0_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2685:17: undefined: snake_case_0_enum
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2692:29: undefined: snake_case_enum_value
examples/internal/proto/examplepb/a_bit_of_everything.pb.gw.go:2697:18: undefined: snake_case_enum
```

## Investigation

So far, looks like the proto generator `protoc-gen-go` creates the types/variables's identifiers related to the enums Camel-cased in the `examples/internal/proto/examplepb/a_bit_of_everything.pb.go` file:

```
type SnakeCaseEnum int32

const (
	SnakeCaseEnum_value_c SnakeCaseEnum = 0
	SnakeCaseEnum_value_d SnakeCaseEnum = 1
)

// Enum value maps for SnakeCaseEnum.
var (
	SnakeCaseEnum_name = map[int32]string{
		0: "value_c",
		1: "value_d",
	}
	SnakeCaseEnum_value = map[string]int32{
		"value_c": 0,
		"value_d": 1,
	}
)

// ...

type SnakeCase_0Enum int32

const (
	SnakeCase_0Enum_value_e SnakeCase_0Enum = 0
	SnakeCase_0Enum_value_f SnakeCase_0Enum = 1
)

// Enum value maps for SnakeCase_0Enum.
var (
	SnakeCase_0Enum_name = map[int32]string{
		0: "value_e",
		1: "value_f",
	}
	SnakeCase_0Enum_value = map[string]int32{
		"value_e": 0,
		"value_f": 1,
	}
)
```

While the grpc-gateway proto generator `protoc-gen-grpc-gateway` generates the derived enum identifiers without Camel-casing (no modification to the identifier actually) within the following method in the `examples/internal/proto/examplepb/a_bit_of_everything.pb.go` file:
```
func local_request_SnakeEnum_SnakeEnum_0(ctx context.Context, marshaler runtime.Marshaler, server SnakeEnumServer, req *http.Request, pathParams map[string]string) (proto.Message, runtime.ServerMetadata, error) {
	var protoReq SnakeEnumRequest
	var metadata runtime.ServerMetadata

	var (
		val string
		e   int32
		ok  bool
		err error
		_   = err
	)

	val, ok = pathParams["who"]
	if !ok {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "missing parameter %s", "who")
	}

	e, err = runtime.Enum(val, snake_case_0_enum_value)
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", "who", err)
	}

	protoReq.Who = snake_case_0_enum(e)

	val, ok = pathParams["what"]
	if !ok {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "missing parameter %s", "what")
	}

	e, err = runtime.Enum(val, snake_case_enum_value)
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", "what", err)
	}

	protoReq.What = snake_case_enum(e)

	msg, err := server.SnakeEnum(ctx, &protoReq)
	return msg, metadata, err

}
```

Causing the derived enum identifiers in the `.pb.gw.go` to be undeclared, and a compilation issue.

### Identifiers mismatch summary:

| `.pb.go` identifiers | `.pb.gw.go` identifiers |
| ------------------- | ---------------------- |
| `SnakeCase_0Enum` | `snake_case_0_enum` |
| `SnakeCaseEnum_value` | `snake_case_0_enum_value` |
| `SnakeCaseEnum` | `snake_case_enum` |
| `SnakeCaseEnum_value` | `snake_case_enum_value` |

## Fix proposal

I suspect the fix is to apply the [Camel-casing function](https://github.com/grpc-ecosystem/grpc-gateway/blob/672f079d1ec726166bbdf413b1fa24bf6da2cb35/internal/casing/camel.go#L14) to the enum names as is done for the service and method names in the [applyTemplate](https://github.com/grpc-ecosystem/grpc-gateway/blob/672f079d1ec726166bbdf413b1fa24bf6da2cb35/protoc-gen-grpc-gateway/internal/gengateway/template.go#L151) function.

But in this case, the difference is that looping through the service and methods' names (+ casing) is done in the code, while looping through the enum's names is done [within the template](https://github.com/grpc-ecosystem/grpc-gateway/blob/672f079d1ec726166bbdf413b1fa24bf6da2cb35/protoc-gen-grpc-gateway/internal/gengateway/template.go#L543).

Could you point if I'm in the right direction and hint a path for the fix?
Also some guidance on testing this will be great, thanks!
