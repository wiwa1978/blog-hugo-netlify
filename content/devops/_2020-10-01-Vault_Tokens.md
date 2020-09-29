---
title: Hashicorp Vault - Tokens
date: 2020-09-01T16:19:50+01:00
draft: true
categories:
  - Cloud Native
  - All
tags:
  - Vault
---

### Get Token capabilities

```bash
➜  Programming vault login -method=userpass username=wim
Password (will be hidden): 
WARNING! The VAULT_TOKEN environment variable is set! This takes precedence
over the value set by this command. To use the value set by this command,
unset the VAULT_TOKEN environment variable or set it to the token displayed
below.

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.N4vTKAZzE9hPkFd6IwYabE1Z
token_accessor         sfsh3c11xtqrhe31X76RUABI
token_duration         768h
token_renewable        true
token_policies         ["default" "policy_secrets"]
identity_policies      []
policies               ["default" "policy_secrets"]
token_meta_username    wim
```
Pay particular attention to the message `The VAULT_TOKEN environment variable is set! This takes precedence
over the value set by this command.`. Therefore, let's ensure the VAULT_TOKEN is set correctly:

```bash
➜  Programming export VAULT_TOKEN=s.N4vTKAZzE9hPkFd6IwYabE1Z  
```

```bash
➜  Programming vault token capabilities sys/            
deny
```

```bash
➜  Programming vault token capabilities cubbyhole/          
create, delete, list, read, update
```

Now, let's say we want to check the capabilities for our root user. We can then also give the root token as an argument:

```
➜  Programming vault login s.eZdMr5KDFOVRlsuJurTiGKc0                                                                        
WARNING! The VAULT_TOKEN environment variable is set! This takes precedence
over the value set by this command. To use the value set by this command,
unset the VAULT_TOKEN environment variable or set it to the token displayed
below.

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.eZdMr5KDFOVRlsuJurTiGKc0
token_accessor       gqzFQ9bQdRNfYBsndHkvgFg9
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
➜  Programming vault token capabilities s.eZdMr5KDFOVRlsuJurTiGKc0 sys/ 
root
```




