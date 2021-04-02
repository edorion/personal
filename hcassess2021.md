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
1.
```
$ vault secrets enable aws
Success! Enabled the aws secrets engine at: aws/
```
2.
Using a user I alreay have setup for AWS SDK intergration testing to my own AWS account which isnt the root user (as thats what the docs say is required) 
```
$ vault write aws/config/root \
>     access_key=<some AWS IAM user access key> \
>     secret_key=<some AWS IAM user secret ky> \
>     region=ap-southeast-2
Success! Data written to: aws/config/root
```

3.
Configure a Vault role that maps to a set of permissions in AWS as well as an AWS credential type.
my-role
```
$ vault write aws/roles/my-role \
>     credential_type=iam_user \
>     policy_document=-<<EOF
> {
>   "Version": "2012-10-17",
>   "Statement": [
>     {
>       "Effect": "Allow",
>       "Action": "ec2:*",
>       "Resource": "*"
>     }
>   ]
> }
> EOF
Success! Data written to: aws/roles/my-role
```
4. 
Verify setup
```
$ vault read aws/creds/my-role
Key                Value
---                -----
lease_id           aws/creds/my-role/bOdUeyZEZVte24Pqp6utKx7B
lease_duration     768h
lease_renewable    true
access_key         AKIAVVBTO3TGPLQCTLYC
secret_key         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
security_token     <nil>
```
###### Translate CLI command to API command
Initial thoughts - I know (from previous experience / notes / verified here https://learn.hashicorp.com/tutorials/vault/getting-started-apis) that there is a vault command flag -output-curl-string that will output what the vault app is actually saying to the vault API

```
$ vault write -output-curl-string aws/roles/my-role credential_type=iam_user     policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    }
  ]
}
EOF

curl -X PUT -H "X-Vault-Request: true" -H "X-Vault-Token: $(vault print token)" -d '{"credential_type":"iam_user","policy_document":"{\n  \"Version\": \"2012-10-17\",\n  \"Statement\": [\n    {\n      \"Effect\": \"Allow\",\n      \"Action\": \"ec2:*\",\n      \"Resource\": \"*\"\n    }\n  ]\n}\n"}' http://127.0.0.1:8200/v1/aws/roles/my-role
```






Option 3 - Policies
Initial thoughts - policy only allows access to /sys/* which i would expect to be for system backend process i.e. audit policies config etc not for interacting with a secrets engine. Simple test will be to create 2 policies, one the same as they have to make sure the actual is factual and one that will let them interact with the OpenLDAP Secrets Engine. It will also be interesting to see if they can grant them selves access to the secrets engine with their current policy and supply that to them in the response as well.

1 - Customer Response


2 - Troubleshooting


Option 4 - TLS
Initial thoughts - As a guess it will be TLS1.3 as 1 we want to be super secure with Vault and 2 anything before it is mostly deprecated or not recommended (google search result to be sure https://datatracker.ietf.org/doc/rfc8996/). Will need to verify. May also need to advise that while most current OS's will be OK, for Microsoft servers .NET needs to be patched and have some registry changes made to make them TLS1.3 compatible which could cause problems if an app is trying to get a secret from Vault and the TLS version it's trying to use isn't available

1 - Customer response


2 - Troubleshooting
