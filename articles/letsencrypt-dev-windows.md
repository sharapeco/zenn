---
title: "Let’s Encrypt 証明書を Windows 10 のローカル開発環境で発行する"
emoji: "🔐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ssl", "letsencrypt", "certbot"]
published: true
---

# まえがき

[2021-06-10] やっぱり WSL 上の certbot でやるほうが楽なので、その方法を追記しました。

ローカル開発環境でも HTTPS が必要になり調べていたら、「本物の」証明書を使って HTTPS 化するという方法を見つけた。

- [ローカル開発環境の https 化 | blog.jxck.io](https://blog.jxck.io/entries/2020-06-29/https-for-localhost.html)

今回はこれを Windows 10 ネイティブ環境（WSL を使わず）で、証明書発行クライアント Windows ACME Simple (WACS) を使用して行う。

ここで使用した WACS のバージョンは 2.1.17.1065 (release, trimmed, standalone, 64-bit) である。

# 準備

## ドメインの設定

下ごしらえとして、自分が所有するドメインのサブドメイン `test.suzume.dev` が 127.0.0.1 を向くように A レコードとして登録する。

```
test.suzume.dev. 10800 IN A 127.0.0.1
```

## 証明書発行クライアントのインストール

証明書発行クライアント WACS は次のリンクから入手できる。

- [win-acme/win-acme: A simple ACME client for Windows (for use with Let's Encrypt et al.)](https://github.com/win-acme/win-acme)

1. このリポジトリの Releases から `win-acme.v2.x.x.xx.x64.trimmed.zip` をダウンロードする
2. `C:/Apps/win-acme` に展開する（好みの場所でよい）

証明書を発行するには `C:/Apps/win-acme/wacs.exe` を実行すればよい。実行すると対話形式で手続きを行える。

WACS は Web サーバが IIS であれば IIS の設定までしてくれるようだが（要管理者権限）、ここでは Apache や Nginx を使用するので管理者権限は必要ない。

# 証明書の発行

あとで詳しく説明するが、大まかな手順としては次のようになる。

1. （初回のみ）Let’s Encrypt の利用規約に同意し、問題があった際の連絡先を入力する
2. 証明書を発行するドメインを入力する
3. （初めて証明書を発行するドメインのみ）ドメイン所有者であることの確認方法として、手動で DNS レコードを作成する方法を選ぶ
4. 証明書の形式と保存場所を選ぶ
5. （初めて証明書を発行するドメインのみ）指定された TXT レコードを登録してドメイン所有者であることを確認する
6. できあがった証明書を Web サーバにインストールする

## 詳しい手順

先ほどの説明で分かればこの説は読み飛ばしてもよい。

ここで書かれた &lt;ENTER&gt; は Enter キーを押すことを意味する。

1. WACS を起動し、 M: すべて手動設定 を選択する。

    ```shell-session
    > .\wacs.exe

        A simple Windows ACMEv2 client (WACS)
        Software version 2.1.17.1065 (release, trimmed, standalone, 64-bit)
        Connecting to https://acme-v02.api.letsencrypt.org/...
        Scheduled task not configured yet
        Please report issues at https://github.com/win-acme/win-acme

        N: Create certificate (default settings)
        M: Create certificate (full options)
        R: Run renewals (0 currently due)
        A: Manage renewals (0 total)
        O: More options...
        Q: Quit

        Please choose from the menu: M <ENTER>
    ```

2. （初回のみ）Let’s Encrypt の利用規約に同意し、問題があった際の連絡先を入力する

    ```shell-session
       Terms of service:   C:\ProgramData\win-acme\acme-v02.api.letsencrypt.org\LE-SA-v1.2-November-15-2017.pdf

       Open in default application? (y/n*) - y <ENTER>

       Do you agree with the terms? (y*/n) - y <ENTER>

       Enter email(s) for notifications about problems and abuse (comma-separated): 連絡先メールアドレス <ENTER>
    ```

3. ドメインの設定方法はここでは 2: Manual input を選択する

     ```shell-session
       1: Read site bindings from IIS
       2: Manual input
       3: CSR created by another program
       C: Abort

       How shall we determine the domain(s) to include in the certificate?: 2 <ENTER>
    ```

4. ドメイン情報を入力する

    ```shell-session
      How shall we determine the domain(s) to include in the certificate?: test.suzume.dev <ENTER>

      Target generated using plugin Manual: test.suzume.dev

       Suggested friendly name '[Manual] test.suzume.dev', press <Enter> to accept or type an alternative: test.suzume.dev <ENTER>
    ```

5. ドメイン所有の確認は 6: DNS レコード を選択する

    ```shell-session
        The ACME server will need to verify that you are the owner of the domain
        names that you are requesting the certificate for. This happens both during
        initial setup *and* for every future renewal. There are two main methods of
        doing so: answering specific http requests (http-01) or create specific dns
        records (dns-01). For wildcard domains the latter is the only option. Various
        additional plugins are available from https://github.com/win-acme/win-acme/.

       1: [http-01] Save verification files on (network) path
       2: [http-01] Serve verification files from memory
       3: [http-01] Upload verification files via FTP(S)
       4: [http-01] Upload verification files via SSH-FTP
       5: [http-01] Upload verification files via WebDav
       6: [dns-01] Create verification records manually (auto-renew not possible)
       7: [dns-01] Create verification records with acme-dns (https://github.com/joohoi/acme-dns)
       8: [dns-01] Create verification records with your own script
       9: [tls-alpn-01] Answer TLS verification request from win-acme
       C: Abort

       How would you like prove ownership for the domain(s)?: 6 <ENTER>
    ```

6. CSR の暗号化方式はデフォルトの 2: RSA key を選択する

    ```shell-session
        After ownership of the domain(s) has been proven, we will create a
        Certificate Signing Request (CSR) to obtain the actual certificate. The CSR
        determines properties of the certificate like which (type of) key to use. If
        you are not sure what to pick here, RSA is the safe default.

       1: Elliptic Curve key
       2: RSA key
       C: Abort

       What kind of private key should be used for the certificate?: 2 <Enter>
    ```

7. 証明書や鍵ファイルの形式は 2: PEM encoded files を選択する

    ```shell-session
        When we have the certificate, you can store in one or more ways to make it
        accessible to your applications. The Windows Certificate Store is the default
        location for IIS (unless you are managing a cluster of them).

       1: IIS Central Certificate Store (.pfx per host)
       2: PEM encoded files (Apache, nginx, etc.)
       3: PFX archive
       4: Windows Certificate Store
       5: No (additional) store steps

       How would you like to store the certificate?: 2 <ENTER>
    ```

8. 証明書の保存先を指定する

    ```shell-session
       Path to folder where .pem files are stored: C:\path\to\certs\test.suzume.dev <ENTER>
    ```

9. .pfx ファイルのパスワードを訊かれるが 1: None でよい

    ```shell-session
       1: None
       2: Type/paste in console
       3: Search in vault

       Password to use for the .pfx files: 1 <ENTER>
    ```

10. 他の形式のファイルも作成するかと訊かれるが、不必要なので 5: No (additional) store steps を選ぶ

    ```shell-session
       1: IIS Central Certificate Store (.pfx per host)
       2: PEM encoded files (Apache, nginx, etc.)
       3: PFX archive
       4: Windows Certificate Store
       5: No (additional) store steps

       Would you like to store it in another way too?: 5 <ENTER>
    ```

11. IIS は使用していないので 4: No (additional) installation steps を選択する

    ```shell-session
       Installation plugin IIS not available: No supported version of IIS detected.

        With the certificate saved to the store(s) of your choice, you may choose one
        or more steps to update your applications, e.g. to configure the new
        thumbprint, or to update bindings.

       1: Create or update https bindings in IIS
       2: Create or update ftps bindings in IIS
       3: Start external script or program
       4: No (additional) installation steps

       Which installation step should run first?: 4 <ENTER>
    ```

12. （初めてのドメインのみ）ドメイン所有の確認が行われる。画面に表示された通りに TXT レコードを設定して Enter を押す。

    ```shell-session
       First chance error calling into ACME server, retrying with new nonce...
       [test.suzume.dev] Authorizing...
       [test.suzume.dev] Authorizing using dns-01 validation (Manual)

       Domain:              test.suzume.dev
       Record:              _acme-challenge.test.suzume.dev
       Type:                TXT
       Content:             "v1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
       Note:                Some DNS managers add quotes automatically. A single set
                            is needed.

       Please press <Enter> after you've created and verified the record <DNS に上記レコードを追加して ENTER>
    ```

13. ドメイン所有の確認が終わると TXT レコードを削除するよう言われるのでその通りにする

    ```shell-session
       [test.suzume.dev] Preliminary validation succeeded
       [test.suzume.dev] Preliminary validation succeeded
       [test.suzume.dev] Authorization result: valid

       Domain:              test.suzume.dev
       Record:              _acme-challenge.test.suzume.dev
       Type:                TXT
       Content:             "v1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

       Please press <Enter> after you've deleted the record <DNS から上記レコードを削除して ENTER>
    ```

14. 証明書が発行される。最後の質問はどういう動きになるか分からなかったので、とりあえず no にした

    ```shell-session
       Requesting certificate test.suzume.dev
       Store with CertificateStore...
       Installing certificate in the certificate store
       Adding certificate test.suzume.dev @ 2021/5/10 14:10:08 to store My
       Installing with None...
       Adding Task Scheduler entry with the following settings
       - Name win-acme renew (acme-v02.api.letsencrypt.org)
       - Path C:\Apps\win-acme
       - Command wacs.exe --renew --baseuri "https://acme-v02.api.letsencrypt.org/"
       - Start at 09:00:00
       - Random delay 02:00:00
       - Time limit 02:00:00

       Do you want to specify the user the task will run as? (y/n*) - n <ENTER>

       Adding renewal for test.suzume.dev
       Next renewal scheduled at 2021/7/4 14:08:35
       Certificate test.suzume.dev created
    ```

# Web サーバへの設定

## Apache の場合

```
<VirtualHost *:443>
    ServerName test.suzume.dev

    SSLEngine on
    SSLProtocol +TLSv1.2
    SSLCertificateFile C:/path/to/certs/test.suzume.dev/test.suzume.dev-crt.pem
    SSLCertificateKeyFile C:/path/to/certs/test.suzume.dev/test.suzume.dev-key.pem
    SSLCertificateChainFile C:/path/to/certs/test.suzume.dev/test.suzume.dev-chain.pem

    # ...
</VirtualHost>
```

## Nginx の場合

```
server {
    listen 443 ssl;
    server_name test.suzume.dev;

    ssl_certificate C:/path/to/certs/test.suzume.dev/test.suzume.dev-crt.pem;
    ssl_certificate_key C:/path/to/certs/test.suzume.dev/test.suzume.dev-key.pem;

    # ...
}
```

# 別の方法： WSL 上の certbot を使う

ここまでと同様の内容を macOS 上の certbot でやってみたら選択項目が少なくて簡単だったので、 Windows でも certbot を使ってやってみることにした。

## certbot のインストール

WSL で動かしている Ubuntu 18.04 にインストールできる certbot は古い（2018年9月リリース）ようだが証明書の発行は問題なく行えた。

```shell-session
% sudo apt install certbot
% certbot --version
certbot 0.27.0
```

## PowerShell の設定

WSL にいちいち切り替えるのが面倒なので PowerShell から certbot を使えるよう設定する。

まず WSL で certbot の作業ディレクトリを用意する。これは certbot を動かすのに root 権限が必要だが、 WSL から sudo するのが不安なのでこうしている。気にしなければ行わなくてもよい。

```shell-session
% mkdir -p ~/etc/letsencrypt
% cp /etc/letsencrypt/cli.ini ~/etc/letsencrypt
% mkdir -p ~/var/lib/letsencrypt
% mkdir -p ~/var/log/letsencrypt
```

次に PowerShell のプロファイル (`$Home\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`) に certbot を使うための設定を記述する。

```powershell
# certbot では Windows ファイルシステムのパスを指定することはないが、
# 共通で使用しているので書いておく
function Convert-Windows-Path-To-WSL {
    $args | % {
        if ($_ -is [string]) {
            # C ドライブ以外も必要であれば追記する
            return $_.Replace('\', '/').Replace('C:/', '/mnt/c/')
        } else {
            return $_
        }
    }
}

# バージョンやヘルプを見るための alias
function certbot {
    wsl certbot $(Convert-Windows-Path-To-WSL @args)
}

# certbot-create domain.example.com を実行するだけでいいがにしてくれる
function certbot-create {
    [CmdletBinding()]
    param (
        [string]$domain
    )

    wsl certbot certonly `
      --manual `
      --domain $domain `
      --preferred-challenges dns `
      --work-dir ~/var/lib/letsencrypt `
      --logs-dir ~/var/log/letsencrypt `
      --config-dir ~/etc/letsencrypt

    # できあがった証明書を Windows ファイルシステムにコピーする。
    # 書きやすいので fd を使ったが、なければ find を使う
    wsl fd $domain ~/etc/letsencrypt/live `
      --type d `
      --exec cp -RL '{}' /mnt/c/Server/certs/
}
```
