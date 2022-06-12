---
layout: 
title: wpscan
tags: cheatsheet wpscan wordpress cms web xmlrpc plugin php
---

# wpscan

脆弱なプラグイン、ユーザ、脆弱なテーマ、コンフィグバックアップ、DBエクスポートを列挙する
```
wpscan --url <target_url> -e vp,u,vt,cb,dbe -o wpscan.out
```

ユーザ名を固定して総当たり攻撃
```
wpscan --url <target_url> -usernames <username> -P /usr/share/wordlists/rockyou.txt --max-threads 10
```

## references

https://github.com/wpscanteam/wpscan/wiki/WPScan-User-Documentation
