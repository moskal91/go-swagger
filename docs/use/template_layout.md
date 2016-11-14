# Custom generation

To completely customize the templates that are being considered, their file names and paths, go-swagger allows you to pass in a configuration file.
There are basically 4 types of items that are being generated: 

  * [Models](https://godoc.org/github.com/go-swagger/go-swagger/generator#GenDefinition)
  * [Operations](https://godoc.org/github.com/go-swagger/go-swagger/generator#GenOperation)
  * [Operation groups](https://godoc.org/github.com/go-swagger/go-swagger/generator#GenOperationGroup) (tagged groups of operations)
  * [Application](https://godoc.org/github.com/go-swagger/go-swagger/generator#GenApp)

You provide a configuration that describes the type of template, the source for where to find the template. For built-in templates the name should be prefixed with `asset:`.
You also provide the target directory and the file name. Directory and file names are processed as templates too and allow for a number of filters.

We use the viper library to read config values, this means you can provide the configuration file in whichever format you like: json, yaml, hcl or toml.

## Available filters in templates 

There are a number of filters you can use inside a template to manipulate values:

Filter | Description
-------|-------------
pascalize | every word boundary starts with a capital; "some long name" becomes SomeLongName
camelize | every word boundary starts with a capital except the first word; "some long name" becomes SomeLongName
varname | like camelize, but respects golint naming rules; "some url" becomes someURL
humanize | takes an identifier and adds spaces to it; someLongName becomes "some long name"
snakize | converts to lowercase and separates words with an underscore; someLongName becomes some_long_name
dasherize | convert to lowercase and separates words with a dash; someLongName becomes some-long-name
pluralizeFirstWord | pluralizes the first word of a sentence
json | converts the argument to compact json
prettyjson | converts the argument to pretty printed json
stripPackage | strips the package name from an identifier
upper | converts the string to upper case
contains | for use in conditions, returns true when the second argument is contained in the first argument, the first argument is a list of strings
joinFilePath | joins the arguments as a file path, cross platform
padSurround | surround an entry with a character n times before and m times after the entry

## Server generation

```shell
swagger generate server -A TodoList -f ./swagger.json -C default-server.yml
```

For the default server generator this config file would have the following content:

```yaml
layout:
  application:
    - name: configure
      source: asset:serverConfigureapi
      target: "{{ joinFilePath .Target .ServerPackage }}"
      file_name: "{{ .Name }}_client.go"
      skip_exists: true
    - name: main
      source: asset:serverMain
      target: "{{ joinFilePath .Target \"cmd\" (dasherize (pascalize .Name)) }}-server"
      file_name: "main.go"
    - name: embedded_spec
      source: asset:swaggerJsonEmbed
      target: "{{ joinFilePath .Target .ServerPackage }}"
      file_name: "embedded_spec.go"
    - name: server
      source: asset:serverServer
      target: "{{ joinFilePath .Target .ServerPackage }}"
      file_name: "server.go"
    - name: builder
      source: asset:serverBuilder
      target: "{{ joinFilePath .Target .ServerPackage .Package }}"
      file_name: "{{ snakize (pascalize .Name) }}_api.go"
    - name: doc
      source: asset:serverDoc
      target: "{{ joinFilePath .Target .ServerPackage }}"
      file_name: "doc.go"
  models:
    - name: definition
      source: asset:model
      target: "{{ joinFilePath .Target .ModelPackage }}"
      file_name: "{{ (snakize (pascalize .Name)) }}.go"
  operations:
    - name: parameters
      source: asset:serverParameter
      target: "{{ if gt (len .Tags) 0 }}{{ joinFilePath .Target .ServerPackage .APIPackage .Package  }}{{ else }}{{ joinFilePath .Target .ServerPackage .Package  }}{{ end }}"
      file_name: "{{ (snakize (pascalize .Name)) }}_parameters.go"
    - name: responses
      source: asset:serverResponses
      target: "{{ if gt (len .Tags) 0 }}{{ joinFilePath .Target .ServerPackage .APIPackage .Package  }}{{ else }}{{ joinFilePath .Target .ServerPackage .Package  }}{{ end }}"
      file_name: "{{ (snakize (pascalize .Name)) }}_responses.go"
    - name: handler
      source: asset:serverOperation
      target: "{{ if gt (len .Tags) 0 }}{{ joinFilePath .Target .ServerPackage .APIPackage .Package  }}{{ else }}{{ joinFilePath .Target .ServerPackage .Package  }}{{ end }}"
      file_name: "{{ (snakize (pascalize .Name)) }}.go"
  operation_groups:

```

## Client generation

```shell
swagger generate server -A TodoList -f ./swagger.json -C default-client.yml
```

For the default client generator this config file would have the following content.

```yaml
layout:
  application:
    - name: facade
      source: asset:clientFacade
      target: "{{ joinFilePath .Target .ClientPackage }}"
      file_name: "{{ .Name }}_client.go"
  models:
    - name: definition
      source: asset:model
      target: "{{ joinFilePath .Target .ModelPackage }}"
      file_name: "{{ (snakize (pascalize .Name)) }}.go"
  operations:
    - name: parameters
      source: asset:clientParameter
      target: "{{ joinFilePath .Target .ClientPackage .Package }}"
      file_name: "{{ (snakize (pascalize .Name)) }}_parameters.go"
    - name: responses
      source: asset:clientResponse
      target: "{{ joinFilePath .Target .ClientPackage .Package }}"
      file_name: "{{ (snakize (pascalize .Name)) }}_responses.go"
  operation_groups:
    - name: client
      source: asset:clientClient
      target: "{{ joinFilePath .Target .ClientPackage .Name }}"
      file_name: "{{ (snakize (pascalize .Name)) }}_client.go"
```