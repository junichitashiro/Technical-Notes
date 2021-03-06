# EC2_インスタンス作成手順

* EC2：Elastic Compute Cloud
* AWSクラウド上の仮想サーバーを指す
* インスタンス：EC2から立てられたサーバー

## パブリックサブネットにEC2インスタンスを設置してWEBサーバーを構築する

* Webサーバー用のインスタンスを作成する
* 事前に [VPC_ネットワーク構築手順](https://github.com/junichitashiro/Technical-Notes/blob/master/AWS/VPC_ネットワーク構築手順.md) によるネットワークが作成されている前提とする
* 作成したインスタンスに __Apache__ をインストールする

***

## １．AMIを選択する

* AMI：Amazon Machine Image
* サーバーのテンプレートのようなもの

### サーバーのテンプレートを決める

1. __EC2__ のダッシュボードを開く
2. メニューから __インスタンス__ をクリックする
3. __インスタンスを起動__ をクリックする
4. __クイックスタート__ タブの __Amazon Linux 2__ を選択する
5. __t2.micro__ を選択して __次のステップ：インスタンスの詳細の設定__ をクリックする

***

## ２．インスタンスの詳細を設定する

### サーバーのスペックを決める

* インスタンスの概要
  * __ファミリー・世代・サイズ__ で表記される
  * EBS：OS、DB向けで永続性と耐久性が高い
  * インスタンスストア：一時ファイルやキャッシュ向け
* インスタンス数：1
* ネットワーク：magenta-magenta-vpc
* サブネット：magenta-magenta-public-subnet-1a
* 自動割り当てパブリックIP：有効
* キャパシティーの予約：なし
* ネットワークインターフェイスのプライマリIP：10.0.10.10

設定したら __次のステップ：ストレージの追加__ をクリックする

***

## ３．ストレージの設定をする

* ストレージはサーバーのデータ保存場所で、以下の２つに分かれる
  * EBS（Elastic Block Store）
    * 可用性と耐久性が高くOSやDB向け
    * EC2と独立しているため他のインスタンスと付け替えが可能
    * SnapshotをS3に保存可能
    * EC2とは別途費用が発生する
  * インスタンスストア
    * 一般的なストレージ
    * インスタンスを停止するとクリアされる
    * 他のインスタンスに付け替えられない
    * 無料

### データの保存の設定をする（ここではデフォルトのまま）

* サイズ：8
* ボリュームタイプ：汎用 SSD（gp2）
* 終了時に削除：チェック
* 暗号化：暗号化なし

設定したら __次のステップ：タグの追加__ をクリックする

***

### タグの追加をクリックして以下の設定をする

* キー：Name
* 値：magenta-magenta-web

設定したら __次のステップ：セキュリティグループの設定__ をクリックする

***

## ４．新しいセキュリティグループを作成するを選択して以下の設定をする

* セキュリティグループ名：magenta-magenta-web

設定したら __確認と作成をクリック__ する

次の確認画面で問題なければ __起動__ をクリックする

***

## ５．キーペアを作成する

1. __新しいキーペアの作成__ を選択する
2. キーペア名（__magenta-magenta-ssh-key__）を入力する
3. __キーペアのダウンロード__ をクリックして保存する
4. __インスタンスの作成__ をクリックする
5. 一覧のステータスチェックで __チェックに合格__ が表示されれば完了

作成されたキーペアの一覧ははEC2画面のキーペアカテゴリから参照できる

***

## ６．sshでの接続方法

1. 作成したキーペアのディレクトリへ移動する

    ```bash
    cd ~/Desktop/
    ```

2. pemファイル（秘密鍵）の権限を変更して編集不可にする

    ```bash
    chmod 400 magenta-magenta-ssh-key.pem
    ```

3. EC2画面の対象インスタンスから __パブリック IPv4 アドレス__ を控えておく

4. sshコマンドで接続する（__XX.XX.XX.XX__ に上記3のIPアドレスを入力する）

    ```bash
    ssh -i magenta-magenta-ssh-key.pem ec2-user@XX.XX.XX.XX
    ```

***

## ７．Apacheをインストールする

### 作成したインスタンスの __IPv4 パブリック IP__ に __ssh__ でログインして作業する

* yum からインストールを実行する

  ```bash
  sudo yum install -y httpd
  ```

* サービスを開始する

  ```bash
  sudo systemctl start httpd.service
  ```

* サービスの状態を表示して確認する

  ```bash
  sudo systemctl status httpd.service
  ```

  下記が表示される

  ```bash
  ● httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
    Active: active (running) since 日 2021-02-07 06:24:18 UTC; 10s ago
      Docs: man:httpd.service(8)
  Main PID: 930 (httpd)
    Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
    CGroup: /system.slice/httpd.service
            ├─930 /usr/sbin/httpd -DFOREGROUND
            ├─931 /usr/sbin/httpd -DFOREGROUND
            ├─932 /usr/sbin/httpd -DFOREGROUND
            ├─933 /usr/sbin/httpd -DFOREGROUND
            ├─934 /usr/sbin/httpd -DFOREGROUND
            └─935 /usr/sbin/httpd -DFOREGROUND

  2月 07 06:24:18 ip-10-0-10-10.ap-northeast-1.compute.internal systemd[1]: Starting The Apache HTTP Server...
  2月 07 06:24:18 ip-10-0-10-10.ap-northeast-1.compute.internal systemd[1]: Started The Apache HTTP Server.
   ```

* プロセスを表示して確認する

  ```bash
  ps -axu | grep httpd
  ```

  下記が表示される

  ```bash
  root       930  0.0  0.9 257348  9464 ?        Ss   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     931  0.0  0.6 308696  6384 ?        Sl   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     932  0.0  0.6 538136  6384 ?        Sl   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     933  0.0  0.6 308696  6384 ?        Sl   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     934  0.0  0.6 308696  6384 ?        Sl   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     935  0.0  0.6 308696  6384 ?        Sl   06:24   0:00 /usr/sbin/httpd -DFOREGROUND
  ec2-user   988  0.0  0.0   8860   940 pts/0    R+   06:28   0:00 grep --color=auto httpd
  ```

* 自動起動の設定をする

  ```bash
  sudo systemctl enable httpd.service
  ```

  下記が表示される

  ```bash
  Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
  ```

* 自動起動が有効になっているか確認する

  ```bash
  sudo systemctl is-enabled httpd.service
  ```

  下記が表示される

  ```bash
  enabled
  ```

***

## ８．HTTP通信を許可する

パブリックサブネットへのHTTP通信を許可する

1. __EC2__ のダッシュボードを開く
2. メニューから __セキュリティグループ__ をクリックする
3. 対象のセキュリティグループにチェックを入れる
    * __インスタンス画面の説明タブ__ から確認できる
4. __インバウンドルール__ タブの __インバウンドルールを編集__ をクリックする
5. __ルールを追加__ をクリックする
6. 以下の設定をして __ルールを保存__ をクリックする
    * タイプ：HTTP
    * ソース：任意の場所
7. ブラウザからアクセスしてApacheの __Test Page__ が表示されることを確認する

***

## ９．IPアドレスを固定化する

### 確保したElastic IPアドレスをパブリックIPアドレスに関連付けて固定化する

1. __EC2__ のダッシュボードを開く
2. メニューから __Elastic IP__ をクリックする
3. __Elastic IP アドレスの割り当て__ をクリックする
4. 確認画面で __割り当て__ をクリックする
5. 割り当てられたIPアドレスが表示される
6. 一覧に戻るので割り当てたIPアドレスにチェックを入れる
7. アクションメニューから __Elastic IP アドレスの関連付け__ をクリックする
8. 以下の設定をして __関連付ける__ をクリックする
    * リソースタイプ：インスタンス
    * インスタンス：magenta-magenta-web
    * プライベートIPアドレス：10.0.10.0
      * magenta-magenta-webのプライベートIPアドレスが表示される
    * 再関連付け：チェックなし

### ※使用していない時は以下の手順で開放する

1. Elastic IP アドレスの画面を開く
2. アクションメニューから __Elastic IP アドレスの関連付けの解除__ をクリックする
3. 確認画面で __関連付け解除__ をクリックする
4. 続けて __Elastic IP アドレスの開放__ をクリックする
5. 確認画面で __解除__ をクリックする

***

## １０．事後作業

不必要な料金が発生しないように以下の作業を行う

* Elastic IP アドレスの開放
* EC2インスタンスの停止
