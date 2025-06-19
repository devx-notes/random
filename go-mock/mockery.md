# Mockery Setup

Visit the releases page to download one of the pre-built binaries

```sh
https://github.com/vektra/mockery/releases
```

Compile mockery from source with `go install` is supported, but not recommended

```sh
go install github.com/vektra/mockery/v3@v3.4.0
```

## mockery init

The `mockery init module_name` command creates a fully-functioning configuration set

```sh
mockery init gitlab.com/markdown-online/mvp/internal
```

The default `.mockery.yml` file for module gitlab.com/markdown-online/mvp/internal will look like this:

- [Parameter Descriptions](https://vektra.github.io/mockery/latest/configuration/#parameter-descriptions)

```yml
all: false # Generate all interfaces for the specified packages.
dir: '{{.InterfaceDir}}'
filename: mocks_test.go
force-file-write: true
formatter: goimports
log-level: info
structname: '{{.Mock}}{{.InterfaceName}}'
pkgname: '{{.SrcPackageName}}' # The package name given to the generated mock files
recursive: false # Recursively search for all sub-packages and inject those packages into the config map
require-template-schema-exists: true
template: testify
template-schema: '{{.Template}}.schema.json'
packages: # A dictionary containing configuration describing the packages and interfaces to generate mocks for.
  gitlab.com/markdown-online/mvp/internal:
    config: # Set the location of the mockery config file.
      all: true # Generate all interfaces for the specified packages.

```

Run `mockery` against this config to generate the code. Mockery will scan all files under the directory and generate mocks for any interfaces it finds.

```sh
2025-06-17T10:45:59.020489600+07:00 INF Starting mockery config-file=C:/SSO/sso/.mockery.yml version=v3.4.0
2025-06-17T10:45:59.022185200+07:00 INF Parsing configured packages... version=v3.4.0
2025-06-17T10:46:00.483832700+07:00 INF Done parsing configured packages. version=v3.4.0
2025-06-17T10:46:00.484381800+07:00 INF adding interface to collection collection=C:/SSO/sso/internal/client/bps/mocks_test.go interface=Client package-path=gitlab.com/gookie/sso/internal/client/bps version=v3.4.0  
2025-06-17T10:46:00.486148900+07:00 INF Executing template file=C:/SSO/sso/internal/client/bps/mocks_test.go version=v3.4.0
2025-06-17T10:46:00.490861400+07:00 INF Writing template to file file=C:/SSO/sso/internal/client/bps/mocks_test.go version=v3.4.0
```

## Variables

Predefined variables is passed to the TemplateData struct. TemplateData is the data sent to the template for the config file.

- **ConfigDir**: the directory of where the mockery config file is located.
- **InterfaceDir**: the directory of the interface being mocked.
- **InterfaceDirRelative** is the same as InterfaceDir, but made relative to the ConfigDir.
- **InterfaceFile** is the filename of where the interface is defined.
- **InterfaceName**: the name of the interface.
- **Mock**: a parameter that takes the value of "Mock" if the interface is exported, and "mock" otherwise.
- **StructName**: the configured name of the mock.
- **SrcPackageName**: the name of the source package.
- **SrcPackagePath**: the fully qualified package path of the source package. e.g. "github.com/vektra/mockery/v3".
- **Template**: the value of the `template` parameter.

### Custom Output 

Output name can be custom using these keywords: lower, upper, camelcase, snakecase, kebabcase, firstLower, firstUpper

```yml
all: false
dir: "mocks/{{ .InterfaceDirRelative }}"
filename: "{{ .InterfaceName | snakecase }}.go"
force-file-write: true
formatter: goimports
log-level: info
structname: '{{.Mock}}{{.InterfaceName}}'
pkgname: '{{.SrcPackageName}}'
recursive: false
require-template-schema-exists: true
template: testify
template-schema: '{{.Template}}.schema.json'
packages:
  gitlab.com/gookie/sso/internal/client:
    config:
      all: true
      recursive: true
      dir: "mocks/client/{{ .SrcPackageName }}"
      pkgname: "mock{{.SrcPackageName}}"
      filename: "{{ .SrcPackageName | snakecase }}.go"
  gitlab.com/gookie/sso/internal/rest/api/v1:
    config:
      all: true
      recursive: true
      dir: "mocks/v1/"
      pkgname: "mockv1"
  gitlab.com/gookie/sso/internal/cache:
    config:
      all: true
      recursive: true
      dir: "mocks/cache/"
      pkgname: "mock{{.SrcPackageName}}"
  gitlab.com/gookie/sso/pkg/redis:
    config:
      all: true
      pkgname: "mockredis"
```

### v3 Migration

-  v3 no longer allowed generating mocks for function types
- `resolve-type-alias` is permanently set to `false`
- `keeptree` has been removed
- `inpackage` has been removed 
- `exclude` is renamed to `exclude-subpkg-regex`
- `unroll-variadic` has been moved under the `template-data`
- `with-expecter` has been removed, testify-style mocks will always generate expecter methods.
