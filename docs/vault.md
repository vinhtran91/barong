# Vault configuration

## Introduction

This document describe how to create vault tokens in order to configure **barong-rails** to be able **to encrypt** and **to decrypt** api_key secrets, **to renew** token and **to manage** totp, to configure **barong-authz** to be able **to decrypt** api_key secrets, **to renew** token.

## Connect to vault

You can validate it works running the following command:
```bash
$ vault status

Type: shamir
Sealed: false
Key Shares: 1
Key Threshold: 1
Unseal Progress: 0
Unseal Nonce: 
Version: 1.3.4
Cluster Name: vault-cluster-650930cf
Cluster ID: 9f40327d-ec71-9655-b728-7588ce47d0b4

High-Availability Enabled: false
```
## Create ACL groups

### Create the following policy files

**barong-rails.hcl**

```bash
# Access system health status
path "sys/health" {
  capabilities = ["read", "list"]
}

# Manage the transit secrets engine
path "transit/keys/*" {
  capabilities = [ "create", "read", "list" ]
}

# Encrypt engines secrets
path "transit/encrypt/opendax_apikeys_*" {
  capabilities = [ "create", "read", "update" ]
}

# Decrypt engines secrets
path "transit/decrypt/opendax_apikeys_*" {
  capabilities = [ "create", "read", "update" ]
}

# Renew tokens
path "auth/token/renew" {
  capabilities = [ "update" ]
}

# Lookup tokens
path "auth/token/lookup" {
  capabilities = [ "update" ]
}

# Generate otp code
path "totp/keys/opendax_*" {
  capabilities = ["create", "read"]
}

# Verify an otp code
path "totp/code/opendax_*" {
  capabilities = ["update"]
}
```

**barong-authz.hcl**

```bash
# Access system health status
path "sys/health" {
  capabilities = ["read", "list"]
}

# Manage the transit secrets engine
path "transit/keys/*" {
  capabilities = [ "create", "read", "list" ]
}

# Decrypt engines secrets
path "transit/decrypt/opendax_apikeys_*" {
  capabilities = [ "create", "read", "update" ]
}

# Renew tokens
path "auth/token/renew" {
  capabilities = [ "update" ]
}

# Lookup tokens
path "auth/token/lookup" {
  capabilities = [ "update" ]
}
```

### Create the ACL groups in vault

```bash
vault policy write barong-rails barong-rails.hcl
vault policy write barong-authz barong-authz.hcl
```

### Create applications tokens

```bash
vault token create -policy=barong-rails -period=240h
vault token create -policy=barong-authz -period=240h
```

## Configure Barong

Set those variables according to your deployment:
```bash
export VAULT_URL=http://127.0.0.1:8200
export VAULT_TOKEN=s.jyH1vmrOmkZ0FZZ0NZtgRenS
export VAULT_APP_NAME=opendax
```