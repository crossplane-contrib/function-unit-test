# function-unit-test

<!-- 
[![CI](https://github.com/crossplane/function-template-go/actions/workflows/ci.yml/badge.svg)](https://github.com/crossplane/function-template-go/actions/workflows/ci.yml)

-->
This function can run CEL expressions against your desired state.

## Installing

Functions require Crossplane 1.14 or newer. Apply the following manifest to your cluster:

```yaml
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-unit-test
spec:
  package: index.docker.io/steve/function-unit-test:v0.1.0
```

## Configuring Unit Tests

See tests in [examples](examples).

After all your desired resources have been rendered with other functions,
call the unittest function and define CEL `TestCases`.

If `errorOnFailedTest` is set to true, failing tests will return an error. This is
useful when running this function in CI pipelines via `crossplane beta render`.

```yaml
 - step:
    functionRef:
      name: function-unit-test
    input:
      apiVersion: unittest.fn.crossplane.io/v1beta1
      kind: TestCases
      errorOnFailedTest: false
      testCases:
      - description: "test pass"
        assert: observed.composite.resource.spec.env == "dev"
      - description: "test fail"
        assert: observed.composite.resource.spec.env == "prod"
      # - description: "test error"
      #   assert: a == b
      - assert: |-
          "kind" in desired.resources['test-resource'].resource &&
          desired.resources['test-resource'].resource.kind == 'NopResource'
        description: all resources "test" is of "NopResource" kind
      - assert: |- 
          desired.resources.all(r, "labels" in desired.resources[r].resource.metadata && 
          "security-setting" in desired.resources[r].resource.metadata.labels &&
          desired.resources[r].resource.metadata.labels["security-setting"] == "true")
        description: All resources have the "security-setting" label
```

## Building the Function

```shell
# Run code generation - see input/generate.go
$ go generate ./...

# Run tests - see fn_test.go
$ go test ./...

# Build the function's runtime image - see Dockerfile
$ docker build . --tag=function-unit-test-runtime

# Build a function package - see package/crossplane.yaml
$ crossplane xpkg build -f package --embed-runtime-image=function-unit-test-runtime
```

[functions]: https://docs.crossplane.io/latest/concepts/composition-functions
[go]: https://go.dev
[function guide]: https://docs.crossplane.io/knowledge-base/guides/write-a-composition-function-in-go
[package docs]: https://pkg.go.dev/github.com/crossplane/function-sdk-go
[docker]: https://www.docker.com
[cli]: https://docs.crossplane.io/latest/cli