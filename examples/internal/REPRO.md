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


Proto files generation creates the following files:

The proto file:

```
grpc-gateway/examples/internal/examplepb/a_bit_of_everything.pb.go
```
Contains these types:
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
```

The grpc-gateway proto:
```
grpc-gateway/examples/internal/examplepb/a_bit_of_everything.pb.gw.go
```
Contains these functions at line 2680.
Which must reference variable named: `SnakeCaseEnum_value`
From the proto file, but instead,
Were generated referecing variable named: `snake_case_enum_value`
```
func request_CamelvsSnakeEnum_SnakeEnum_0(ctx context.Context, marshaler runtime.Marshaler, client CamelvsSnakeEnumClient, req *http.Request, pathParams map[string]string) (proto.Message, runtime.ServerMetadata, error) {
	var protoReq SnakeEnumRequest
	var metadata runtime.ServerMetadata

	var (
		val string
		e   int32
		ok  bool
		err error
		_   = err
	)

	val, ok = pathParams["what"]
	if !ok {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "missing parameter %s", "what")
	}

	e, err = runtime.Enum(val, snake_case_enum_value)
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", "what", err)
	}

	protoReq.What = snake_case_enum(e)

	msg, err := client.SnakeEnum(ctx, &protoReq, grpc.Header(&metadata.HeaderMD), grpc.Trailer(&metadata.TrailerMD))
	return msg, metadata, err

}
```

```
func local_request_CamelvsSnakeEnum_SnakeEnum_0(ctx context.Context, marshaler runtime.Marshaler, server CamelvsSnakeEnumServer, req *http.Request, pathParams map[string]string) (proto.Message, runtime.ServerMetadata, error) {
	var protoReq SnakeEnumRequest
	var metadata runtime.ServerMetadata

	var (
		val string
		e   int32
		ok  bool
		err error
		_   = err
	)

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

The code on those areas is generated with a template
at file: `/protoc-gen-grpc-gateway/internal/gengateway/template.go`

At line 532:
```
{{if $param.IsNestedProto3}}
	err = runtime.PopulateFieldFromPath(&protoReq, {{$param | printf "%q"}}, val)
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", {{$param | printf "%q"}}, err)
	}
	{{if $enum}}
		e{{if $param.IsRepeated}}s{{end}}, err = {{$param.ConvertFuncExpr}}(val{{if $param.IsRepeated}}, {{$binding.Registry.GetRepeatedPathParamSeparator | printf "%c" | printf "%q"}}{{end}}, {{$enum.GoType $param.Method.Service.File.GoPkg.Path}}_value)
		if err != nil {
			return nil, metadata, status.Errorf(codes.InvalidArgument, "could not parse path as enum value, parameter: %s, error: %v", {{$param | printf "%q"}}, err)
		}
	{{end}}
{{else if $enum}}
	e{{if $param.IsRepeated}}s{{end}}, err = {{$param.ConvertFuncExpr}}(val{{if $param.IsRepeated}}, {{$binding.Registry.GetRepeatedPathParamSeparator | printf "%c" | printf "%q"}}{{end}}, {{$enum.GoType $param.Method.Service.File.GoPkg.Path}}_value)
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", {{$param | printf "%q"}}, err)
	}
{{else}}
	{{- $protoReq := $param.AssignableExprPrep "protoReq" -}}
	{{- if ne "" $protoReq }}
	{{printf "%s" $protoReq }}
	{{- end}}
	{{$param.AssignableExpr "protoReq"}}, err = {{$param.ConvertFuncExpr}}(val{{if $param.IsRepeated}}, {{$binding.Registry.GetRepeatedPathParamSeparator | printf "%c" | printf "%q"}}{{end}})
	if err != nil {
		return nil, metadata, status.Errorf(codes.InvalidArgument, "type mismatch, parameter: %s, error: %v", {{$param | printf "%q"}}, err)
	}
{{end}}
```
