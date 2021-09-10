---
title: LinuxでSHA-512のパスワードハッシュ作成方法まとめ
tags: Linux Perl Python Security
author: yumenomatayume
slide: false
---
「password」という平文を「salt」というsaltで、SHA-512で暗号化する場合を考える。
saltは本来ランダムな文字列にするが、ここでは出力結果を統一するために固定している。

出力形式はLinuxの`/etc/shadow`のパスワードフィールドの記載に合わせている。
以下の3つのフィールドをつなげたものである。

| 暗号化形式 | salt  | 暗号化パスワード       |
| ----- | ----- | -------------- |
| $6    | $salt | $IxDD~(略)~elDCy.g. |

(補足)暗号化形式の番号は以下のように対応する。

- 1: MD5
- 5: SHA-256
- 6: SHA-512

複数の方法があるが、汎用的と思われる順番で記載していく。

## perl

Linux環境には既にインストールされている場合が多い。

```bash
$ perl -e 'print crypt("password", "\$6\$salt");';echo
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.
```

最小構成(minimal)のOSにはインストールされていない事がある。

```bash
# docker(ubuntu:20.04)
$ docker run -it ubuntu:20.04 perl -e 'print crypt("password", "\$6\$salt");';echo
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.

# docker(centos:8) -> 未インストール
$ docker run -it centos:8 perl -e 'print crypt("password", "\$6\$salt");';echo
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "perl": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled 
```

## python

2.X系と3.X系でコマンド(print関数)が違うので、Platformに合わせる必要がある。

```bash
# python2.X
$ python2 -c "import crypt, getpass, pwd; print crypt.crypt('password','\$6\$salt\$')"
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.

# python3.X
$ python3 -c "import crypt, getpass, pwd; print(crypt.crypt('password','\$6\$salt\$'))"
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.
```

最小構成(minimal)のOSにはインストールされていない場合がある。
CentOS8にはシステム利用のための`platform-python`が入っている。

```bash
# docker(centos:8)
$ docker run -it centos:8 /usr/libexec/platform-python -c "import crypt, getpass, pwd; print(crypt.crypt('password','\$6\$salt\$'))"
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.
```

## openssl

```bash
$ openssl passwd -6 -salt=salt password
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.
```

ただし、opensslのversionが低い場合は、SHA-512を作成するためのオプションがない場合がある。

### SHA-512オプションがない例

CentOS6

```bash
$ cat /etc/redhat-release 
CentOS release 6.10 (Final)

$ openssl version
OpenSSL 1.0.1e-fips 11 Feb 2013

$ openssl passwd --help
Usage: passwd [options] [passwords]
where options are
-crypt             standard Unix password algorithm (default)
-1                 MD5-based password algorithm
-apr1              MD5-based password algorithm, Apache variant
-salt string       use provided salt
-in file           read passwords from file
-stdin             read passwords from stdin
-noverify          never verify when reading password from terminal
-quiet             no warnings
-table             format output as table
-reverse           switch table columns
```

### SHA-512オプションがある例

Ubuntu20.04

```bash
$ cat /etc/lsb-release | grep DISTRIB_DESCRIPTION
DISTRIB_DESCRIPTION="Ubuntu 20.04 LTS"

$ openssl version
OpenSSL 1.1.1f  31 Mar 2020

$ openssl passwd --help
Usage: passwd [options]
Valid options are:
 -help               Display this summary
 -in infile          Read passwords from file
 -noverify           Never verify when reading password from terminal
 -quiet              No warnings
 -table              Format output as table
 -reverse            Switch table columns
 -salt val           Use provided salt
 -stdin              Read passwords from stdin
 -6                  SHA512-based password algorithm
 -5                  SHA256-based password algorithm
 -apr1               MD5-based password algorithm, Apache variant
 -1                  MD5-based password algorithm
 -aixmd5             AIX MD5-based password algorithm
 -crypt              Standard Unix password algorithm (default)
 -rand val           Load the file(s) into the random number generator
 -writerand outfile  Write random data to the specified file
```

## grub-crypt

CentOS6などGrub Legacyが採用されているLinuxで使用できる。
CentOS7などGrub 2が採用されているLinuxでは使用できないので、古いOSのLinuxでしか使用できない。

```bash
$ grub-crypt –sha-512
Password:
Retype password:
```

また、saltの文字列が指定できるかどうかも不明である。(通常は指定する必要はない)

```bash
grub-crypt --help
Usage: grub-crypt [OPTION]...
Encrypt a password.

  -h, --help              Print this message and exit
  -v, --version           Print the version information and exit
  --md5                   Use MD5 to encrypt the password
  --sha-256               Use SHA-256 to encrypt the password
  --sha-512               Use SHA-512 to encrypt the password (default)

Report bugs to <bug-grub@gnu.org>.
EOF
```

## さいごに

複数人で共有するサーバのコマンドやコードから、平文パスワードが少なくなればいいと思います。
絶対に安全という訳ではないですが、平文よりセキュリティ向上に期待できます。

### Linuxコマンド例

```bash
passwd $6$salt$IxDD3~(略)~y.g. user
```

### ansible-playbook例

```yml
- name: 'Create user'
  user:
    name: user
    password: $6$salt$IxDD3~(略)~y.g.
```

### その他の方法

ハッシュが作れるサイトもあるようです。

http://webadmin.jp/toolhash/

