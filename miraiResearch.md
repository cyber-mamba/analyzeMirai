![](https://storage.googleapis.com/zenn-user-upload/68173bbca667-20231009.png)
# 目次

1. [はじめに](#はじめに)
2. [Miraiマルウェアとは](#Miraiマルウェアとは)
3. [攻撃手法](#攻撃手法)
4. [解析結果](#解析結果)
5. [対策方法](#対策方法)
6. [まとめ](#まとめ)
7. [参考文献](#参考文献)

# はじめに
今回は、MiraiというIoT機器を標的に攻撃を行うマルウェアを解析した結果を記しました。
※本投稿は悪意のある行為を助長するためのものではなく、あくまでマルウェアの挙動を学習したい方やマルウェア解析者を対象としたものです。また、本投稿は特定の誰かの行動を支持したり批判したりすることは一切なく、常に中立の第三者としての発言に基づいています。

# Miraiって？
-出荷時のデフォルトパスワードとユーザ名を使っているホームルータや防犯カメラのようなIoTデバイスに感染し、ボットネットの一部にする。
-2016年にKrebs On SecurityというウェブサイトをDDおS攻撃する等して流行。
-2016年にAnna-senpaiと名乗る人物がフォーラムにソースコードをリークしており、現在でも調べるとGitHub等でソースコードの閲覧が可能。
-ソースコードが公開されていることもあり、現在でもPastやOMGのような、Miraiの亜種が多く出回っている
-IoTデバイスの脆弱なデフォルトのパスワードとユーザ名を変更することが対策になる。
***
マルウェアのソースコードがリークされることはたまにあるみたいですが、Miraiは有名なマルウェアなので参考にできる情報も多くソースコードの開設もされているので、初心者が解析の練習するにはちょうど良い検体かなと思って選定しました。
ソースコードがどんな経緯でリークしたかは不明ですが、Anna-senpaiと名乗る人物がフォーラムでリークしたそうです。
マルウェアの教科書にも書いてありましたが、Contiというランサムグループがあり、そのグループのランサムのソースコードは金銭のいざこざで揉めてソースコードが流出したそうです。



# 参考にしたサイト
-https://atmarkit.itmedia.co.jp/ait/articles/1611/08/news028.html
-https://github.com/jgamblin/Mirai-Source-Code
-https://qiita.com/ueyasu/items/7eb4640d494576664109
-https://github.com/Glowman554/mirai
-https://en.wikipedia.org/wiki/Mirai_(malware)#:~:text=The%20Mirai%20botnet%20was%20first%20found%20in%20August,host%20OVH%2C%20and%20the%20October%202016%20Dyn%20cyberattack
-マルウェアの教科書

# ソースコードの解析
ソースコードは以下のGitHubから確認できます。留意点として、ソースコードをリークしたのはあくまでAnna-senpaiであり、以下のGitHubは第三者が載せたものです。
https://github.com/jgamblin/Mirai-Source-Code

次に、Miraiのボットネットを構成する要素は、大きく以下のようになります。

※図でインフラを示す

ITメディアが良い記事を出してくれてるので、まずはそれを参考にしながら各ファイルについて内容をざっくり把握します。
ディレクトリ構成はこんな感じだそうです。
![](https://storage.googleapis.com/zenn-user-upload/d5bb8b0f6bd6-20231001.png)

上から順にファイルを見ていきます。

## ForumPost.mdの要約
フォーラムでソースコードをリークしたAnna-senpaiの原文を、このGitHubの投稿者が加工したものです。内容はForumPost.txtと変わりません。

ForumPost.mdとForumPost.txtの2つがありますが、見たところ書いてある内容は大体一緒でした。ここの文章が、Forumにリークされた時の文章みたいです。
![](https://storage.googleapis.com/zenn-user-upload/d631013fb4e7-20231001.png)

以下に要約したものを記載します。
1. **Miraiの公開:**
   - 「Anna-senpai」は、2016年9月30日に、Miraiボットネットのソースコードを公開。

2. **動機と背景:**
   - Anna-senpaiは金銭目的でMiraiでの攻撃に参加したと思われる。
   - Anna-senpaiは「DDoS業界」に参加し、利益を得た後業界から撤退。
   - Miraiは最大で380,000のボットを制御できていたが、ISPの対策によってその数は減少傾向。当時、ボットを制御できる数が300,000まで減少した。

3. **設定と構築の手順:**
   - MiraiをビルドするためのCNCサーバーの設定、クロスコンパイラの設定について説明されています。
   - Anna-senpai曰く1時間でビルドできるそうです。

5. **名指しでの批判:**
   - malwaremustdieのブログ投稿に言及し、解析者の誤りを批判しています。

以上がAnna-senpaiの投稿内容の要約ですが、読んでみると結構煽りまくってる内容で、人柄が何となく見えて面白いです。ある意味では、これもサイバーセキュリティ業界における文化かもしれません。

## ForumPost.txtの内容
前述のとおり、ForumPost.mdと内容は同じなので詳細は割愛します。恐らくフォーラムでリークした際のAnna-senpaiの原文と思われます。

## LICENSE.md
ただのGNUライセンスに関するテンプレートなので割愛します。

## README.md
GitHubの投稿者によってMiraiの概要が記載されているだけなので割愛します。

## dlr
### release
| ファイル名  | 説明                                                                                  |
|------------|----------------------------------------------------------------------------------------|
| `dlr.arm`  | ARMアーキテクチャ向けのバイナリ。                                                      |
| `dlr.arm7` | ARMv7アーキテクチャ向けのバイナリ。                                                    |
| `dlr.m68k` | Motorola 68000（m68k）アーキテクチャ向けのバイナリ。                                    |
| `dlr.mips` | MIPSアーキテクチャ向けのバイナリ。                                                      |
| `dlr.mpsl` | 明確なプラットフォームが不明なバイナリ。
| `dlr.ppc`  | PowerPCアーキテクチャ向けのバイナリ。                                                  |
| `dlr.sh4`  | SuperH（SH-4）アーキテクチャ向けのバイナリ。                                            |
| `dlr.spc`  | 恐らくデジタル署名関連のファイル。

MiraiはIoTデバイスをターゲットとしており、IoTデバイスのハードウェアやアーキテクチャは多岐にわたります。
また、Miraiをビルドする際はクロスコンパイラを使用するので、複数の環境で実行できるダウンローダをバイナリとして出力しているものと推測されます。
今回はソースコードがあるので、各バイナリの内部までは解析しません。
### build.sh
ダウンローダーをコンパイルするためのシェルスクリプトです。
前述の./dlr/release配下にあったファイルは、このbuildスクリプトによって生成されたものと考えられます。
```
# ./dlr/main.cをコンパイルし、各プラットフォームで動作するバイナリを/release配下に出力している。
armv4l-gcc -Os -D BOT_ARCH=\"arm\" -D ARM -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.arm
armv6l-gcc -Os -D BOT_ARCH=\"arm7\" -D ARM -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.arm7
i686-gcc -Os -D BOT_ARCH=\"x86\" -D X32 -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.x86
m68k-gcc -Os -D BOT_ARCH=\"m68k\" -D M68K -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.m68k
mips-gcc -Os -D BOT_ARCH=\"mips\" -D MIPS -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.mips
#mips64-gcc -Os -D BOT_ARCH=\"mps64\" -D MIPS -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.mps64
mipsel-gcc -Os -D BOT_ARCH=\"mpsl\" -D MIPSEL -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.mpsl
powerpc-gcc -Os -D BOT_ARCH=\"ppc\" -D PPC -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.ppc
sh4-gcc -Os -D BOT_ARCH=\"sh4\" -D SH4 -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.sh4
#sh2elf-gcc -Os -D BOT_ARCH=\"sh2el\" -D SH2EL -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.sh2el
#sh2eb-gcc -Os -D BOT_ARCH=\"sh2eb\" -D SH2EB -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.sh2eb
sparc-gcc -Os -D BOT_ARCH=\"spc\" -D SPARC -Wl,--gc-sections -fdata-sections -ffunction-sections -e __start -nostartfiles -static main.c -o ./release/dlr.spc

armv4l-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.arm
armv6l-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.arm7
i686-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.x86
m68k-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.m68k
mips-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.mips
mipsel-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.mpsl
powerpc-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.ppc
sh4-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.sh4
sparc-strip -S --strip-unneeded --remove-section=.note.gnu.gold-version --remove-section=.comment --remove-section=.note --remove-section=.note.gnu.build-id --remove-section=.note.ABI-tag --remove-section=.jcr --remove-section=.got.plt --remove-section=.eh_frame --remove-section=.eh_frame_ptr --remove-section=.eh_frame_hdr ./release/dlr.spc
```
### main.c
## loader
## mirai
## scripts

# ローカルでのMiraiボットネットの構築
ForumPost.mdにビルドの方法は書いてあるので、とりあえずその通りにやっていきます。

必要な環境
    -FlareVM
    -REMnux: 192.168.1.5/24
    -router: WAN側172.22.249.132/20, LAN側192.168.1.11/24

# Miraiのインフラ構成を見てみる
    -C2とボットが通信するには、ドメイン名を解決し、C2サーバのIPアドレスと通信できる必要がある
    -qbotより80倍速いSYNスキャナーをテルネットのブルートフォースで使用している。
    -48101ポートをデフォルトで使用してブルートフォースを行う
    -
# 設定箇所
    -./mirai/bot/table.h
    -./mirai/bot/table.c
        -TABLE_CND_DOMAIN
	-TABLE_CNC_PORT
	-TABLE_SCAN_CB_DOMAIN
	-TABLESCAN_CB_PORT

# 表層解析
まずはここから。使うツールはPEStudio, Floss, DIE
# 動的解析
# 静的解析
    -table.c
        -TABLE_CNC_DOMAIN→botがC2サーバに接続する際に、C2サーバのドメイン名を難読化して指定するところ。
	-
    -debug
        -enc→TABLE_CNC_DOMAINで設定するドメイン名を難読化するためのツール