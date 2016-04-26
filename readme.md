![](http://s3.amazonaws.com/faraday-assets/wilsons-banner.svg)

# Secretfile

A simple, standard, open pattern for safely using secrets from secure stores like Vault in your applications.

## Background

**Applications need secrets**—API keys for integrated services, database credentials, etc. With any luck, you likely stopped including these secrets in your codebase long ago. If you follow the [twelve-factor app][12factor] pattern, you probably store passwords or other secrets in the environment and safely reference these environment variables in your app code.

**But how do the secrets get put in the environment?** In simple cases, this can be part of a deploy process. But once your application deployment becomes more complex, it's much easier to store passwords in a central, secure store such as Hashicorp's [Vault][vault] or Square's [Keywhiz][keywhiz].

**Now you have a new problem**: should you burden all of your applications with Vault-specific code? What if you switch backing stores, or use different methods in your development environment?

## The Secretfile

The solution is to relay secrets from the store to the application in a standard way. Introducing the Secretfile:

```
# Secretfile
AWS_ACCESS_KEY_ID     secrets/services/aws:id
AWS_ACCESS_KEY_SECRET secrets/services/aws:secret
PG_USERNAME           postgresql/$VAULT_ENV/creds/readonly:username
PG_PASSWORD           postgresql/$VAULT_ENV/creds/readonly:password
```

In the `AWS` examples, we're specifying that the secrets named on the left should be drawn from the corresponding Vault path on the right. In the `PG` examples, we show that Secretfile consumers will interpolate environment variables (`$VAR` or `${VAR}`) as needed.

## Installation

1. Create a file called `Secretfile` in your application directory following the format above.
2. Make sure your secret store is set up in your environment. For example, Vault needs the `VAULT_ADDR` environment variable and a token in either `VAULT_TOKEN` or `~/.vault-token`.
3. Make the specified secrets available to your application by using either a wrapper tool or a Secretfile client library.

### Wrapper tools

* **[credentials-to-env][c2e]**: Keep using 12factor-style environment variable references in your application to access secrets by translating the Secretfile into your environment. Written in highly portable Rust with static binaries available.

```shell
$ credentials-to-env myprogram arg1 arg2
```

### Libraries

* **[credentials][credentials]**: Load secrets from your Secretfile into your Rust application.

```rust
credentials::var("TOP_SECRET").unwrap();
```

* **[SecretGarden][sg]**: Load secrets from your Secretfile into your Ruby application. Includes support for Rails.

```ruby
SecretGarden.add_backend :vault
s3 = AWS::S3.new access_key_id: SecretGarden.fetch('AWS_ACCESS_KEY_ID')
```

### Credits and corporate support

[![Faraday logo](https://s3.amazonaws.com/faraday-assets/files/img/logo.svg)](http://faraday.io)

[Faraday](http://faraday.io) leads the Secretfile project and sponsors [credentials-to-env][c2e], [credentials][credentials], and [SecretGarden][sg].

[12factor]: http://12factor.net/
[vault]: https://www.vaultproject.io/
[keywhiz]: https://square.github.io/keywhiz/
[c2e]: https://github.com/faradayio/credentials_to_env
[credentials]: https://github.com/emk/credentials
[sg]: https://github.com/dkastner/secret_garden