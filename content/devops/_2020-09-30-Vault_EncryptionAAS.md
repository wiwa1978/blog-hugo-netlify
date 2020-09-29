---
title: Hashicorp Vault - Encryption as a Service
date: 2020-09-05T16:19:50+01:00
draft: true
categories:
  - Cloud Native
  - All
tags:
  - Vault
---
### Enable the secrets engine

```bash
➜  blog-hugo-netlify git:(master) ✗ vault secrets enable -path=transit transit
Success! Enabled the transit secrets engine at: transit/
```

![UI](/images/2020-09-30-1.png)

### Encrypting text

We will use the CLI to encrypt the following text `This is enceypted text`. Note that this string must be first encoded to Base64 format. We can use the website https://www.base64encode.org to encode this string to Base64.

![UI](/images/2020-09-30-2.png)

```bash
➜  blog-hugo-netlify git:(master) ✗ vault write transit/encrypt/key1 plaintext=VGhpcyBpcyBlbmNyeXB0ZWQgdGV4dA==
Key            Value
---            -----
ciphertext     vault:v1:TgmsTfprF3tdtLfNDOsr9iXQZKEWQRkGV+62obTOUG0pBKplmIOFsYrkTf3SCZ9FHBs=
key_version    1
```
In the UI, you will notice that a key `key1` has been added:

![UI](/images/2020-09-30-3.png)

However note that Vault will not store the encrypted key itself, it's intended for data in transit, hence the name.

### Decrypting text

```bash
➜  blog-hugo-netlify git:(master) ✗ vault write transit/decrypt/key1 ciphertext=vault:v1:TgmsTfprF3tdtLfNDOsr9iXQZKEWQRkGV+62obTOUG0pBKplmIOFsYrkTf3SCZ9FHBs=
Key          Value
---          -----
plaintext    VGhpcyBpcyBlbmNyeXB0ZWQgdGV4dA==
```
Here you will get back the unencrypted value. Again, note that this is Base64 encoded. We can use the website https://www.base64decode.org/ to decode this string.

![UI](/images/2020-09-30-4.png)

Et voila, we get back our decrypted text `This is encrypted text`