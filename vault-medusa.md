# Installation
1. Clone the repo from Git repositoty using command `git clone git@github.com:jonasvinther/medusa.git`
2. Go to `medusa` folder using command:
    ```
    cd medusa
    ```
3. Now we need to build this package. Run the command below:
    P.S.: Befor run this command check `golang` version. It should be < 17.0
    ```
    go build .
    ```

# Usage
Befor using `Medusa` export Vault's address and Token

    export VAULT_ADDR="https://vault.com:8200"; export VAULT_NAMESPACE="admin"
    export VAULT_TOKEN=hvc.xxxxxxxxxxxxxxxxxxxxxxxxx

## To export the secrets:
    ./medusa export secret > filename --format="json" --insecure

Command above will create file with name `filename` and write in it all secrets from `secret` path on `JSON` format.

## To import the secrets:
    ./medusa import secrets ./filename --insecure
