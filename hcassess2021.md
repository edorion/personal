### Pre assessment practice
- Read over assessment twice to make sure I understood what is required and if i have any questions
- Titled / dot pointed requirements as numbered headings to make sure everything is answered
- Noted down some initial thought from current knowledge / experiences

### Initial Exercise - AWS Secrets Engine

##### 1a - Setup AWS secrets Engine
Initial plan, run up a vault instance, then add add engine to personal AWS.
Aslo, note to self, really really need to ensure my VAULT_ADDR is pointing to the right instance and i'm off the VPN, don't want to touch my current employers production one

###### Get vault image for local testing
As I don't have any interest in persisting this outside of this assessment, i'll use a dev instance
from https://hub.docker.com/_/vault

```
:~/git/docker$ docker run --cap-add=IPC_LOCK -d --name=dev-vault -e 'VAULT_DEV_ROOT_TOKEN_ID=abc123' --network host vault
521c98af1bb49338be3ad2d792f3137a5c262bd05a58aaf5a9bc7654101f8475
:~/git/docker$ vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                abc123
token_accessor       I2CedfcAT1uQ602FUcnWAgNJ
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

###### Setup Secrets Engine (https://www.vaultproject.io/docs/secrets/aws)
$ vault secrets enable aws
Success! Enabled the aws secrets engine at: aws/









1b - Translate CLI command
Initial thoughts - I know (from previous experience / notes / verified here https://learn.hashicorp.com/tutorials/vault/getting-started-apis) that there is a vault command flag -output-curl-string that will output what the vault app is actually saying to the vault API



Option 3 - Policies
Initial thoughts - policy only allows access to /sys/* which i would expect to be for system backend process i.e. audit policies config etc not for interacting with a secrets engine. Simple test will be to create 2 policies, one the same as they have to make sure the actual is factual and one that will let them interact with the OpenLDAP Secrets Engine. It will also be interesting to see if they can grant them selves access to the secrets engine with their current policy and supply that to them in the response as well.

1 - Customer Response


2 - Troubleshooting


Option 4 - TLS
Initial thoughts - As a guess it will be TLS1.3 as 1 we want to be super secure with Vault and 2 anything before it is mostly deprecated or not recommended (google search result to be sure https://datatracker.ietf.org/doc/rfc8996/). Will need to verify. May also need to advise that while most current OS's will be OK, for Microsoft servers .NET needs to be patched and have some registry changes made to make them TLS1.3 compatible which could cause problems if an app is trying to get a secret from Vault and the TLS version it's trying to use isn't available

1 - Customer response


2 - Troubleshooting
