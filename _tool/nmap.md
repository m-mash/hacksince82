---
layout: post
title: nmap
tags: nmap enumeration reconnaissance scan 
created: 2022-06-17
updated: 2022-06-17
---

## 基本
```
$ nmap -A -p- -oN <filename> <ip address>
```

## /usr/share/nmapの中身

### nmap-os-db

`/usr/share/nmap/nmap-os-db`

nmapがフィンガープリントからOSを判別する際に使用する定義ファイル。

ひとつのOS定義に以下のような記載がされている。

```
# Windows 10 build 10240
Fingerprint Microsoft Windows 10 1507
Class Microsoft | Windows | 10 | general purpose
CPE cpe:/o:microsoft:windows_10:1507 auto
SEQ(SP=104-10E%GCD=1-6%ISR=106-110%TI=I%CI=I%II=I%SS=S%TS=A)
OPS(O1=M5BCNW8ST11%O2=M5BCNW8ST11%O3=M5BCNW8NNT11%O4=M5BCNW8ST11%O5=M5BCNW8ST11%O6=M5BCST11)
WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)
ECN(R=Y%DF=Y%T=7B-85%TG=80%W=2000%O=M5BCNW8NNS%CC=N%Q=)
T1(R=Y%DF=Y%T=7B-85%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)
T3(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)
T4(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
T5(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
T6(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
T7(R=Y%DF=Y%T=7B-85%TG=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
U1(DF=N%T=7B-85%TG=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)
IE(DFI=N%T=7B-85%TG=80%CD=Z)
```

ここで例えば`"cpe:/o:microsoft:windows"`で`grep`して`sort | uniq`してみると以下が得られる。
(本当にこの数のOS・バージョンを区別できるかは確認していない。複数の異なるCPEが同一のフィンガープリントとして識別されることもあるのかもしれない)

```
$ cat nmap-os-db | grep "cpe:/o:microsoft:windows" | sort | uniq

CPE cpe:/o:microsoft:windows_10:1507 auto
CPE cpe:/o:microsoft:windows_10:1511 auto
CPE cpe:/o:microsoft:windows_10:1607 auto
CPE cpe:/o:microsoft:windows_10:1703 auto
CPE cpe:/o:microsoft:windows_10 auto
CPE cpe:/o:microsoft:windows_2000::-
CPE cpe:/o:microsoft:windows_2000:::advanced_server
CPE cpe:/o:microsoft:windows_2000 auto
CPE cpe:/o:microsoft:windows_2000::- auto
CPE cpe:/o:microsoft:windows_2000:::professional
CPE cpe:/o:microsoft:windows_2000::sp1
CPE cpe:/o:microsoft:windows_2000::sp2
CPE cpe:/o:microsoft:windows_2000::sp2 auto
CPE cpe:/o:microsoft:windows_2000::sp3
CPE cpe:/o:microsoft:windows_2000::sp3 auto
CPE cpe:/o:microsoft:windows_2000::sp4
CPE cpe:/o:microsoft:windows_2000::sp4:advanced_server
CPE cpe:/o:microsoft:windows_2000::sp4 auto
CPE cpe:/o:microsoft:windows_2000::sp4:professional
CPE cpe:/o:microsoft:windows_2000::sp4:server
CPE cpe:/o:microsoft:windows_2000::sp6
CPE cpe:/o:microsoft:windows_2008
CPE cpe:/o:microsoft:windows_2012
CPE cpe:/o:microsoft:windows:3.11 auto
CPE cpe:/o:microsoft:windows:3.1 auto
CPE cpe:/o:microsoft:windows_7
CPE cpe:/o:microsoft:windows_7::-
CPE cpe:/o:microsoft:windows_7 auto
CPE cpe:/o:microsoft:windows_7::- auto
CPE cpe:/o:microsoft:windows_7::-:professional
CPE cpe:/o:microsoft:windows_7::sp1
CPE cpe:/o:microsoft:windows_7::sp1 auto
CPE cpe:/o:microsoft:windows_7::sp1:professional
CPE cpe:/o:microsoft:windows_7::sp1:ultimate
CPE cpe:/o:microsoft:windows_7::-:ultimate
CPE cpe:/o:microsoft:windows_7:::ultimate
CPE cpe:/o:microsoft:windows_8
CPE cpe:/o:microsoft:windows_8.1
CPE cpe:/o:microsoft:windows_8.1 auto
CPE cpe:/o:microsoft:windows_8.1:r1 auto
CPE cpe:/o:microsoft:windows_8 auto
CPE cpe:/o:microsoft:windows_95 auto
CPE cpe:/o:microsoft:windows_98 auto
CPE cpe:/o:microsoft:windows auto
CPE cpe:/o:microsoft:windows_ce:3.0 auto
CPE cpe:/o:microsoft:windows_ce:4.20
CPE cpe:/o:microsoft:windows_ce:4.2 auto
CPE cpe:/o:microsoft:windows_ce:5.0
CPE cpe:/o:microsoft:windows_ce:5.0.1400
CPE cpe:/o:microsoft:windows_ce:5.0 auto
CPE cpe:/o:microsoft:windows_ce:5 auto
CPE cpe:/o:microsoft:windows_ce:6.0 auto
CPE cpe:/o:microsoft:windows_ce auto
CPE cpe:/o:microsoft:windows_embedded_compact_7
CPE cpe:/o:microsoft:windows_me auto
CPE cpe:/o:microsoft:windows_mobile:2003 auto
CPE cpe:/o:microsoft:windows_mobile:5 auto
CPE cpe:/o:microsoft:windows_mobile:6 auto
CPE cpe:/o:microsoft:windows_nt:3.1 auto
CPE cpe:/o:microsoft:windows_nt:3.51 auto
CPE cpe:/o:microsoft:windows_nt:3.51:sp5
CPE cpe:/o:microsoft:windows_nt:4.0 auto
CPE cpe:/o:microsoft:windows_nt:4.0:- auto
CPE cpe:/o:microsoft:windows_nt:4.0:sp3 auto
CPE cpe:/o:microsoft:windows_nt:4.0:sp5 auto
CPE cpe:/o:microsoft:windows_nt:4.0:sp6 auto
CPE cpe:/o:microsoft:windows_nt:4.0:sp7 auto
CPE cpe:/o:microsoft:windows_nt auto
CPE cpe:/o:microsoft:windows_nt::sp6a
CPE cpe:/o:microsoft:windows_nt::sp6 auto
CPE cpe:/o:microsoft:windows_server_2003
CPE cpe:/o:microsoft:windows_server_2003::-
CPE cpe:/o:microsoft:windows_server_2003 auto
CPE cpe:/o:microsoft:windows_server_2003::- auto
CPE cpe:/o:microsoft:windows_server_2003:::enterprise
CPE cpe:/o:microsoft:windows_server_2003::r2
CPE cpe:/o:microsoft:windows_server_2003:r2 auto
CPE cpe:/o:microsoft:windows_server_2003:r2:sp1
CPE cpe:/o:microsoft:windows_server_2003:r2:sp2
CPE cpe:/o:microsoft:windows_server_2003::r2_sp2
CPE cpe:/o:microsoft:windows_server_2003:r2:sp2 auto
CPE cpe:/o:microsoft:windows_server_2003::sp1
CPE cpe:/o:microsoft:windows_server_2003::sp1 auto
CPE cpe:/o:microsoft:windows_server_2003::sp2
CPE cpe:/o:microsoft:windows_server_2003::sp2 auto
CPE cpe:/o:microsoft:windows_server_2003::sp2:enterprise
CPE cpe:/o:microsoft:windows_server_2003::sp2:x64
CPE cpe:/o:microsoft:windows_server_2003::sp3
CPE cpe:/o:microsoft:windows_server_2008
CPE cpe:/o:microsoft:windows_server_2008 auto
CPE cpe:/o:microsoft:windows_server_2008::beta3
CPE cpe:/o:microsoft:windows_server_2008:r2
CPE cpe:/o:microsoft:windows_server_2008:r2 auto
CPE cpe:/o:microsoft:windows_server_2008:r2:sp1
CPE cpe:/o:microsoft:windows_server_2008:r2:sp1 auto
CPE cpe:/o:microsoft:windows_server_2008::sp1
CPE cpe:/o:microsoft:windows_server_2008::sp1 auto
CPE cpe:/o:microsoft:windows_server_2008::sp2
CPE cpe:/o:microsoft:windows_server_2008::sp2 auto
CPE cpe:/o:microsoft:windows_server_2012
CPE cpe:/o:microsoft:windows_server_2012:-
CPE cpe:/o:microsoft:windows_server_2012 auto
CPE cpe:/o:microsoft:windows_server_2012:r2
CPE cpe:/o:microsoft:windows_server_2012:r2 auto
CPE cpe:/o:microsoft:windows_server_2016 auto
CPE cpe:/o:microsoft:windows_vista
CPE cpe:/o:microsoft:windows_vista::-
CPE cpe:/o:microsoft:windows_vista auto
CPE cpe:/o:microsoft:windows_vista::- auto
CPE cpe:/o:microsoft:windows_vista:::business
CPE cpe:/o:microsoft:windows_vista:::enterprise
CPE cpe:/o:microsoft:windows_vista::sp1
CPE cpe:/o:microsoft:windows_vista::sp1 auto
CPE cpe:/o:microsoft:windows_vista::sp1:home_premium
CPE cpe:/o:microsoft:windows_vista::sp2
CPE cpe:/o:microsoft:windows_vista::sp2 auto
CPE cpe:/o:microsoft:windows_xp
CPE cpe:/o:microsoft:windows_xp::-
CPE cpe:/o:microsoft:windows_xp auto
CPE cpe:/o:microsoft:windows_xp::- auto
CPE cpe:/o:microsoft:windows_xp:::embedded
CPE cpe:/o:microsoft:windows_xp:::professional
CPE cpe:/o:microsoft:windows_xp::sp1
CPE cpe:/o:microsoft:windows_xp::sp1a
CPE cpe:/o:microsoft:windows_xp::sp1 auto
CPE cpe:/o:microsoft:windows_xp::sp1:home
CPE cpe:/o:microsoft:windows_xp::sp1:professional
CPE cpe:/o:microsoft:windows_xp::sp2
CPE cpe:/o:microsoft:windows_xp::sp2 auto
CPE cpe:/o:microsoft:windows_xp::sp2:home
CPE cpe:/o:microsoft:windows_xp::sp2:media_center
CPE cpe:/o:microsoft:windows_xp::sp2:professional
CPE cpe:/o:microsoft:windows_xp::sp3
CPE cpe:/o:microsoft:windows_xp::sp3 auto
CPE cpe:/o:microsoft:windows_xp::sp3:embedded
CPE cpe:/o:microsoft:windows_xp::sp3:home
CPE cpe:/o:microsoft:windows_xp::sp3:professional
```

### nmap-services

`/usr/share/nmap/nmap-services`

nmapがスキャンするポートとサービスを対応付ける定義ファイル。

```
$ less nmap-services

...
ftp-data        20/sctp 0.000000        # File Transfer [Default Data] | FTP
ftp-data        20/tcp  0.001079        # File Transfer [Default Data]
ftp-data        20/udp  0.001878        # File Transfer [Default Data]
ftp     21/sctp 0.000000        # File Transfer [Control] | File Transfer Protocol [Control]
ftp     21/tcp  0.197667        # File Transfer [Control]
ftp     21/udp  0.004844        # File Transfer [Control]
ssh     22/sctp 0.000000        # Secure Shell Login | The Secure Shell (SSH) Protocol
ssh     22/tcp  0.182286        # Secure Shell Login
ssh     22/udp  0.003905        # Secure Shell Login
telnet  23/tcp  0.221265
telnet  23/udp  0.006211
...
```

各レコードの項目は、左から

- サービス名
- ポート番号/プロトコル
- Open-Frequency（正確には把握していないけど開いている確率？）
- コメント

となっている。
Open-Frequencyの使いどころは、`-port-ratio`オプションを使用して、指定した値よりもOpen-Freqencyが高いポートをスキャンするときに使用される。


## scripts

`/usr/share/nmap/scripts/`

スキャンする際のスクリプトが入ったディレクトリ。

2022/4/5現在で608の`.nse`ファイルが存在

## nselib

`/usr/share/nmap/nselib/`

`nse`ファイルを記述する上でのライブラリ。