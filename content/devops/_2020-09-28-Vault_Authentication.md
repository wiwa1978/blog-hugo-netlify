---
title: Hashicorp Vault - Authentication and Authorization
date: 2020-09-02T16:19:50+01:00
draft: true
categories:
  - Cloud Native
  - All
tags:
  - Hashicorp
  - Vault
---

### Auth methods

```
➜  blog-hugo-netlify git:(master) ✗ vault auth list  
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_40a31392    token based credentials
```

This explains why -so far- we have only been able to enter using the token parameter.

### Enabling auth methods

```
➜  blog-hugo-netlify git:(master) ✗ vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
```

```
➜  blog-hugo-netlify git:(master) ✗ vault auth list           
Path         Type        Accessor                  Description
----         ----        --------                  -----------
token/       token       auth_token_40a31392       token based credentials
userpass/    userpass    auth_userpass_fec0ad5e    n/a
```

![UI](/images/2020-09-28-1.png)

![UI](/images/2020-09-28-2.png)

### Add users

``` 
➜  blog-hugo-netlify git:(master) ✗ vault write auth/userpass/users/wim password=secret
Success! Data written to: auth/userpass/users/wim
```

### List users

```
➜  blog-hugo-netlify git:(master) ✗ vault list auth/userpass/users                                           
Keys
----
wim
```

### Login

```
➜  blog-hugo-netlify git:(master) ✗ vault login -method=userpass username=wim
Password (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.H8YyCCPX3BXc2DqLA9De8tEM
token_accessor         dGT6jiDQWcf7cIPc7StAHOu2
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    wim
```

### Token lookup

###### Current token
```
➜  blog-hugo-netlify git:(master) ✗ vault token lookup
Key                 Value
---                 -----
accessor            tsXnk0xKZsoA2b6kNEon7yIy
creation_time       1599993795
creation_ttl        0s
display_name        root
entity_id           n/a
expire_time         <nil>
explicit_max_ttl    0s
id                  s.5EdP7fXFxKJcDdw6snqty93K
meta                <nil>
num_uses            0
orphan              true
path                auth/token/root
policies            [root]
ttl                 0s
type                service
```

###### Change token

```
➜  blog-hugo-netlify git:(master) ✗ vault login -method=userpass username=wim
Password (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.hwzvWIgDPaisSPYVgnHXUCND
token_accessor         bhX4USUCIkSP714Vo19MtyvU
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    wim
```

```
➜  blog-hugo-netlify git:(master) ✗ vault token lookup                       
Key                 Value
---                 -----
accessor            bhX4USUCIkSP714Vo19MtyvU
creation_time       1600006687
creation_ttl        768h
display_name        userpass-wim
entity_id           3f165779-5f7c-01a4-9316-3de7962ce11e
expire_time         2020-10-15T16:18:07.936905+02:00
explicit_max_ttl    0s
id                  s.hwzvWIgDPaisSPYVgnHXUCND
issue_time          2020-09-13T16:18:07.936922+02:00
meta                map[username:wim]
num_uses            0
orphan              true
path                auth/userpass/login/wim
policies            [default]
renewable           true
ttl                 767h59m42s
type                service
```

### Reset password

```
➜  blog-hugo-netlify git:(master) ✗ vault write auth/userpass/users/wim/password password=secretpassword
Error writing data to auth/userpass/users/wim/password: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/auth/userpass/users/wim/password
Code: 403. Errors:

* 1 error occurred:
        * permission denied
```
Look at the error: the current user has no permission to reset its own password. Therefore we need to go back to the root token.

```
➜  blog-hugo-netlify git:(master) ✗ vault login                                                         
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.5EdP7fXFxKJcDdw6snqty93K
token_accessor       tsXnk0xKZsoA2b6kNEon7yIy
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```
Next, let's try to change the password once again:

```
➜  blog-hugo-netlify git:(master) ✗ vault write auth/userpass/users/wim/password password=secretpassword
Success! Data written to: auth/userpass/users/wim/password
```
And as you can see, it succeeded now.

### Delete user

```
➜  blog-hugo-netlify git:(master) ✗ vault delete auth/userpass/users/wim                                
Success! Data deleted (if it existed) at: auth/userpass/users/wim
```
We can verify using:

```
➜  blog-hugo-netlify git:(master) ✗ vault list auth/userpass/users  
No value found at auth/userpass/users/
```

### Remove authentication method

```
➜  blog-hugo-netlify git:(master) ✗ vault auth disable userpass   
Success! Disabled the auth method (if it existed) at: userpass/
```


### Policies
Next, let's create a user again in order to experiment a bit with `policies`.

```bash
➜  blog-hugo-netlify git:(master) ✗ vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
➜  blog-hugo-netlify git:(master) ✗ vault write auth/userpass/users/wim password=secret                 
Success! Data written to: auth/userpass/users/wim
➜  blog-hugo-netlify git:(master) ✗ vault login -method=userpass username=wim password=secret
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.o8IGLcJaiJNg8TRFwVikUv13
token_accessor         QZHKiotMtySgdnVI8oZFWmAt
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    wim
```
You will notice here that we got assigned a `default` policy which is very restrictive.

```bash
➜  blog-hugo-netlify git:(master) ✗ vault secrets list                                       
Error listing secrets engines: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/mounts
Code: 403. Errors:

* 1 error occurred:
        * permission denied


➜  blog-hugo-netlify git:(master) ✗  vault policy list
Error listing policies: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/policies/acl?list=true
Code: 403. Errors:

* 1 error occurred:
        * permission denied
```
The above snippet shows us that we don't have the permission to execute commands like `vault secrets list` or `vault policy list`. We can fix this through creating a new policy, either through the UI or through the CLI. As the UI is pretty self-explanatory, we will do this through the CLI.

```bash
➜  Programming vault login                                              
Token (will be hidden): 
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
```


```bash
➜  Programming cat policy_secrets_list.hcl 
path "sys/mounts" {
  capabilities = ["read"]
}
➜  Programming vault policy write policy_secrets policy_secrets_list.hcl
Success! Uploaded policy: policy_secrets
```
![UI](/images/2020-09-28-4.png)
![UI](/images/2020-09-28-3.png)

Let's login with our normal (non-root) user:

```bash
➜  Programming vault login -method=userpass username=wim password=secret
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.iRt1pB9MeMfRHdSAfk8czhwd
token_accessor         lCdWtfBlqmS4Kllnh6I1Zotc
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    wim
➜  Programming vault secrets list                                       
Error listing secrets engines: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/mounts
Code: 403. Errors:

* 1 error occurred:
        * permission denied
```
You will see that nothing was solved. The reason is that we need to assign the policy to a user. Let's do this through the user interface for a second. Go to `Access > Auth Methods > userpass > [youruser] > edit user` and add the policy we just created (policy_secrets) to the token as follows:

![UI](/images/2020-09-28-5.png)

Let's try it again now:

```bash
➜  Programming vault secrets list
Error listing secrets engines: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/mounts
Code: 403. Errors:

* 1 error occurred:
        * permission denied
```
Still doesn't work! Reason is that the policy is added to the token but the token needs to be renewed for the policy to take effect. So let's do that:
```bash
➜  Programming vault login -method=userpass username=wim password=secret
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.LO49eU6CzemuNJ7IO9mZxa4k
token_accessor         OIGMvxgJQvSVPcbdKZs1OTHx
token_duration         768h
token_renewable        true
token_policies         ["default" "policy_secrets"]
identity_policies      []
policies               ["default" "policy_secrets"]
token_meta_username    wim
```
Note that our policy is now mentioned in the policies array. And as expected, we can now run the command successfully.

```bash
➜  Programming vault secrets list                                       
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_c5accb9e    per-token private secret storage
identity/     identity     identity_7e0d4273     identity store
secret/       kv           kv_dd7676f0           key/value secret storage
sys/          system       system_7e344b71       system endpoints used for control, policy and debugging
transit/      transit      transit_a98ae819      n/a
```










vault policy list

vault policy write [policy] [policyfile.hcl]

vault write sys/policy/[policy] policy=[policy_file.hcl]


vault delete sys/policy/[policy]

{{identity.entity.id}}