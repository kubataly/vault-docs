# Agent Sidecar Injector
The Vault Agent Injector alters pod specifications to include Vault Agent containers that render Vault secrets to a shared memory volume using
Vault Agent Templates. By rendering secrets to a shared volume, containers within the pod can consume Vault secrets without being Vault aware.

At a minimum, every container in the pod will be configured to mount a shared memory volume. This volume is mounted to `/vault/secrets` and will be used by the Vault Agent containers for sharing secrets with the other containers in the pod.

Next, two types of Vault Agent containers can be injected: init and sidecar. The init container will prepopulate the shared memory volume with the requested secrets prior to the other containers starting. The sidecar container will continue to authenticate and render secrets to the same location as the pod runs. Using annotations, the initialization and sidecar containers may be disabled.

# Requesting Secrets from HCP Vault
There are two methods of configuring the Vault Agent containers to render secrets:
  - the `vault.hashicorp.com/agent-inject-secret` annotation, or
  - a configuration map containing Vault Agent configuration files.
  
Only one of these methods may be used at any time.

### Secrets via Annotations
To configure secret injection using annotations, the user must supply:
 - one or more _secret_ annotations, and
 - the Vault role used to access those secrets.
The annotation must have the format:

`vault.hashicorp.com/agent-inject-secret-<unique-name>: /path/to/secret/in_vault`

The unique name will be the **filename** of the rendered secret and must be unique. 
For example, consider the following secret annotations:
```
vault.hashicorp.com/agent-inject-secret-foo: database/roles/app
vault.hashicorp.com/role: 'app'
```
The secret unique name must consist of alphanumeric characters, `.`, `_` or `-`.

# Secret Templates
How the secret is rendered to the file is also configurable. To configure the template used, the user must supply a template annotation using the same unique name of the secret. The annotation must have the following format:
```
vault.hashicorp.com/agent-inject-template-<unique-name>: |
  <
    TEMPLATE
    HERE
  >

```
For example, consider the following:
```
vault.hashicorp.com/agent-inject-secret-foo: 'database/creds/db-app'
vault.hashicorp.com/agent-inject-template-foo: |
  {{- with secret "database/creds/db-app" -}}
  postgres://{{ .Data.username }}:{{ .Data.password }}@postgres:5432/mydb?sslmode=disable
  {{- end }}
vault.hashicorp.com/role: 'app'
```
The rendered secret would look like this within the container:
```
$ cat /vault/secrets/foo
postgres://v-kubernet-pg-app-q0Z7WPfVN:A1a-BUEuQR52oAqPrP1J@postgres:5432/mydb?sslmode=disable
```
> The default left and right template delimiters are `{{` and `}}`.

If no template is provided the following generic template is used:
```
{{ with secret "/path/to/secret" }}
    {{ range $k, $v := .Data }}
        {{ $k }}: {{ $v }}
    {{ end }}
{{ end }}
```

# Installing Vault with Helm

```
kubectl  -n external-secrets exec -ti _<pod_name>_ -- cat /vault/secrets/db-creds
kubectl logs _<pod_name>_ -c vault-agent-init
```
