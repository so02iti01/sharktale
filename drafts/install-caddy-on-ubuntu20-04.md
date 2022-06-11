---
title: ubuntu|用 package 安装 caddy
date: 2022-06-11 17:04:17
tags:
- caddy

---



> E: The repository 'https://dl.cloudsmith.io/public/caddy/xcaddy/deb/debian any-version InRelease' is not signed.

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```



Try running this:

```bash
gpg --show-keys --fingerprint --with-subkey-fingerprint /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```

You should see the same output, notice the subkey has `ABA1 F9B8 875A 6661` at the end which matches the fingerprint in the error you got.

```bash
pub   rsa4096 2016-04-01 [SC]
      6576 0C51 EDEA 2017 CEA2  CA15 155B 6D79 CA56 EA34
uid                      Caddy Web Server <contact@caddyserver.com>
sub   rsa4096 2020-12-29 [S] [expires: 2025-12-28]
      2F5C 3BE9 886A CD29 1329  9EFB ABA1 F9B8 875A 6661
```

