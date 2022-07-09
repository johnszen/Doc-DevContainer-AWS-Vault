# Documentation: Running `aws-vault` in `devContainer`
- [Documentation: Running `aws-vault` in `devContainer`](#documentation-running-aws-vault-in-devcontainer)
  - [Steps](#steps)
    - [set up `~/.aws/config`](#set-up-awsconfig)
    - [install `aws-vault` in dockerfile](#install-aws-vault-in-dockerfile)
    - [set environment variables in `.decontainer/devcontainer.env`](#set-environment-variables-in-decontainerdevcontainerenv)
    - [`.devcontainer/devcontainer.json`](#devcontainerdevcontainerjson)
    - [Verify](#verify)
  - [TODO](#todo)


`aws-vault` allows accessing aws with different iam-role.

`aws-vault` on desktop uses MacOS's KeyChain as backend to store aws ACCESS KEY and SECRET.

For `devContainer`, I use encrypted file as backend.

## Steps
### set up `~/.aws/config`
Set up aws config file for assume role as follow:
```apacheconf
[profile devcontainer_aws]
mfa_serial = arn:aws:iam::1234567890:mfa/john.zen

[profile team-role-a]
output = json
region = ap-southeast-2
source_profile = devcontainer_aws
role_arn = arn:aws:iam::1234567890:role/team-role-a
mfa_serial = arn:aws:iam::1234567890:mfa/my_aws_user_name
role_session_name = my_aws_user_nameInstall aws-vault in .devcontainer/Dockerfile
```
### install `aws-vault` in dockerfile
In .devcontainer/Dockerfile , add the following:
```dockerfile
#...
# select the archtype depending on your MacOS; mine is Intel, select the closest amd64
ADD https://github.com/99designs/aws-vault/releases/download/v6.2.0/aws-vault-linux-amd64 /usr/bin/aws-vault
RUN chmod +x /usr/bin/aws-vault
#...
```

### set environment variables in `.decontainer/devcontainer.env`
Add to .gitignore as this file contain sensitive information
```bash
# .gitignore
devcontainer.env
#...
```

Add the following to `.devcontainer/devcontainer.env` 
```bash
AWS_VAULT_BACKEND=file
AWS_VAULT_FILE_DIR="/root/.aws-vault"
AWS_VAULT_FILE_PASSPHRASE="mypassword"

AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=yyy
```
The first 3 lines is to set aws-vault backing vault to an encrypted file on devcontainer start up.
The next 2 line are AWS Access Key to be used to add to encrypted file above via `aws-vault`.

### `.devcontainer/devcontainer.json`
```json
{
  "name": "terraform-awscli",
  "build": {
      "dockerfile": "Dockerfile",
      "context": "."
    },
  "mounts": [
    "source=${localEnv:HOME}/.aws,target=/root/.aws,type=bind,consistency=cached"
    ],
  "runArgs": [
      "--env-file", ".devcontainer/devcontainer.env"
    ],
  "postStartCommand": "aws-vault add devcontainer_aws --env"
}
```
### Verify
Rebuild Container. Then, run the following in devContainer environment.
```bash
aws-vault exec team-role-a -- ls
Enter token for arn:aws:iam::1234567890:mfa/my_aws_user_name

# Go to a terraform directory to try:
aws-vault exec team-role-a -- terraform init
aws-vault exec team-role-a -- terraform plan
aws-vault exec team-role-a -- terraform apply
```


## TODO
- Find out way to read KeyChain to pass to devContainer instead of storing ACCESS KEY and ACCESS SECRET in `devContainer.env` file.