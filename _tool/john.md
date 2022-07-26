---
layout: post_general
title: john the ripper - Rule Based Atack
tags: john jtr crack password hash 'rule based attack'
published: 2022-07-26:w
updated:
---

TryHackMeの[Crack The Hash Level 2](https://tryhackme.com/room/crackthehashlevel2)でJohnのRule Based Attackを使う機会があったのでまとめる。

## Rule Based Attack

基本的にJohn the RipperやHashcatなどパスワードクラッキングに使用されるツールでは、パスワードに使用されることが多い単語がリスト化されたもの（いわゆる「辞書」）を利用するが、たいていは辞書にそのまま載っているパスワードは当たらない。なので次にどんな単語を用意しようと考えたときに、辞書に記載されている単語に数字を足してみたり、大文字・小文字変換をしてみたり、Leet表記を使用してみたりと、辞書の単語に一定のルールをもって変換を加える。

しかしながらディスクスペースを圧迫してしまうことや汎用性の観点から、辞書内の単語をルールで変換して生成した大量の単語を静的なファイルとして用意するのはよろしくない。そこで辞書内の単語を一定の規則で変化させるルールを、JohnやHashcat実行時に与えてやるのがよい。

JohnでRule Based Attackを使用する際には、以下のように`--rules`オプションを与える。
```bash
$ john <FILENAME> --format=<FORMAT> --wordlist=<DICIONARY> --rules=NameOfRule
```
ここに登場する`NameOfRule`は自身でJohnの設定ファイルに定義したルールの名称。

## ルールを作成する

Johnの設定ファイルを編集して、外部の設定ファイルをIncludeする

```bash
$ vim /etc/john/john.conf
...
.include '/path/to/your/conf'
...
```

ルールの本体を書く

※以下で記載している`c`は先頭だけ大文字にするコマンド、`u`はすべて大文字にするコマンド、`l`はすべて小文字にするコマンド。コマンドの一覧はこのあとの「ルールに使用できるコマンド一覧」を参照

```bash
$ vim /path/to/your/conf
...
[List.Rules:NameOfRule1]  <-  ここのルール名をコマンドの--rulesオプションに指定する
c
[List.Rules:NameOfRule2]
u
[List.Rules:NameOfRule3]
l
...
```

ひとつのルールの中に複数のコマンドを使用することができるが、改行を入れたパターンと入れないパターンでは生成される単語のリストが異なる点に注意。

```bash
$ vim /path/to/your/conf
...
[List.Rules:command_for_eachline]
# 各行に分けて書く
c
l
u

[List.Rules:command_in_oneline]
# 1行で書く
clu
...

$ cat example.dict
example

$ john --wordlist=example.dict --rules=command_for_eachline --stdout
Using default input encoding: UTF-8
Example
EXAMPLE
example
3p 0:00:00:00 100.00% (2022-07-26 09:46) 75.00p/s example

$ john --wordlist=example.dict --rules=command_in_oneline --stdout
Using default input encoding: UTF-8
EXAMPLE
1p 0:00:00:00 100.00% (2022-07-26 09:47) 33.33p/s EXAMPLE

```

## ルールに使用できるコマンド一覧

### 文字を変換するコマンド

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`:`|何もしない|`example`|`:`|`example`|
|`c`|最初を大文字にする（capitalize）|`example`|`c`|`Example`|
|`C`|最初を小文字に、他は大文字にする|`example`|`C`|`eXAMPLE`|
|`V`|母音は小文字に、子音は大文字に|`example`|`V`|`eXaMPLe`|
|`R`|キーボードの右隣のキーに変換する|`exampl3`|`R`|`rcs,[;4`|
|`L`|キーボードの左隣のキーに変換する|`exampl3`|`L`|`wzanok2`|
|`l`|すべて小文字にする（lowercase）|`EXAMPLE`|`l`|`example`|
|`u`|すべて大文字にする（uppercase）|`example`|`u`|`EXAMPLE`|
|`t`|すべての小文字は大文字に、大文字は小文字にする（toggle）|`ExAmPlE`|`t`|`eXaMpLe`|
|`TN`|`N`番目の文字について、小文字は大文字に、大文字は小文字にする（toggle N）|`ExAmPlE`|`T3`|`ExAMPlE`|
|`WN`|`N`番目の文字について、Shiftを押しながら入力したときの文字に変換する|`exampl3`|`W6`|`exampl#`|
|`S`|すべての文字について、Shiftを押しながら入力したときの文字に変換する|`exampl3`|`S`|`EXAMPL#`|
|`r`|文字列を逆にする（reverse）|`example`|`W6`|`elpmaxe`|
|`{`|文字列を左にローテートする|`example`|`{`|`xamplee`|
|`}`|文字列を右にローテートする|`example`|`{`|`eexampl`|


### 文字列に追加/削除するコマンド

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`d`|文字列を複製する（duplicate）|`example`|`d`|`exampleexample`|
|`pN`|文字列を`N`回複製する（duplicate N）|`example`|`p3`|`exampleexampleexampleexample`|
|`zN`|最初の文字を`N`回複製する|`example`|`z3`|`eeeexample`|
|`q`|すべての文字を複製する|`example`|`q`|`eexxaammppllee`|
|`f`|逆にした文字列を追加する（reflect）|`example`|`f`|`exampleelpmaxe`|
|`$`|文字を末尾に追加する|`example`|`$1`|`example1`|
|`^`|文字を先頭に追加する|`example`|`^1`|`1example`|
|`[`|最初の文字を削除する|`example`|`[`|`xample`|
|`]`|最後の文字を削除する|`example`|`]`|`exampl`|
|`'N`|`N`番目以降の文字を削除する|`example`|`'3`|`exa`|
|`DN`|`N`番目の文字を削除する（Delete）|`example`|`D3`|`exaple`|
|`xNM`|`N`番目から始まる`M`文字を抽出する（Extract）|`example`|`x14`|`xamp`|
|`ONM`|`N`番目から始まる`M`文字を削除する（Omit）|`example`|`x14`|`ele`|
|`iNX`|`N`番目に文字`X`を挿入する（insert）|`example`|`i2@`|`ex@ample`|
|`oNX`|`N`番目を文字`X`で置換する（overwrite）|`example`|`o2@`|`ex@mple`|
|`sXY`|すべての文字`X`を文字`Y`で置換する|`example`|`se$`|`3xampl3`|
|`@X`|すべての文字`X`を削除する|`example`|`@e`|`xampl`|

### 英単語の操作をするコマンド

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`p`|単語に複数形のsをつける（pluralize）|`crack`|`p`|`cracks`|
|`P`|単語に過去形のd/edをつける|`crack`|`P`|`cracked`|
|`I`|単語を名詞化するingをつける|`crack`|`I`|`cracking`|

### ルールを適用する条件を設定するコマンド - 文字列の長さ編

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`<L`|長さ`L`より短い文字列にルールを適用する|`example`|`c<9`|`EXAMPLE`|
|`>L`|長さ`L`より長い文字列にルールを適用する|`example`|c>3|`EXAMPLE`|
|`_L`|長さ`L`の文字列にルールを適用する|`example`|`c_7`|`EXAMPLE`|

### ルールを適用する条件を設定するコマンド - その他編

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`-:`||`example`||`EXAMPLE`|
|`-c`||`example`||`EXAMPLE`|
|`-8`||`example`||`EXAMPLE`|
|`-s`||`example`||`EXAMPLE`|
|`-p`||`example`||`EXAMPLE`|
|`-u`||`example`||`EXAMPLE`|
|`-U`||`example`||`EXAMPLE`|
|`->N`||`example`||`EXAMPLE`|
|`-<N`||`example`||`EXAMPLE`|

### 単語を保存したり、保存した単語を活用するコマンド

|コマンド|効果|入力|ルールの例|生成される単語|
|-|-|-|-|-|
|`M`|左からコマンドを適用していったその時点での単語を保存する|`example`|`M`|`example`|
|`4`|`M`で保存した単語を末尾に追加する|`example`|`uM4`|`exampleEXAMPLE`|
|`6`|`M`で保存した単語を先頭に追加する|`example`|`uM6`|`EXAMPLEexample`|
|`XNLI`|保存した単語の`N`番目から長さ`L`を取り出し`I`番目に挿入する|`example`|`MuX253`|`EXAampleMPLE`|

## 作成したルールからどんな単語が生成されるのかを確認する

```bash
$ john --wordlist=<DICIONARY> --rules=NameOfRule --stdout
```

## References

- [John the Ripper - wordlist rules syntax](https://www.openwall.com/john/doc/RULES.shtml)
- [Comprehensive Guide to John the Ripper. Part 5: Rule-based attack - Ethical hacking and penetration testing](https://miloserdov.org/?p=5477#51)
