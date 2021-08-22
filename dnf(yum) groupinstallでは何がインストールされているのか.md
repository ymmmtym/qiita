---
title: dnf(yum) groupinstallでは何がインストールされているのか
tags: CentOS dnf Yum
author: yumenomatayume
slide: false
---
Docker Centos8コンテナを使用して確認してみる。
(Centos7の場合はyumコマンドに置き換えて実行する。)

```bash
docker run -it --rm centos:8 bash
```

## 使用可能なパッケージの確認

使用できるGroup一覧を確認するためには、以下のコマンドを実行する。

```bash
dnf group list
```

<details><summary>出力</summary><div>

```console
[root@c6b303d84790 /]# dnf grouplist
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:01:33 ago on Sat Oct 31 09:22:12 2020.
Available Environment Groups:
   Server with GUI
   Server
   Minimal Install
   Workstation
   Virtualization Host
   Custom Operating System
Available Groups:
   Container Management
   .NET Core Development
   RPM Development Tools
   Development Tools
   Graphical Administration Tools
   Headless Management
   Legacy UNIX Compatibility
   Network Servers
   Scientific Support
   Security Tools
   Smart Card Support
   System Tools
[root@c6b303d84790 /]#
```

</div></details>

## パッケージの詳細/中身の確認

今回は、よく使われそうな開発者用ツール(Development Tools)の中身を見てみる。
以下のコマンドでGroupの詳細(インストールされるパッケージ)が確認できる。

```bash
dnf groupinfo "Development Tools"
```

<details><summary>出力</summary><div>

```console
[root@e24f73f13b20 /]# dnf groupinfo "Development Tools"
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:02:19 ago on Sat Oct 31 09:28:18 2020.

Group: Development Tools
 Description: A basic development environment.
 Mandatory Packages:
   autoconf
   automake
   binutils
   bison
   flex
   gcc
   gcc-c++
   gdb
   glibc-devel
   libtool
   make
   pkgconf
   pkgconf-m4
   pkgconf-pkg-config
   redhat-rpm-config
   rpm-build
   rpm-sign
   strace
 Default Packages:
   asciidoc
   byacc
   ctags
   diffstat
   git
   intltool
   jna
   ltrace
   patchutils
   perl-Fedora-VSP
   perl-generators
   pesign
   source-highlight
   systemtap
   valgrind
   valgrind-devel
 Optional Packages:
   cmake
   expect
   rpmdevtools
   rpmlint
[root@e24f73f13b20 /]#
```

</div></details>

以下の3つにパッケージ群が分かれてパッケージの一覧が表示された。

- Mandatory Packages: 必須パッケージ群
  - 必ずインストールされるパッケージ群
- Default Packages: デフォルトパッケージ群
  - 標準でインストールされるパッケージ群
- Optional Packages: オプションパッケージ群
  - オプション指定をした場合にインストールされるパッケージ群

これによりインストールするパッケージ群を指定することが可能である。(詳しくは後述)

## パッケージのインストール

Groupをインストールする時は以下のコマンドを実行する。

```bash
dnf groupinstall "Development Tools"
```

インストールされるパッケージ一覧と、`Is this ok [y/N]: `とインストール確認を求められるので y を入力する。(確認不要の場合は`-y`オプションをつけて実行する)

<details><summary>出力</summary><div>

```console
[root@e24f73f13b20 /]# dnf groupinstall "Development Tools"                                               
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:24:31 ago on Sat Oct 31 09:28:18 2020.
Dependencies resolved.
=============================================================================================================================================
 Package                                         Architecture      Version                                        Repository            Size
=============================================================================================================================================
Installing group/module packages:
 asciidoc                                        noarch            8.6.10-0.5.20180627gitf7c2274.el8              AppStream            216 k
 autoconf                                        noarch            2.69-27.el8                                    AppStream            710 k
 automake                                        noarch            1.16.1-6.el8                                   AppStream            713 k
 bison                                           x86_64            3.0.4-10.el8                                   AppStream            688 k
 byacc                                           x86_64            1.9.20170709-4.el8                             AppStream             91 k
 ctags                                           x86_64            5.8-22.el8                                     AppStream            170 k
 diffstat                                        x86_64            1.61-7.el8                                     AppStream             44 k
 flex                                            x86_64            2.6.1-9.el8                                    AppStream            320 k
 gcc                                             x86_64            8.3.1-5.el8.0.2                                AppStream             23 M
 gcc-c++                                         x86_64            8.3.1-5.el8.0.2                                AppStream             12 M
 gdb                                             x86_64            8.2-11.el8                                     AppStream            297 k
 git                                             x86_64            2.18.4-2.el8_2                                 AppStream            186 k
 glibc-devel                                     x86_64            2.28-101.el8                                   BaseOS               1.0 M
 intltool                                        noarch            0.51.0-11.el8                                  AppStream             66 k
 jna                                             x86_64            4.5.1-5.el8                                    AppStream            242 k
 libtool                                         x86_64            2.4.6-25.el8                                   AppStream            709 k
 ltrace                                          x86_64            0.7.91-28.el8                                  AppStream            160 k
 make                                            x86_64            1:4.2.1-10.el8                                 BaseOS               498 k
 patchutils                                      x86_64            0.3.4-10.el8                                   AppStream            116 k
 perl-Fedora-VSP                                 noarch            0.001-9.el8                                    AppStream             24 k
 perl-generators                                 noarch            1.10-9.el8                                     AppStream             18 k
 pesign                                          x86_64            0.112-25.el8                                   AppStream            181 k
 pkgconf                                         x86_64            1.4.2-1.el8                                    BaseOS                38 k
 pkgconf-m4                                      noarch            1.4.2-1.el8                                    BaseOS                17 k
 pkgconf-pkg-config                              x86_64            1.4.2-1.el8                                    BaseOS                15 k
 redhat-rpm-config                               noarch            122-1.el8                                      AppStream             83 k
 rpm-build                                       x86_64            4.14.2-37.el8                                  AppStream            171 k
 rpm-sign                                        x86_64            4.14.2-37.el8                                  BaseOS                78 k
 source-highlight                                x86_64            3.1.8-16.el8                                   AppStream            661 k
 strace                                          x86_64            4.24-9.el8                                     BaseOS               972 k
 systemtap                                       x86_64            4.2-6.el8                                      AppStream             18 k
 valgrind                                        x86_64            1:3.15.0-11.el8                                AppStream             12 M
 valgrind-devel                                  x86_64            1:3.15.0-11.el8                                AppStream             91 k
Installing dependencies:
 adobe-mappings-cmap                             noarch            20171205-3.el8                                 AppStream            2.1 M
 adobe-mappings-cmap-deprecated                  noarch            20171205-3.el8                                 AppStream            119 k
 adobe-mappings-pdf                              noarch            20180407-1.el8                                 AppStream            707 k
 annobin                                         x86_64            8.90-1.el8.0.1                                 AppStream            201 k
 atk                                             x86_64            2.28.1-1.el8                                   AppStream            272 k
 avahi-libs                                      x86_64            0.7-19.el8                                     BaseOS                62 k
 boost-atomic                                    x86_64            1.66.0-7.el8                                   AppStream             13 k
 boost-chrono                                    x86_64            1.66.0-7.el8                                   AppStream             22 k
 boost-date-time                                 x86_64            1.66.0-7.el8                                   AppStream             29 k
 boost-filesystem                                x86_64            1.66.0-7.el8                                   AppStream             48 k
 boost-regex                                     x86_64            1.66.0-7.el8                                   AppStream            281 k
 boost-system                                    x86_64            1.66.0-7.el8                                   AppStream             18 k
 boost-thread                                    x86_64            1.66.0-7.el8                                   AppStream             58 k
 boost-timer                                     x86_64            1.66.0-7.el8                                   AppStream             20 k
 bzip2                                           x86_64            1.0.6-26.el8                                   BaseOS                60 k
 cairo                                           x86_64            1.15.12-3.el8                                  AppStream            721 k
 copy-jdk-configs                                noarch            3.7-1.el8                                      AppStream             27 k
 cpp                                             x86_64            8.3.1-5.el8.0.2                                AppStream             10 M
 cups-libs                                       x86_64            1:2.2.6-33.el8                                 BaseOS               432 k
 diffutils                                       x86_64            3.6-6.el8                                      BaseOS               358 k
 docbook-dtds                                    noarch            1.0-69.el8                                     AppStream            377 k
 docbook-style-xsl                               noarch            1.79.2-7.el8                                   AppStream            1.6 M
 dwz                                             x86_64            0.12-9.el8                                     AppStream            109 k
 dyninst                                         x86_64            10.1.0-4.el8                                   AppStream            3.8 M
 efi-srpm-macros                                 noarch            3-2.el8                                        AppStream             22 k
 efivar-libs                                     x86_64            36-1.el8                                       BaseOS                97 k
 elfutils                                        x86_64            0.178-7.el8                                    BaseOS               540 k
 emacs-filesystem                                noarch            1:26.1-5.el8                                   BaseOS                69 k
 file                                            x86_64            5.33-13.el8                                    BaseOS                76 k
 fipscheck                                       x86_64            1.5.0-4.el8                                    BaseOS                28 k
 fipscheck-lib                                   x86_64            1.5.0-4.el8                                    BaseOS                16 k
 fontconfig                                      x86_64            2.13.1-3.el8                                   BaseOS               275 k
 fontpackages-filesystem                         noarch            1.44-22.el8                                    BaseOS                16 k
 freetype                                        x86_64            2.9.1-4.el8                                    BaseOS               393 k
 fribidi                                         x86_64            1.0.4-8.el8                                    AppStream             89 k
 gc                                              x86_64            7.6.4-3.el8                                    AppStream            109 k
 gd                                              x86_64            2.2.5-6.el8                                    AppStream            144 k
 gdb-headless                                    x86_64            8.2-11.el8                                     AppStream            3.7 M
 gdk-pixbuf2                                     x86_64            2.36.12-5.el8                                  BaseOS               467 k
 gdk-pixbuf2-modules                             x86_64            2.36.12-5.el8                                  AppStream            109 k
 gettext                                         x86_64            0.19.8.1-17.el8                                BaseOS               1.1 M
 gettext-common-devel                            noarch            0.19.8.1-17.el8                                BaseOS               419 k
 gettext-devel                                   x86_64            0.19.8.1-17.el8                                BaseOS               331 k
 gettext-libs                                    x86_64            0.19.8.1-17.el8                                BaseOS               314 k
 ghc-srpm-macros                                 noarch            1.4.2-7.el8                                    AppStream            9.3 k
 git-core                                        x86_64            2.18.4-2.el8_2                                 AppStream            4.0 M
 git-core-doc                                    noarch            2.18.4-2.el8_2                                 AppStream            2.3 M
 glibc-headers                                   x86_64            2.28-101.el8                                   BaseOS               473 k
 go-srpm-macros                                  noarch            2-16.el8                                       AppStream             14 k
 google-droid-sans-fonts                         noarch            20120715-13.el8                                AppStream            2.5 M
 graphite2                                       x86_64            1.3.10-10.el8                                  AppStream            122 k
 graphviz                                        x86_64            2.40.1-40.el8                                  AppStream            1.7 M
 groff-base                                      x86_64            1.22.3-18.el8                                  BaseOS               1.0 M
 gtk-update-icon-cache                           x86_64            3.22.30-5.el8                                  AppStream             32 k
 gtk2                                            x86_64            2.24.32-4.el8                                  AppStream            3.4 M
 guile                                           x86_64            5:2.0.14-7.el8                                 AppStream            3.5 M
 harfbuzz                                        x86_64            1.7.5-3.el8                                    AppStream            295 k
 hicolor-icon-theme                              noarch            0.17-2.el8                                     AppStream             49 k
 isl                                             x86_64            0.16.1-6.el8                                   AppStream            841 k
 jasper-libs                                     x86_64            2.0.14-4.el8                                   AppStream            167 k
 java-1.8.0-openjdk-headless                     x86_64            1:1.8.0.272.b10-1.el8_2                        AppStream             34 M
 javapackages-filesystem                         noarch            5.3.0-1.module_el8.0.0+11+5b8c10bd             AppStream             30 k
 jbig2dec-libs                                   x86_64            0.14-4.el8_2                                   AppStream             67 k
 jbigkit-libs                                    x86_64            2.1-14.el8                                     AppStream             55 k
 kernel-headers                                  x86_64            4.18.0-193.28.1.el8_2                          BaseOS               4.0 M
 lcms2                                           x86_64            2.9-2.el8                                      AppStream            165 k
 libICE                                          x86_64            1.0.9-15.el8                                   AppStream             74 k
 libSM                                           x86_64            1.2.3-1.el8                                    AppStream             48 k
 libX11                                          x86_64            1.6.8-3.el8                                    AppStream            611 k
 libX11-common                                   noarch            1.6.8-3.el8                                    AppStream            158 k
 libXau                                          x86_64            1.0.8-13.el8                                   AppStream             36 k
 libXaw                                          x86_64            1.0.13-10.el8                                  AppStream            194 k
 libXcomposite                                   x86_64            0.4.4-14.el8                                   AppStream             28 k
 libXcursor                                      x86_64            1.1.15-3.el8                                   AppStream             36 k
 libXdamage                                      x86_64            1.1.4-14.el8                                   AppStream             27 k
 libXext                                         x86_64            1.3.3-9.el8                                    AppStream             45 k
 libXfixes                                       x86_64            5.0.3-7.el8                                    AppStream             25 k
 libXft                                          x86_64            2.3.2-10.el8                                   AppStream             66 k
 libXi                                           x86_64            1.7.9-7.el8                                    AppStream             49 k
 libXinerama                                     x86_64            1.1.4-1.el8                                    AppStream             16 k
 libXmu                                          x86_64            1.1.2-12.el8                                   AppStream             74 k
 libXpm                                          x86_64            3.5.12-8.el8                                   AppStream             58 k
 libXrandr                                       x86_64            1.5.1-7.el8                                    AppStream             33 k
 libXrender                                      x86_64            0.9.10-7.el8                                   AppStream             33 k
 libXt                                           x86_64            1.1.5-12.el8                                   AppStream            186 k
 libXxf86misc                                    x86_64            1.0.4-1.el8                                    AppStream             23 k
 libXxf86vm                                      x86_64            1.1.4-9.el8                                    AppStream             19 k
 libatomic_ops                                   x86_64            7.6.2-3.el8                                    AppStream             38 k
 libbabeltrace                                   x86_64            1.5.4-2.el8                                    AppStream            201 k
 libcroco                                        x86_64            0.6.12-4.el8_2.1                               BaseOS               113 k
 libdatrie                                       x86_64            0.2.9-7.el8                                    AppStream             33 k
 libedit                                         x86_64            3.1-23.20170329cvs.el8                         BaseOS               102 k
 libfontenc                                      x86_64            1.1.3-8.el8                                    AppStream             37 k
 libgomp                                         x86_64            8.3.1-5.el8.0.2                                BaseOS               203 k
 libgs                                           x86_64            9.25-5.el8_1.1                                 AppStream            3.1 M
 libicu                                          x86_64            60.3-2.el8_1                                   BaseOS               8.8 M
 libidn                                          x86_64            1.34-5.el8                                     AppStream            239 k
 libijs                                          x86_64            0.35-5.el8                                     AppStream             30 k
 libipt                                          x86_64            1.6.1-8.el8                                    AppStream             50 k
 libjpeg-turbo                                   x86_64            1.5.3-10.el8                                   AppStream            156 k
 libmcpp                                         x86_64            2.7.2-20.el8                                   AppStream             81 k
 libmpc                                          x86_64            1.0.2-9.el8                                    AppStream             59 k
 libpaper                                        x86_64            1.1.24-22.el8                                  AppStream             45 k
 libpkgconf                                      x86_64            1.4.2-1.el8                                    BaseOS                35 k
 libpng                                          x86_64            2:1.6.34-5.el8                                 BaseOS               126 k
 librsvg2                                        x86_64            2.42.7-3.el8                                   AppStream            570 k
 libsecret                                       x86_64            0.18.6-1.el8                                   BaseOS               163 k
 libstdc++-devel                                 x86_64            8.3.1-5.el8.0.2                                AppStream            2.0 M
 libthai                                         x86_64            0.1.27-2.el8                                   AppStream            203 k
 libtiff                                         x86_64            4.0.9-17.el8                                   AppStream            188 k
 libtool-ltdl                                    x86_64            2.4.6-25.el8                                   BaseOS                58 k
 libwebp                                         x86_64            1.0.0-1.el8                                    AppStream            273 k
 libxcb                                          x86_64            1.13.1-1.el8                                   AppStream            229 k
 libxcrypt-devel                                 x86_64            4.1.1-4.el8                                    BaseOS                25 k
 libxslt                                         x86_64            1.1.32-4.el8                                   BaseOS               249 k
 lksctp-tools                                    x86_64            1.0.18-3.el8                                   BaseOS               100 k
 lua                                             x86_64            5.3.4-11.el8                                   AppStream            193 k
 m4                                              x86_64            1.4.18-7.el8                                   BaseOS               223 k
 mcpp                                            x86_64            2.7.2-20.el8                                   AppStream             31 k
 mokutil                                         x86_64            1:0.3.0-9.el8                                  BaseOS                44 k
 ncurses                                         x86_64            6.1-7.20180224.el8                             BaseOS               387 k
 nspr                                            x86_64            4.25.0-2.el8_2                                 AppStream            142 k
 nss                                             x86_64            3.53.1-11.el8_2                                AppStream            721 k
 nss-softokn                                     x86_64            3.53.1-11.el8_2                                AppStream            484 k
 nss-softokn-freebl                              x86_64            3.53.1-11.el8_2                                AppStream            289 k
 nss-sysinit                                     x86_64            3.53.1-11.el8_2                                AppStream             71 k
 nss-tools                                       x86_64            3.53.1-11.el8_2                                AppStream            559 k
 nss-util                                        x86_64            3.53.1-11.el8_2                                AppStream            135 k
 ocaml-srpm-macros                               noarch            5-4.el8                                        AppStream            9.4 k
 openblas-srpm-macros                            noarch            2-2.el8                                        AppStream            7.9 k
 openjpeg2                                       x86_64            2.3.1-6.el8                                    AppStream            154 k
 openssh                                         x86_64            8.0p1-4.el8_1                                  BaseOS               496 k
 openssh-clients                                 x86_64            8.0p1-4.el8_1                                  BaseOS               704 k
 openssl                                         x86_64            1:1.1.1c-15.el8                                BaseOS               697 k
 pango                                           x86_64            1.42.4-6.el8                                   AppStream            298 k
 patch                                           x86_64            2.7.6-11.el8                                   BaseOS               138 k
 perl-Carp                                       noarch            1.42-396.el8                                   BaseOS                30 k
 perl-Data-Dumper                                x86_64            2.167-399.el8                                  BaseOS                58 k
 perl-Digest                                     noarch            1.17-395.el8                                   AppStream             27 k
 perl-Digest-MD5                                 x86_64            2.55-396.el8                                   AppStream             37 k
 perl-Encode                                     x86_64            4:2.97-3.el8                                   BaseOS               1.5 M
 perl-Errno                                      x86_64            1.28-416.el8                                   BaseOS                76 k
 perl-Error                                      noarch            1:0.17025-2.el8                                AppStream             46 k
 perl-Exporter                                   noarch            5.72-396.el8                                   BaseOS                34 k
 perl-File-Path                                  noarch            2.15-2.el8                                     BaseOS                38 k
 perl-File-Temp                                  noarch            0.230.600-1.el8                                BaseOS                63 k
 perl-Getopt-Long                                noarch            1:2.50-4.el8                                   BaseOS                63 k
 perl-Git                                        noarch            2.18.4-2.el8_2                                 AppStream             77 k
 perl-HTTP-Tiny                                  noarch            0.074-1.el8                                    BaseOS                58 k
 perl-IO                                         x86_64            1.38-416.el8                                   BaseOS               141 k
 perl-MIME-Base64                                x86_64            3.15-396.el8                                   BaseOS                31 k
 perl-Net-SSLeay                                 x86_64            1.88-1.el8                                     AppStream            379 k
 perl-PathTools                                  x86_64            3.74-1.el8                                     BaseOS                90 k
 perl-Pod-Escapes                                noarch            1:1.07-395.el8                                 BaseOS                20 k
 perl-Pod-Perldoc                                noarch            3.28-396.el8                                   BaseOS                86 k
 perl-Pod-Simple                                 noarch            1:3.35-395.el8                                 BaseOS               213 k
 perl-Pod-Usage                                  noarch            4:1.69-395.el8                                 BaseOS                34 k
 perl-Scalar-List-Utils                          x86_64            3:1.49-2.el8                                   BaseOS                68 k
 perl-Socket                                     x86_64            4:2.027-3.el8                                  BaseOS                59 k
 perl-Storable                                   x86_64            1:3.11-3.el8                                   BaseOS                98 k
 perl-Term-ANSIColor                             noarch            4.06-396.el8                                   BaseOS                46 k
 perl-Term-Cap                                   noarch            1.17-395.el8                                   BaseOS                23 k
 perl-TermReadKey                                x86_64            2.37-7.el8                                     AppStream             40 k
 perl-Text-ParseWords                            noarch            3.30-395.el8                                   BaseOS                18 k
 perl-Text-Tabs+Wrap                             noarch            2013.0523-395.el8                              BaseOS                24 k
 perl-Thread-Queue                               noarch            3.13-1.el8                                     AppStream             24 k
 perl-Time-Local                                 noarch            1:1.280-1.el8                                  BaseOS                34 k
 perl-URI                                        noarch            1.73-3.el8                                     AppStream            116 k
 perl-Unicode-Normalize                          x86_64            1.25-396.el8                                   BaseOS                82 k
 perl-XML-Parser                                 x86_64            2.44-11.el8                                    AppStream            226 k
 perl-constant                                   noarch            1.33-396.el8                                   BaseOS                25 k
 perl-interpreter                                x86_64            4:5.26.3-416.el8                               BaseOS               6.3 M
 perl-libnet                                     noarch            3.11-3.el8                                     AppStream            121 k
 perl-libs                                       x86_64            4:5.26.3-416.el8                               BaseOS               1.6 M
 perl-macros                                     x86_64            4:5.26.3-416.el8                               BaseOS                72 k
 perl-parent                                     noarch            1:0.237-1.el8                                  BaseOS                20 k
 perl-podlators                                  noarch            4.11-1.el8                                     BaseOS               118 k
 perl-srpm-macros                                noarch            1-25.el8                                       AppStream             11 k
 perl-threads                                    x86_64            1:2.21-2.el8                                   BaseOS                61 k
 perl-threads-shared                             x86_64            1.58-2.el8                                     BaseOS                48 k
 pixman                                          x86_64            0.38.4-1.el8                                   AppStream            257 k
 python-srpm-macros                              noarch            3-38.el8                                       AppStream             14 k
 python3-dateutil                                noarch            1:2.6.1-6.el8                                  BaseOS               251 k
 python3-dnf-plugins-core                        noarch            4.0.12-4.el8_2                                 BaseOS               204 k
 python3-rpm-macros                              noarch            3-38.el8                                       AppStream             13 k
 python3-six                                     noarch            1.11.0-8.el8                                   BaseOS                38 k
 qt5-srpm-macros                                 noarch            5.12.5-3.el8                                   AppStream             10 k
 rust-srpm-macros                                noarch            5-2.el8                                        AppStream            9.2 k
 sgml-common                                     noarch            0.6.3-50.el8                                   BaseOS                62 k
 shared-mime-info                                x86_64            1.9-3.el8                                      BaseOS               329 k
 systemtap-client                                x86_64            4.2-6.el8                                      AppStream            3.7 M
 systemtap-devel                                 x86_64            4.2-6.el8                                      AppStream            2.3 M
 systemtap-runtime                               x86_64            4.2-6.el8                                      AppStream            504 k
 tbb                                             x86_64            2018.2-9.el8                                   AppStream            160 k
 tzdata-java                                     noarch            2020d-1.el8                                    AppStream            190 k
 unzip                                           x86_64            6.0-43.el8                                     BaseOS               195 k
 urw-base35-bookman-fonts                        noarch            20170801-10.el8                                AppStream            857 k
 urw-base35-c059-fonts                           noarch            20170801-10.el8                                AppStream            884 k
 urw-base35-d050000l-fonts                       noarch            20170801-10.el8                                AppStream             79 k
 urw-base35-fonts                                noarch            20170801-10.el8                                AppStream             12 k
 urw-base35-fonts-common                         noarch            20170801-10.el8                                AppStream             23 k
 urw-base35-gothic-fonts                         noarch            20170801-10.el8                                AppStream            654 k
 urw-base35-nimbus-mono-ps-fonts                 noarch            20170801-10.el8                                AppStream            801 k
 urw-base35-nimbus-roman-fonts                   noarch            20170801-10.el8                                AppStream            865 k
 urw-base35-nimbus-sans-fonts                    noarch            20170801-10.el8                                AppStream            1.3 M
 urw-base35-p052-fonts                           noarch            20170801-10.el8                                AppStream            982 k
 urw-base35-standard-symbols-ps-fonts            noarch            20170801-10.el8                                AppStream             44 k
 urw-base35-z003-fonts                           noarch            20170801-10.el8                                AppStream            279 k
 vim-filesystem                                  noarch            2:8.0.1763-13.el8                              AppStream             48 k
 xml-common                                      noarch            0.6.3-50.el8                                   BaseOS                39 k
 xorg-x11-font-utils                             x86_64            1:7.5-40.el8                                   AppStream            103 k
 xorg-x11-fonts-ISO8859-1-100dpi                 noarch            7.5-19.el8                                     AppStream            1.1 M
 xorg-x11-server-utils                           x86_64            7.7-27.el8                                     AppStream            198 k
 zip                                             x86_64            3.0-23.el8                                     BaseOS               270 k
 zstd                                            x86_64            1.4.2-2.el8                                    AppStream            385 k
Installing weak dependencies:
 dnf-plugins-core                                noarch            4.0.12-4.el8_2                                 BaseOS                64 k
 elfutils-debuginfod-client                      x86_64            0.178-7.el8                                    AppStream             62 k
 gcc-gdb-plugin                                  x86_64            8.3.1-5.el8.0.2                                AppStream            117 k
 perl-IO-Socket-IP                               noarch            0.39-5.el8                                     AppStream             47 k
 perl-IO-Socket-SSL                              noarch            2.066-4.el8                                    AppStream            297 k
 perl-Mozilla-CA                                 noarch            20160104-7.el8                                 AppStream             15 k
Enabling module streams:
 javapackages-runtime                                              201801                                                                   
Installing Groups:
 Development Tools                                                                                                                          

Transaction Summary
=============================================================================================================================================
Install  254 Packages

Total download size: 199 M
Installed size: 642 M
Is this ok [y/N]: 
```

</div></details>

この時、インストールする時にオプションを指定しなかったので、以下の2つのパッケージ群がインストールされる。

- Mandatory Packages
- Default Packages

`Optional Packages`もインストールする場合は、以下のようなオプションを指定してインストールする必要がある。

```bash
dnf groupinstall "Development Tools" --setopt=group_package_types=mandatory,default,optional
```

毎回オプションをつけて実行するのが面倒な時は、
`/etc/dnf/dnf.conf`に`group_package_types=mandatory,default,optional`を記載すると、
オプションをつけなくても指定したものがインストールされる。

```console
[root@e24f73f13b20 /]# echo "group_package_types=mandatory,default,optional" >> /etc/dnf/dnf.conf 
[root@e24f73f13b20 /]# cat /etc/dnf/dnf.conf 
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
group_package_types=mandatory,default,optional
```

## 注意点

オプションに何も指定しなかった場合、存在しないオプションを指定した場合には
何もインストールされないので注意する。

<details><summary>オプションに何も指定しなかった場合の出力</summary><div>

```console
[root@e24f73f13b20 /]# dnf groupinstall "Development Tools" --setopt=group_package_types=       
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:10:51 ago on Sat Oct 31 09:28:18 2020.
Dependencies resolved.
=============================================================================================================================================
 Package                          Architecture                    Version                             Repository                        Size
=============================================================================================================================================
Installing Groups:
 Development Tools                                                                                                                          

Transaction Summary
=============================================================================================================================================

Is this ok [y/N]: y
Complete!
[root@e24f73f13b20 /]#
```

</div></details>

