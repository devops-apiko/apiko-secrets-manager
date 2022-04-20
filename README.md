# apiko-secrets-manager

This package encrypts and decrypts files using the GPG program and keys stored in AWS Parameter Store.

The package was created and tested on macOS and hasn't tested on other platforms yet.

## What problem does this package solve?

Sometimes you may need to pass many environment variables into your CI/CD pipelines (like 10-20 variables in each environment). Also, it may be a problem to figure out with `appsettings.json` files in .Net (GitHub doesn't allow storing files in Action Secrets).

This package allows you to store secret variables in encrypted format inside your Git repository and retrieve them when you need them.

## Requirements

To use this package, you need pre-install the following utilities:

- [AWS CLI](https://aws.amazon.com/cli/) - a tool for managing AWS services;

- [GPG](https://gnupg.org/index.html) - an implementation of the OpenPGP standard for data encryption;

- [jq](https://stedolan.github.io/jq/) - command-line JSON processor.

## Installation

```bash
npm install -g apiko-secrets-manager
```

## Configuration

### 1. Create a new item (or items) in the Parameter Store

- Use the next pattern for the parameter name:

`/<project_name>/<environment>/secrets_encryption_key`

- `project_name` - a name of your project

- `environment` - name of the environment (development, production, etc)

Use a `Standard` tier and a `String` type.

- Use a randomly generated string for a parameter value

### 2. Create a new user

- Create a new IAM policy for a user (it's easier to do with the visual editor)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ssm:GetParameter",
      "Resource": "arn:aws:ssm:<region>:<aws_account_id>:parameter/<project_name>/<environment>/secrets_encryption_key"
    }
  ]
}
```

- Create a new IAM user and attach the new policy to it

- This user needs only programmatic access

### 3. Configure AWS profile locally

- Run the command for configuring the profile:

```bash
aws --profile <profile_name> configure
```

- Enter credentials from the previous step and the region where you store parameters

### 4. Configure `secrets_manager`

The package is working with the configuration file `.secrets_manager_config`.

To create this file you should run the following command:

```bash
secrets_manager config
```

You need to specify the `<project_name>` and `<profile_name>` from previous steps during the configuration.

## How to use it?

### Files encryption

To encrypt a file, use the next command:

```bash
secrets_manager encrypt <file_name> <environment>
```

The `environment` is optional and is equal to `development` if not specified.

As a result of this command, you'll get an encrypted file with a `.gpg` extension. You can add this file to your Git repository.

### Files decryption

To decrypt a `.gpg` file, use the next command:

```bash
secrets_manager decrypt <file_name>.gpg <environment>
```

## Future work

- [ ] Test on other platforms
  - [x] macOS
  - [ ] Linux
  - [ ] Windows
- [ ] Improve logging
