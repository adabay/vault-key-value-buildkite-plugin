# Vault Key/Value Buildkite Plugin

|Linting|Tests|
|---|---|
|-|-|

This plugin enables pipelines to use secrets from a Hashicorp Vault instance (Key/Value Secrets Engine) as environment variables.

Pipeline steps can define multiple secrets that should be provided as environment variables to the subsequent hooks.
Secrets have to be specified by their path and their key.

To authorize the access to Vault, this plugin provides 2 methods:
* Use a preexisting token environment variable (`VAULT_TOKEN`). This could be injected by an agent environment hook for example.
* Use a token that was written to a file. This is the appropriate method to authorize if you use the Vault Agent's auto-auth method to keep a valid token on the build agent.
To use this method, you have to specify the path to the token file using the plugin parameter `token_file_path` or using the environment variable `BUILDKITE_PLUGIN_VAULT_TOKEN_FILE_PATH`.

## Example

### Single secret example

```yml
steps:
  - command: 'curl -H "Authorization: Bearer $API_ACCESS_TOKEN" https://api.example.com'
    plugins:
      - maierj/vault#master:
          secret_path: "static/api_access_token"
          secret_key: "token"
          exported_env_variable_name: "API_ACCESS_TOKEN"
```

### Multiple secrets example

```yml
steps:
  - command: 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
    plugins:
      - maierj/vault#master:
          secrets:
          - secret_path: "static/docker/registry1"
            secret_key: "username"
            exported_env_variable_name: "DOCKER_USERNAME"
          - secret_path: "static/docker/registry1"
            secret_key: "password"
            exported_env_variable_name: "DOCKER_PASSWORD"
```

## Configuration

### `secrets` (array)

If you want to export multiple secrets, you can use this array. Each entry in this array has to have the configuration properties that are listed below.

### `secret_path` (string)

This parameter defines the path of a secret. If you only want to export a single secret, you can specify this parameter at the top-level of the plugin configuration (See single secret example). Otherwise, specify it for each entry in the `secrets` array.

### `secret_key` (string)

Since there can be multiple key/value entries in a single secret, you have to specify the key of the entry that you want to export. This is done using this configuration parameter. If you only want to export a single secret, you can specify this parameter at the top-level of the plugin configuration (See single secret example). Otherwise, specify it for each entry in the `secrets` array.

### `exported_env_variable_name` (string)

With this parameter you can define the name of the environment variable that you want to set to the value of the secret entry. If you only want to export a single secret, you can specify this parameter at the top-level of the plugin configuration (See single secret example). Otherwise, specify it for each entry in the `secrets` array.

## Developing

To run the linter:

```shell
docker run -it --rm -v "${PWD}:/plugin:ro" buildkite/plugin-linter --id adabay/vault
```
