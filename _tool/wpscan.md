---
layout: post_general
title: wpscan
tags: cheatsheet wpscan wordpress cms plugin
published: 2022-06-13
updated: 
---

## 基本

プラグイン、テーマ、timthumb、ユーザ、コンフィグバックアップ、DBエクスポートを列挙する
```
wpscan --url <target_url> -e ap,at,tt,u,cb,dbe -o wpscan.out
```

ユーザ名を固定して総当たり攻撃
```
wpscan --url <target_url> -usernames <username> -P <password_list>
```

### -t オプション

実行するスレッドの数。デフォルトは5。

### &#45;&#45;stealthyオプション

```--random-user-agent --detection-mode passive --plugins-version-detection passive```のエイリアス。

## WordPress Vulnerability Database API

```-e vp,vt```で脆弱なプラグイン・テーマの列挙ができるが、それをするには[WordPress Vulnerability Database API](https://wpscan.com/api)にて取得したAPIキーをコマンドライン引数にいれて、脆弱性情報を取得できる状態となっている必要がある。

## エラー対処

### Scan Aborted: The remote website is up, but does not seem to be running WordPress.が出たとき

```--url```の指定を変えるとうまくいくかもしれない。```https://example.com/wordpress/```みたいに```wordpress```まで含めてみる


## References

[https://github.com/wpscanteam/wpscan/wiki/WPScan-User-Documentation](https://github.com/wpscanteam/wpscan/wiki/WPScan-User-Documentation)
