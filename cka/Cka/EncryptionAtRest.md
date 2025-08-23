# Encryption at Rest

## Content

- API server uses an `EncryptionConfiguration` to encrypt Secrets in etcd.
- Providers include `aescbc`, `kms` plugins.
- Rotate keys by adding new entries and re-encrypting data.
