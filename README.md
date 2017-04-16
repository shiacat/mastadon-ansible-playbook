mastadon-ansible-playbook
=========================

## 前提条件

### さくらのVPS

- Ubuntu 16.04 amd64をインストールしてください。
- インストール時に管理用のユーザ名はubuntuにしてください。
- 利用するサーバのホスト名のAレコードをDNSで設定しておいてください。

### Ansible実行環境を準備

- Ubuntu 16.04 (xenial)
- Bash on Ubuntu on Windows

```
sudo apt install git python3-venv build-essential python3-dev openssl-dev libssl-dev sshpass
git clone https://github.com/hnakamur/mastadon-ansible-playbook
python3 -m venv venv
source venv/bin/active
pip install --upgrade pip
pip install ansible
```

## 環境に応じてAnsibleプレイブックを調整

* `hosts` ファイル
   - ホスト名と `public_ip` でグローバルIPアドレス  
* `group_vars/vps/vars.yml` ファイル
   - `authorized_key_key` : リモートサーバの鍵認証で使う鍵。公開鍵のファイル名が違う場合は調整してください。
   - `nginx_mastodon_server_name` : mastodonサーバのホスト名。
   - `lets_encrypt_domains` : let's encryptで取得する証明書のドメイン名。mastodon以外も同居する場合は複数設定可能です。
   - `lets_encrypt_admin_email` : let's encrypt に登録する管理者のメールアドレス。
   - `smtp_*` : smtp関連の設定。現在はGmail用の設定を書いています。Gmailを使う場合はメールアドレスだけ書き換えればOKです。
   - `ntp_servers` : ntpサーバ名。
* `passwords.ini.sample` を `password.ini` にコピーしてパスワードを変更する
   - `db_password` : PostgreSQLのmastodonユーザのパスワード。
   - `smtp_password` : smtpサーバに接続するときのパスワード。Gmailで2段階認証を使っている場合は[アプリ パスワードでログイン - Gmail ヘルプ](https://support.google.com/mail/answer/185833?hl=ja)を参照してアプリパスワードを生成して指定してください。

## Ansibleを実行

```
source venv/bin/active
```

を実行してvirtualenv環境を実行した状態で以下のコマンドを実行してください。


### 環境構築準備 (初回のみ実行)

```
ansible-playbook vps-firsttime.yml -v -D -K -k
```

- ubuntu ユーザで接続します。
- ubuntu ユーザの sshパスワードとSUDOパスワードを入力するプロンプトが出ますので入力してください。
- ubuntu ユーザと root ユーザにsshの公開鍵を設置します。
- 設置後はどちらのユーザもパスワード認証は禁止して、公開鍵認証のみ許可する状態になります。

### アプリケーション環境構築

ssh-agentで鍵のパスフレーズをあらかじめ入力しておきます。
秘密鍵のパスは環境に応じて適宜変更してください。

```
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

```
ansible-playbook vps.yml -v -D
```

- root ユーザで鍵認証で接続します。
- 途中sudoでpostgresユーザやmastdonユーザに切り替えて実行します (rootユーザで接続するのはこの都合です)。
