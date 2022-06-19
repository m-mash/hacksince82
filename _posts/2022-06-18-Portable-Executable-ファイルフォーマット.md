---
layout: post_blog
tags: windows pe portable executable 
---

[マルウエアデータサイエンス本](https://www.amazon.co.jp/%E3%83%9E%E3%83%AB%E3%82%A6%E3%82%A7%E3%82%A2-%E3%83%87%E3%83%BC%E3%82%BF%E3%82%B5%E3%82%A4%E3%82%A8%E3%83%B3%E3%82%B9-%E3%82%B5%E3%82%A4%E3%83%90%E3%83%BC%E6%94%BB%E6%92%83%E3%81%AE%E6%A4%9C%E5%87%BA%E3%81%A8%E5%88%86%E6%9E%90-Joshua-Saxe-ebook/dp/B07Z61K4RH)を読み始めて少し触れたので、
[Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)とか[corkami](https://web.archive.org/web/20150821170441/https://code.google.com/p/corkami/wiki/PE#NT_Headers_('PE_Header'))とか[0xRick's Blog](https://0xrick.github.io/win-internals/pe2/)を参考に調べた。

## まずは用語から

今は飛ばして、後でわからなかったら戻ってきて。

|用語|意味|
|---|---|
|COFF|UNIXで使用される実行ファイル、オブジェクトファイル、共有ライブラリのフォーマット。PEはCOFFの拡張。|
|RVA|`Relative Virtual Address`でメモリ上でのImage Baseからの相対的なアドレス|
|image file|EXE、DLL等の実行ファイル|
|object file|リンカへの入力として与えられるファイル。リンカはobject fileからimage fileを生成し、それがローダーによる入力として使用される|
|Relocation、再配置|プログラムがメモリにロードされるときは基本`OptionalHeader`にある`ImageBase`にロードされるが、すでに使用されていたりメモリの確保に失敗した場合に行われる操作。|


## フォーマット

ざっくりいうと、以下から構成される。

- DOSヘッダ
- NTヘッダ（PEヘッダ）
- セクションヘッダ * セクションの数（セクションテーブルとも）
- セクション * セクションの数

ここからは詳しく見ていく。

### DOSヘッダ

- `e_magic`：PEの場合マジックナンバーとして`0x5A4D`(MZ)
- `e_lfanew`：NTヘッダのOffset

### NTヘッダ（PEヘッダ）

- `Signature`：PEの場合`0x50450000`(PE\0\0)
- `FileHeader`：構造体、詳細は[こちら](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#coff-file-header-object-and-image)
    - `Machine`：使用されるCPUを記載。`0x014c`=i386、 `0x8664`=x64
    - `NumberOfSections`
    - `TimeDateStamp`：ファイルが作成された日時
    - `PointerToSymbolTable`：シンボルテーブルのOffset
    - `NumberOfSymbols`：シンボルテーブルのエントリの数
    - `SizeOfOptionalHeader`
    - `Characteristics`：ファイルの属性
- `OptiaonalHeader`：構造体、optionalって名前だけど必須、詳細は[こちら](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#optional-header-image-only)
    - `Magic`：`0x10b`=PE、`0x20b`=PE32+
    - `MajorLinkerVersion`
    - `MinorLinkerVersion`
    - `SizeOfCode`：textセクションのサイズ、複数あれば合計で
    - `SizeOfInitializedData`：dataセクションのサイズ、複数あれば合計で
    - `SizeOfUninitializedData`：BSSセクションのサイズ、複数あれば合計で
    - `AddressOfEntryPoint`：メモリ上におけるimage baseからのエントリーポイントの相対的なアドレス
    - `BaseOfCode`：メモリ上におけるimage baseからのtextセクションの開始位置の相対的なアドレス

    移行はWindows特有のフィールド
    - `ImageBase`：PEがメモリにロードされるときのperferredなアドレス。デフォルトはDLL=`0x10000000`、Windows CE EXE=`0x00010000`、Windows NT, Windows 2000, Windows XP, Windows 95, Windows 98, and Windows Me=`0x00400000`
    - `SectionAlignment`：各セクションがメモリにロードされる際のAlignment
    - `FileAlignment`：各セクションのファイル内のAlignment
    - `MajorOperatingSystemVersion` / `MinorOperatingSystemVersion `
    - `MajorImageVersion` / `MinorImageVersion`
    - `MajorSubsystemVersion` / `MinorSubsystemVersion`
    - `Win32VersionValue`
    - `SizeOfImage`：メモリロード時のサイズ、`SectionAlignment`の倍数になる
    - `SizeOfHeaders`：DOSヘッダ、NTヘッダ（PEヘッダ）、セクションヘッダの合計サイズを`File Alignment`の倍数で切り上げた値
    - `CheckSum`
    - `Subsystem`：詳細は[こちら](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#windows-subsystem)
    - `DllCharacteristics`：詳細は[こちら](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#dll-characteristics)
    - `SizeOfStackReserve`：確保したいスタックのサイズ
    - `SizeOfStackCommit`：確約されるスタックのサイズ
    - `SizeOfHeapReserve`
    - `SizeOfHeapCommit`
    - `LoaderFlags`
    - `NumberOfRvaAndSizes`：`OptiaonalHeader`の残り部分にある各Data Directoryエントリの位置とサイズ（`DIRECTORY_ENTRY_EXPORT`、`DIRECTORY_ENTRY_EXPORT`から始まって`IMAGE_DIRECTORY_ENTRY_RESERVED`までのことを言ってるっぽい）**Note that the number of directories is not fixed. Before looking for a specific directory, check the NumberOfRvaAndSizes field in the optional header.** の[記載あり](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#optional-header-data-directories-image-only)

### セクションヘッダ

以下のフィールドがセクションの数だけ存在する。

- `Name`
- `VirtualSize`：メモリロード時のそのセクションの合計サイズ
- `VirtualAddress`：メモリロード時のそのセクションのImage Baseからの相対的なアドレス
- `SizeOfRawData`：ファイル上でのセクションのサイズ
- `PointerToRawData`
- `PointerToRelocations`：そのセクションの`relocation`エントリへのファイルポインタ
- `PointerToLinenumbers`
- `NumberOfRelocations`
- `NumberOfLinenumbers`
- `Characteristics`：各セクションにつける属性。詳細は[こちら](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#section-flags)


### セクション

セクションの数だけ存在する。

[Microsoft Docsに各セクションの説明](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#special-sections)があった。

|セクション|説明|
|---|---|
|.text|実行可能なコード|
|.data|初期化されたデータ|
|.bss|初期化されていないデータ|
|.rdata|Read-Onlyの初期化されたデータ|
|.eata|Export Tableが含まれる|
|.idata|Import Table|
|.reloc|再配置に関する情報|
|.rsrc|画像やアイコンや埋め込まれたバイナリなど、プログラムによって使われるリソース|
|.tls|プログラムの実行中のスレッドが使用するストレージ(Thread Local Storage)|

## pythonの[pefile](https://github.com/erocarrera/pefile)で分析

まずはprint_info()で全体を見る

{% highlight python %}
import pefile

pe = pefile.PE("xxx.exe")
pe.print_info()
{% endhighlight %}

```
----------Parsing Warnings----------

AddressOfEntryPoint lies outside the sections' boundaries. AddressOfEntryPoint: 0xcc00ffee

----------DOS_HEADER----------

[IMAGE_DOS_HEADER]
0x0        0x0   e_magic:                       0x5A4D
0x2        0x2   e_cblp:                        0x90
0x4        0x4   e_cp:                          0x3
0x6        0x6   e_crlc:                        0x0
0x8        0x8   e_cparhdr:                     0x4
0xA        0xA   e_minalloc:                    0x0
0xC        0xC   e_maxalloc:                    0xFFFF
0xE        0xE   e_ss:                          0x0
0x10       0x10  e_sp:                          0xB8
0x12       0x12  e_csum:                        0x0
0x14       0x14  e_ip:                          0x0
0x16       0x16  e_cs:                          0x0
0x18       0x18  e_lfarlc:                      0x40
0x1A       0x1A  e_ovno:                        0x0
0x1C       0x1C  e_res:
0x24       0x24  e_oemid:                       0x0
0x26       0x26  e_oeminfo:                     0x0
0x28       0x28  e_res2:
0x3C       0x3C  e_lfanew:                      0xE0

----------NT_HEADERS----------

[IMAGE_NT_HEADERS]
0xE0       0x0   Signature:                     0x4550

----------FILE_HEADER----------

[IMAGE_FILE_HEADER]
0xE4       0x0   Machine:                       0x14C
0xE6       0x2   NumberOfSections:              0x5
0xE8       0x4   TimeDateStamp:                 0x4F79D506 [Mon Apr  2 16:34:14 2012 UTC]
0xEC       0x8   PointerToSymbolTable:          0x0
0xF0       0xC   NumberOfSymbols:               0x0
0xF4       0x10  SizeOfOptionalHeader:          0xE0
0xF6       0x12  Characteristics:               0x10F
Flags: IMAGE_FILE_32BIT_MACHINE, IMAGE_FILE_EXECUTABLE_IMAGE, IMAGE_FILE_LINE_NUMS_STRIPPED, IMAGE_FILE_LOCAL_SYMS_STRIPPED, IMAGE_FILE_RELOCS_STRIPPED

----------OPTIONAL_HEADER----------

[IMAGE_OPTIONAL_HEADER]
0xF8       0x0   Magic:                         0x10B
0xFA       0x2   MajorLinkerVersion:            0x6
0xFB       0x3   MinorLinkerVersion:            0x0
0xFC       0x4   SizeOfCode:                    0x32A00
0x100      0x8   SizeOfInitializedData:         0x64200
0x104      0xC   SizeOfUninitializedData:       0x0
0x108      0x10  AddressOfEntryPoint:           0xCC00FFEE
0x10C      0x14  BaseOfCode:                    0x1000
0x110      0x18  BaseOfData:                    0x1000
0x114      0x1C  ImageBase:                     0x400000
0x118      0x20  SectionAlignment:              0x1000
0x11C      0x24  FileAlignment:                 0x200
0x120      0x28  MajorOperatingSystemVersion:   0x4
0x122      0x2A  MinorOperatingSystemVersion:   0x0
0x124      0x2C  MajorImageVersion:             0x0
0x126      0x2E  MinorImageVersion:             0x0
0x128      0x30  MajorSubsystemVersion:         0x4
0x12A      0x32  MinorSubsystemVersion:         0x0
0x12C      0x34  Reserved1:                     0x0
0x130      0x38  SizeOfImage:                   0x9A000
0x134      0x3C  SizeOfHeaders:                 0x1000
0x138      0x40  CheckSum:                      0x0
0x13C      0x44  Subsystem:                     0x2
0x13E      0x46  DllCharacteristics:            0x0
0x140      0x48  SizeOfStackReserve:            0x100000
0x144      0x4C  SizeOfStackCommit:             0x1000
0x148      0x50  SizeOfHeapReserve:             0x100000
0x14C      0x54  SizeOfHeapCommit:              0x1000
0x150      0x58  LoaderFlags:                   0x0
0x154      0x5C  NumberOfRvaAndSizes:           0x10
DllCharacteristics:

----------PE Sections----------

[IMAGE_SECTION_HEADER]
0x1D8      0x0   Name:                          .text
0x1E0      0x8   Misc:                          0x32830
0x1E0      0x8   Misc_PhysicalAddress:          0x32830
0x1E0      0x8   Misc_VirtualSize:              0x32830
0x1E4      0xC   VirtualAddress:                0x1000
0x1E8      0x10  SizeOfRawData:                 0x32A00
0x1EC      0x14  PointerToRawData:              0x400
0x1F0      0x18  PointerToRelocations:          0x0
0x1F4      0x1C  PointerToLinenumbers:          0x0
0x1F8      0x20  NumberOfRelocations:           0x0
0x1FA      0x22  NumberOfLinenumbers:           0x0
0x1FC      0x24  Characteristics:               0x60000020
Flags: IMAGE_SCN_CNT_CODE, IMAGE_SCN_MEM_EXECUTE, IMAGE_SCN_MEM_READ
Entropy: 4.457650 (Min=0.0, Max=8.0)
MD5     hash: c3a764364d86547f51d0635201e60562
SHA-1   hash: 3d2230cbe03391be7e5c3766a189b70e0579cb03
SHA-256 hash: 46c2cd9f69b24ce045edeb26a2ebbbaf218974428c493ad6dc8626bed681b7dd
SHA-512 hash: 86ad1fb65a5643128fb53751be0534f3d2b2625f2ec13fe40dad3a762b24fe6f86290128ace56bc62f95eb8598629a1de07cb6145d30e50e965fadb07f4c2d1e

[IMAGE_SECTION_HEADER]
0x200      0x0   Name:                          .rdata
0x208      0x8   Misc:                          0x427A
0x208      0x8   Misc_PhysicalAddress:          0x427A
0x208      0x8   Misc_VirtualSize:              0x427A
0x20C      0xC   VirtualAddress:                0x34000
0x210      0x10  SizeOfRawData:                 0x4400
0x214      0x14  PointerToRawData:              0x32E00
0x218      0x18  PointerToRelocations:          0x0
0x21C      0x1C  PointerToLinenumbers:          0x0
0x220      0x20  NumberOfRelocations:           0x0
0x222      0x22  NumberOfLinenumbers:           0x0
0x224      0x24  Characteristics:               0x40000040
Flags: IMAGE_SCN_CNT_INITIALIZED_DATA, IMAGE_SCN_MEM_READ
Entropy: 5.164174 (Min=0.0, Max=8.0)
MD5     hash: 17fb67e3c303ff7092caa8336ecd7343
SHA-1   hash: c6b2ea68634623aebbb6cdb8c750af9593c6c440
SHA-256 hash: 6be910898286e002634e51239fabac5d4323f99094829072f43d14cc4293c9c8
SHA-512 hash: 3861674650879fcfd123f9695254b8e3e49b9eab710fcd98c97c4b3f5003dfc7e18072aa4ab7f7858519e469d9f56b979402122c0e15e164979a12e3535d624a

[IMAGE_SECTION_HEADER]
0x228      0x0   Name:                          .data
0x230      0x8   Misc:                          0x5CFF8
0x230      0x8   Misc_PhysicalAddress:          0x5CFF8
0x230      0x8   Misc_VirtualSize:              0x5CFF8
0x234      0xC   VirtualAddress:                0x39000
0x238      0x10  SizeOfRawData:                 0x2A00
0x23C      0x14  PointerToRawData:              0x37200
0x240      0x18  PointerToRelocations:          0x0
0x244      0x1C  PointerToLinenumbers:          0x0
0x248      0x20  NumberOfRelocations:           0x0
0x24A      0x22  NumberOfLinenumbers:           0x0
0x24C      0x24  Characteristics:               0xC0000040
Flags: IMAGE_SCN_CNT_INITIALIZED_DATA, IMAGE_SCN_MEM_READ, IMAGE_SCN_MEM_WRITE
Entropy: 2.528280 (Min=0.0, Max=8.0)
MD5     hash: 40bb262272a927dfc9de741f931df366
SHA-1   hash: f8c680d52fc42186397201d7e1b9959796278ccb
SHA-256 hash: 642649d18a65bffdb10cf2273eddb4b35de4283bf65c627bd8557bc2c15756a3
SHA-512 hash: ab7f61643074084fa9ed52ae2e2f02baa98f8599b0cef025d6b43ce7cab64064ee06b9ced3469fd0094f1862bb7f212f88f377371cc8e0a26e0c51a0f0768600

[IMAGE_SECTION_HEADER]
0x250      0x0   Name:                          .idata
0x258      0x8   Misc:                          0xBB0
0x258      0x8   Misc_PhysicalAddress:          0xBB0
0x258      0x8   Misc_VirtualSize:              0xBB0
0x25C      0xC   VirtualAddress:                0x96000
0x260      0x10  SizeOfRawData:                 0xC00
0x264      0x14  PointerToRawData:              0x39C00
0x268      0x18  PointerToRelocations:          0x0
0x26C      0x1C  PointerToLinenumbers:          0x0
0x270      0x20  NumberOfRelocations:           0x0
0x272      0x22  NumberOfLinenumbers:           0x0
0x274      0x24  Characteristics:               0xC0000040
Flags: IMAGE_SCN_CNT_INITIALIZED_DATA, IMAGE_SCN_MEM_READ, IMAGE_SCN_MEM_WRITE
Entropy: 3.487657 (Min=0.0, Max=8.0)
MD5     hash: 4f5feb5038667a10ab5d11233c82be9a
SHA-1   hash: 2fb0d65ec13c59f2d8a58dca8e956da73fd1db50
SHA-256 hash: 346075b0245609ed4a673f4db7a5b2b335835f41b3b3fe0ff97fa853d73e8a70
SHA-512 hash: 7e6b255f8fdb315423d923e6b77ea46050018fafac4b15553e5ee2896c73c30f6b5980406cdd14ae2413de03f89784906d9f38ce859b1d7781b79e9ae5357a41

[IMAGE_SECTION_HEADER]
0x278      0x0   Name:                          .reloc
0x280      0x8   Misc:                          0x211D
0x280      0x8   Misc_PhysicalAddress:          0x211D
0x280      0x8   Misc_VirtualSize:              0x211D
0x284      0xC   VirtualAddress:                0x97000
0x288      0x10  SizeOfRawData:                 0x2200
0x28C      0x14  PointerToRawData:              0x3A800
0x290      0x18  PointerToRelocations:          0x0
0x294      0x1C  PointerToLinenumbers:          0x0
0x298      0x20  NumberOfRelocations:           0x0
0x29A      0x22  NumberOfLinenumbers:           0x0
0x29C      0x24  Characteristics:               0x42000040
Flags: IMAGE_SCN_CNT_INITIALIZED_DATA, IMAGE_SCN_MEM_DISCARDABLE, IMAGE_SCN_MEM_READ
Entropy: 0.000000 (Min=0.0, Max=8.0)
MD5     hash: d946c4e00b10be82f8d142f508ece41d
SHA-1   hash: 87149ca9fc689d0d02866276f9112adbdeea06f2
SHA-256 hash: e8b31e302d11fbf7da124b537ba2d44f88e165da03c6557e2b0f6dc486e025bb
SHA-512 hash: b30f8b91e7591ec872bf52a792bc1a6f97bc368c7ebe7538d87d220608d1d7d3dca20afb27b711a6b81b70c477507df3dac27095b01e2864dee6e21404998872

----------Directories----------

[IMAGE_DIRECTORY_ENTRY_EXPORT]
0x158      0x0   VirtualAddress:                0x0
0x15C      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_IMPORT]
0x160      0x0   VirtualAddress:                0x96000
0x164      0x4   Size:                          0x3C
[IMAGE_DIRECTORY_ENTRY_RESOURCE]
0x168      0x0   VirtualAddress:                0x0
0x16C      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_EXCEPTION]
0x170      0x0   VirtualAddress:                0x0
0x174      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_SECURITY]
0x178      0x0   VirtualAddress:                0x0
0x17C      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_BASERELOC]
0x180      0x0   VirtualAddress:                0x0
0x184      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_DEBUG]
0x188      0x0   VirtualAddress:                0x0
0x18C      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_COPYRIGHT]
0x190      0x0   VirtualAddress:                0x0
0x194      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_GLOBALPTR]
0x198      0x0   VirtualAddress:                0x0
0x19C      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_TLS]
0x1A0      0x0   VirtualAddress:                0x0
0x1A4      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG]
0x1A8      0x0   VirtualAddress:                0x0
0x1AC      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT]
0x1B0      0x0   VirtualAddress:                0x0
0x1B4      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_IAT]
0x1B8      0x0   VirtualAddress:                0x0
0x1BC      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT]
0x1C0      0x0   VirtualAddress:                0x0
0x1C4      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR]
0x1C8      0x0   VirtualAddress:                0x0
0x1CC      0x4   Size:                          0x0
[IMAGE_DIRECTORY_ENTRY_RESERVED]
0x1D0      0x0   VirtualAddress:                0x0
0x1D4      0x4   Size:                          0x0

----------Imported symbols----------

[IMAGE_IMPORT_DESCRIPTOR]
0x39C00    0x0   OriginalFirstThunk:            0x0
0x39C00    0x0   Characteristics:               0x0
0x39C04    0x4   TimeDateStamp:                 0x0        [Thu Jan  1 00:00:00 1970 UTC]
0x39C08    0x8   ForwarderChain:                0x0
0x39C0C    0xC   Name:                          0x96424
0x39C10    0x10  FirstThunk:                    0x96230

KERNEL32.DLL.GetLocalTime Hint[0]
KERNEL32.DLL.ExitThread Hint[0]
KERNEL32.DLL.CloseHandle Hint[0]
KERNEL32.DLL.WriteFile Hint[0]
KERNEL32.DLL.CreateFileA Hint[0]
KERNEL32.DLL.ExitProcess Hint[0]
KERNEL32.DLL.CreateProcessA Hint[0]
KERNEL32.DLL.GetTickCount Hint[0]
KERNEL32.DLL.GetModuleFileNameA Hint[0]
KERNEL32.DLL.GetSystemDirectoryA Hint[0]
KERNEL32.DLL.Sleep Hint[0]
KERNEL32.DLL.GetTimeFormatA Hint[0]
KERNEL32.DLL.GetDateFormatA Hint[0]
KERNEL32.DLL.GetLastError Hint[0]
KERNEL32.DLL.CreateThread Hint[0]
KERNEL32.DLL.GetFileSize Hint[0]
KERNEL32.DLL.GetFileAttributesA Hint[0]
KERNEL32.DLL.FindClose Hint[0]
KERNEL32.DLL.FileTimeToSystemTime Hint[0]
KERNEL32.DLL.FileTimeToLocalFileTime Hint[0]
KERNEL32.DLL.FindNextFileA Hint[0]
KERNEL32.DLL.FindFirstFileA Hint[0]
KERNEL32.DLL.ReadFile Hint[0]
KERNEL32.DLL.SetFilePointer Hint[0]
KERNEL32.DLL.WriteConsoleA Hint[0]
KERNEL32.DLL.GetStdHandle Hint[0]
KERNEL32.DLL.LoadLibraryA Hint[0]
KERNEL32.DLL.GetProcAddress Hint[0]
KERNEL32.DLL.GetModuleHandleA Hint[0]
KERNEL32.DLL.FormatMessageA Hint[0]
KERNEL32.DLL.GlobalUnlock Hint[0]
KERNEL32.DLL.GlobalLock Hint[0]
KERNEL32.DLL.UnmapViewOfFile Hint[0]
KERNEL32.DLL.MapViewOfFile Hint[0]
KERNEL32.DLL.CreateFileMappingA Hint[0]
KERNEL32.DLL.SetFileTime Hint[0]
KERNEL32.DLL.GetFileTime Hint[0]
KERNEL32.DLL.ExpandEnvironmentStringsA Hint[0]
KERNEL32.DLL.SetFileAttributesA Hint[0]
KERNEL32.DLL.GetTempPathA Hint[0]
KERNEL32.DLL.GetCurrentProcess Hint[0]
KERNEL32.DLL.TerminateProcess Hint[0]
KERNEL32.DLL.OpenProcess Hint[0]
KERNEL32.DLL.GetComputerNameA Hint[0]
KERNEL32.DLL.GetLocaleInfoA Hint[0]
KERNEL32.DLL.GetVersionExA Hint[0]
KERNEL32.DLL.TerminateThread Hint[0]
KERNEL32.DLL.FlushFileBuffers Hint[0]
KERNEL32.DLL.SetStdHandle Hint[0]
KERNEL32.DLL.IsBadWritePtr Hint[0]
KERNEL32.DLL.IsBadReadPtr Hint[0]
KERNEL32.DLL.HeapValidate Hint[0]
KERNEL32.DLL.GetStartupInfoA Hint[0]
KERNEL32.DLL.GetCommandLineA Hint[0]
KERNEL32.DLL.GetVersion Hint[0]
KERNEL32.DLL.DebugBreak Hint[0]
KERNEL32.DLL.InterlockedDecrement Hint[0]
KERNEL32.DLL.OutputDebugStringA Hint[0]
KERNEL32.DLL.InterlockedIncrement Hint[0]
KERNEL32.DLL.HeapAlloc Hint[0]
KERNEL32.DLL.HeapReAlloc Hint[0]
KERNEL32.DLL.HeapFree Hint[0]
KERNEL32.DLL.HeapDestroy Hint[0]
KERNEL32.DLL.HeapCreate Hint[0]
KERNEL32.DLL.VirtualFree Hint[0]
KERNEL32.DLL.VirtualAlloc Hint[0]
KERNEL32.DLL.WideCharToMultiByte Hint[0]
KERNEL32.DLL.MultiByteToWideChar Hint[0]
KERNEL32.DLL.LCMapStringA Hint[0]
KERNEL32.DLL.LCMapStringW Hint[0]
KERNEL32.DLL.GetCPInfo Hint[0]
KERNEL32.DLL.GetACP Hint[0]
KERNEL32.DLL.GetOEMCP Hint[0]
KERNEL32.DLL.UnhandledExceptionFilter Hint[0]
KERNEL32.DLL.FreeEnvironmentStringsA Hint[0]
KERNEL32.DLL.FreeEnvironmentStringsW Hint[0]
KERNEL32.DLL.GetEnvironmentStrings Hint[0]
KERNEL32.DLL.GetEnvironmentStringsW Hint[0]
KERNEL32.DLL.SetHandleCount Hint[0]
KERNEL32.DLL.GetFileType Hint[0]
KERNEL32.DLL.RtlUnwind Hint[0]
KERNEL32.DLL.SetConsoleCtrlHandler Hint[0]
KERNEL32.DLL.GetStringTypeA Hint[0]
KERNEL32.DLL.GetStringTypeW Hint[0]
KERNEL32.DLL.SetEndOfFile Hint[0]

[IMAGE_IMPORT_DESCRIPTOR]
0x39C14    0x0   OriginalFirstThunk:            0x0
0x39C14    0x0   Characteristics:               0x0
0x39C18    0x4   TimeDateStamp:                 0x0        [Thu Jan  1 00:00:00 1970 UTC]
0x39C1C    0x8   ForwarderChain:                0x0
0x39C20    0xC   Name:                          0x96431
0x39C24    0x10  FirstThunk:                    0x963F4

USER32.dll.MessageBoxA Hint[0]
```

各セクションを見る

{% highlight python %}
import pefile

pe = pefile.PE("ircbot.exe")

for section in pe.sections:
    print(section.Name, hex(section.VirtualAddress), hex(section.Misc_VirtualSize), section.SizeOfRawData)
{% endhighlight %}

インポートされたシンボルを見る

{% highlight python %}
import pefile

pe = pefile.PE("ircbot.exe")

for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(entry.dll)
    for i in entry.imports:
        print('\t', i.name, '\t', hex(i.address))
{% endhighlight %}

## おまけ

### [icoutils](https://www.nongnu.org/icoutils/)のwrestoolで画像を取り出す

```
mkdir images
mkdir output

# xxx.exeに含まれる画像をimagesに吐き出して、
wrestool -x xxx.exe -o images

# pngに変換してoutputに吐き出す
icotool -x -o output images/*
```

### stringsコマンドででファイル内の文字列を表示する

```
strings xxx.exe
```

このexeファイルが持っていそうな機能に見当をつけることができる。例えば
- ファイルをダウンロードすることができそう
- Webサーバの機能を持っていて外部からの通信を待ち受けることができそう
など。

# references

- [Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)
- [corkami](https://web.archive.org/web/20150821170441/https://code.google.com/p/corkami/wiki/PE#NT_Headers_('PE_Header'))
- [0xRick's Blog](https://0xrick.github.io/win-internals/pe2/)
- [erocarrera/pefile](https://github.com/erocarrera/pefile)
- [icoutils](https://www.nongnu.org/icoutils/)
- [PE format poster](http://www.openrce.org/reference_library/files/reference/PE%20Format.pdf)