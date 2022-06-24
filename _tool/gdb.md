---
layout: post_general
title: gdb
tags: gdb debug pwndbg
published: 2022-06-24
updated: 
---

拡張機能の[pwndbg](https://github.com/pwndbg/pwndbg)を使用している。

## チートシート

|コマンド|説明|
|---|---|
|b `line_num` / b `func_name`|breakpointをセット|
|tbreak `line_num` / tbreak `func_name`|tenporaly breakpointをセット|
|info b | brakpointの一覧|
|disable `breakpoint_num`|指定されたbreakpointを無効化する|
|enable `breakpoint_num`|指定されたbreakpointを有効化する|
|delete `breakpoint_num`|指定されたbreakpointを削除する|
|delete|すべてのbreakpointを削除する|
|watch `val_name` |watchpointをセット|
|info watchpoints |watchpointの一覧|

## 一通りやってみる

### 1. サンプルプログラム

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>

void copy_arg(char *string)
{
    char buffer[10];
    strcpy(buffer, string);
    printf("%s\n", buffer);
    return 0;
}

int main(int argc, char **argv)
{
        printf("argv[1] is:\n");
        copy_arg(argv[1]);
}
{% endhighlight %}

コンパイル

```
# デバッグ情報を埋め込む
$ gcc -g -O0 -o overflow_no_symbol overflow.c

# デバッグ情報を埋め込まない
$ gcc -O0 -o overflow_no_symbol overflow.c

# アセンブラを出力
$ gcc -S -O0 overflow.c

# 一覧
$ ls
overflow  overflow.c  overflow_no_symbol  overflow.s
```

### 2. gdbに読み込ませる

シンボル情報がある場合(`-q`でバナーを出さない)
```
$ gdb ./overflow -q
pwndbg: loaded 198 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from ./overflow...  <= シンボルを読み込んだよ
pwndbg>
```
シンボル情報がない場合
```
$ gdb ./overflow_no_symbol -q
pwndbg: loaded 198 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from ./overflow_no_symbol...
(No debugging symbols found in ./overflow_no_symbol)  <= シンボルがないよ
pwndbg>
```

これ以降はシンボル情報がある場合(`gdb ./overflow -q`)で進める

### 3. breakpoint

breakpointを設定すると、その行の<span style="color:salmon">前</span>で一旦停止できる

line 8にbreakpointをセットする
```
pwndbg> b 8
Breakpoint 1 at 0x1168: file overflow.c, line 8.
```
関数`copy_arg()`breakpointをセットする
```
pwndbg> b copy_arg
Breakpoint 2 at 0x1155: file overflow.c, line 7.
```
関数`main`にtemporary breakpointをセットする
```
pwndbg> tbreak main
Temporary breakpoint 3 at 0x1186: file overflow.c, line 14.
```
条件がTRUEの場合に動作するbreakpointを作る
```
pwndbg> b main if <some conditions>
```

セットされたbreakpointの一覧
```
pwndbg> info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000001168 in copy_arg at overflow.c:8
2       breakpoint     keep y   0x0000000000001155 in copy_arg at overflow.c:7
3       breakpoint     del  y   0x0000000000001186 in main at overflow.c:14
```
breakpointを無効化する(削除はしない)
```
pwndbg> disable 1
pwndbg> info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000001168 in copy_arg at overflow.c:8
2       breakpoint     keep y   0x0000000000001155 in copy_arg at overflow.c:7
3       breakpoint     del  y   0x0000000000001186 in main at overflow.c:14
```
breakpointを有効化する
```
pwndbg> enable 1
pwndbg> info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000001168 in copy_arg at overflow.c:8
2       breakpoint     keep y   0x0000000000001155 in copy_arg at overflow.c:7
3       breakpoint     del  y   0x0000000000001186 in main at overflow.c:14
```
breakpointをひとつ削除する
```
pwndbg> info b
Num     Type           Disp Enb Address            What
2       breakpoint     keep y   0x0000000000001155 in copy_arg at overflow.c:7
3       breakpoint     del  y   0x0000000000001186 in main at overflow.c:14
```
breakpointを全部削除する
```
pwndbg> info b
No breakpoints or watchpoints.
```

### watchpoint

watchpointを設定すると、その変数が変更された<span style="color:salmon">後</span>で一旦停止できる

変数`buffer`にwatchpointをセットする
```
pwndbg> watch buffer
Watchpoint 5: buffer
```
セットされたwatchipointの一覧
```
pwndbg> info watchpoints
Num     Type           Disp Enb Address            What
5       watchpoint     keep y                      buffer
```

## referencess

- [rkubik/cheat_sheet.txt](https://gist.github.com/rkubik/b96c23bd8ed58333de37f2b8cd052c30)
- [pwndbg](https://github.com/pwndbg/pwndbg)