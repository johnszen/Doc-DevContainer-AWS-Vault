# Using `aws sts get-session-token` to create aws session
- [Using `aws sts get-session-token` to create aws session](#using-aws-sts-get-session-token-to-create-aws-session)
  - [TL;DR](#tldr)
  - [Steps](#steps)
    - [Get an aws session](#get-an-aws-session)
    - [Use assume-role in Terraform](#use-assume-role-in-terraform)
## TL;DR
- Create an aws session `aws sts get-session-token`
  - save the session in `~/.aws/credentials`
- in terraform aws provider, use the credentials with credentials and set assume_role

## Steps

### Get an aws session
```bash
# get session token
aws sts get-session-token --serial-number arn:aws:iam::464880672835:mfa/john.zen --token-code 000000
# {
#     "Credentials": {
#         "AccessKeyId": "######",
#         "SecretAccessKey": "$$$$$$$",
#         "SessionToken": "*************",
#         "Expiration": "2022-06-24T19:19:03Z"
#     }
# }

```

Create `~/.aws/tmp_credentials` file and add output to it; for devContainer with USER root, this is: `/root/.aws/tmp_credentials`
```ini
[mfa]
output = json
region = ap-southeast-2
aws_access_key_id = ######
aws_secret_access_key = $$$$$$$
aws_session_token = *************
```
### Use assume-role in Terraform
In terraform `provider.tf`, set the `shared_credentials_file` file if required; default is `~/.aws/credentials`.

```json
provider "aws" {
  region                   = local.region
  allowed_account_ids      = local.allowed_account_ids
  shared_credentials_files = ["/root/.aws/tmp_credentials"]

  assume_role {
    role_arn     = "arn:aws:iam::123456789012:role/ROLE_NAME"
    session_name = "A_SESSION_NAME"
  }
}
```
