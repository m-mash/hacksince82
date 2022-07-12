---
layout: post_general
title: google
tags: google "google dork" osint "search engine"
published: 2022-07-12
updated:
---

普段の調べものだけでなくOSINTにも使えるGoogle検索方法。

## Operator

|Operator|説明|使用例|
|---|---|---|
|`site:`|指定したサイト内のページを検索する|`site:example.com`|
|`intitle:`|指定したキーワードをタイトルに含むページを結果に表示する|`intitle:admin`|
|`inurl:`|指定したキーワードをタイトルに含むページを結果に表示する|`inurl:wp-admin`|
|`filetype:`|指定したファイルの種類に絞って結果に表示する|`filetype:pdf example.com`|
|`*`|ワイルドカード|`how to *`|
|`cache:`|Googleが過去にキャッシュしたバージョンのページを表示する|`cache:example.com`|
|`inanchor:`|指定したキーワードをアンカーテキストに含むページを結果に表示|`inanchor:keyword`|
|`intext:`|指定したキーワードをページテキストに含むページを結果に表示|`intext:keyword`|
|`related:`|指定したサイトと関連するサイトを結果に表示する|`related:`|
|`around(N)`|キーワードがN語以内に近接しているものを結果に表示する|`linux around(5) tutorial`|
|`AND`|AND検索|`offensice AND security`|
|`OR`|OR検索|`tutorial OR course`|
|`()`|Operatorの順序をコントロール|`offensice security AND (tutorial OR course)`|
|`-`|除外検索|`ios -cisco`|

## References
- [Google Refine Web Searches](https://support.google.com/websearch/answer/2466433)
- [Google Hacking Database - Exploit Database](https://www.exploit-db.com/google-hacking-database)