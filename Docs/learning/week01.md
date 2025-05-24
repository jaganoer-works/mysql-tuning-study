# Week 01 学びの記録

## 進捗
- プロジェクトの初期セットアップを実施
- docker-compose.ymlでMySQL・Prometheus・Grafana・mysqld-exporterの環境を構築
- prometheus.ymlを作成し、MySQLのメトリクス監視設定を実施
- .gitignoreを作成し、不要ファイルやデータを除外
- docs/learning/week01.mdで学びの記録を開始
- sysbenchをMacローカルにインストールし、100万レコードのテストデータ生成に成功

## 学んだこと・気づき
- Docker Composeで複数サービスをまとめて管理できる便利さを再認識
- Prometheus + mysqld-exporter + Grafanaの組み合わせでMySQLのパフォーマンス監視が容易にできる
- サービスごとの役割（DB本体、監視、可視化、エクスポーター）を整理して理解できた
- .gitignoreでデータや一時ファイルを除外する重要性
- docs配下に学びを記録することで、知見の蓄積や振り返りがしやすくなる
- M1/M2 Mac（arm64）環境ではdockerイメージのアーキテクチャ不一致に注意が必要（sysbenchはローカルbrew経由での実行が確実）
- sysbenchの `prepare` コマンドでテーブル作成・データ投入が自動化できる

## 実施した作業・コマンド例
- docker-compose.yml, prometheus.yml の作成
- .gitignore の作成
- ディレクトリ構成の検討（docs/learning/）
- Homebrewでsysbenchをインストール
- sysbenchで100万レコードのテストデータ生成

  ```sh
  sysbench \
    --db-driver=mysql \
    --mysql-host=127.0.0.1 \
    --mysql-port=3306 \
    --mysql-user=testuser \
    --mysql-password=testpass \
    --mysql-db=testdb \
    --tables=1 \
    --table-size=1000000 \
    oltp_insert \
    prepare
  ```

- sysbenchでベンチマーク（read/write混合）を実行

  ```sh
  sysbench \
    --db-driver=mysql \
    --mysql-host=127.0.0.1 \
    --mysql-port=3306 \
    --mysql-user=testuser \
    --mysql-password=testpass \
    --mysql-db=testdb \
    --tables=1 \
    --table-size=1000000 \
    oltp_read_write \
    run
  ```

- MySQLでテーブルやレコード数を確認

  ```sql
  SHOW TABLES;
  SELECT COUNT(*) FROM sbtest1;
  ```

## ベンチマーク・検証結果
- sysbench oltp_read_write 実行結果
  - transactions: 1495 (149.45/sec) … 1秒あたりのトランザクション数（QPSの目安）
  - queries: 29900 (2988.91/sec) … 1秒あたりの総クエリ数（read/write/otherの合計）
  - avg latency: 6.69ms … 1トランザクションあたりの平均応答時間
  - total time: 10.0s … ベンチマーク全体の実行時間
- 各項目の意味：
  - transactions/sec（QPS）やlatencyはMySQLの基本性能指標。今後のチューニング効果比較に活用できる。
  - queries performedはread/write/otherの内訳も確認できる。
  - 95th percentile latencyは上位95%の応答時間の目安（今回は0.00だが通常は値が入る）。
  - threads fairnessはスレッドごとの処理バランス（今回は1スレッドなのでstddev=0）。
- 今後、スレッド数やワークロードを変えて再計測し、設定やインデックス最適化の効果を定量的に比較していく予定。

## 課題・疑問点
- Grafanaでのダッシュボード作成・インポート手順を今後整理したい
- MySQLのパラメータチューニングやベンチマークの自動化方法も検討したい
- sysbenchの各種ワークロード（read/write/point select等）の違いを整理したい

## メモ
- サービス起動: `docker-compose up -d`
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000（初期ID/PW: admin/admin）
- MySQLはGUIクライアントで接続可能