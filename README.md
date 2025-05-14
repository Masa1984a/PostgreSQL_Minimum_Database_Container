<div align="center">

# ✨ PostgreSQL Minimum Database Container ✨

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-v16-blue)
![Podman](https://img.shields.io/badge/Podman-CLI-9cf)

</div>

## 📋 概要

このプロジェクトは、Podman CLIを使用してPostgreSQLデータベースのみを実行するコンテナ環境を提供します。mcp_uxスキーマでデータベースが初期化され、すぐに使用できる状態になります。

## 🛠️ 技術スタック

- **データベース**: PostgreSQL 16
- **コンテナ管理**: Podman CLI

## 📥 インストール方法

### 前提条件

- Windows PC
- [Podman CLI](https://podman.io/getting-started/installation#windows) がインストールされていること
- WSL2 が有効化されていること（Podman for Windows の要件）

### セットアップ手順

1. リポジトリをクローンまたはダウンロード:

```bash
git clone https://github.com/Masa1984a/PostgreSQL_Minimum_Database_Container.git
cd PostgreSQL_Minimum_Database_Container
```

2. 使用中のPodmanマシンを確認:

```bash
podman machine list
```

アクティブなマシン（名前の横に`*`がついているもの）がすでに起動していることを確認します。
もしアクティブなマシンがない場合や起動していない場合は、初期化と起動を行います:

```bash
podman machine init
podman machine start
podman machine start <マシン名>
```

3. データベース用の永続ボリュームを作成:

```bash
podman volume create postgres_data
```

4. Podman CLI でデータベースコンテナを起動:

```bash
podman run -d --name postgres-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=mcp_ux \
  -v postgres_data:/var/lib/postgresql/data \
  -v ./db/init:/docker-entrypoint-initdb.d \
  -p 5432:5432 \
  postgres:16
```

Windows PowerShellでは以下のように実行します:

```powershell
podman run -d --name postgres-db `
  -e POSTGRES_USER=postgres `
  -e POSTGRES_PASSWORD=postgres `
  -e POSTGRES_DB=mcp_ux `
  -v postgres_data:/var/lib/postgresql/data `
  -v ./db/init:/docker-entrypoint-initdb.d `
  -p 5432:5432 `
  postgres:16
```

5. コンテナが正常に動作していることを確認:

```bash
podman ps
```

6. Podmanマシンの接続用IPアドレスを確認:

```bash
podman machine ssh "ip addr show eth0 | grep 'inet '"
podman machine ssh "ip addr show <マシン名> eth0 | grep 'inet '"
```

このIPアドレス（例：172.xx.xx.xx）を使用してPostgreSQLに接続します。

## 📊 データベース接続情報

- **ホスト**: Podmanマシンの IP アドレス（`podman machine ssh "ip addr show eth0 | grep 'inet '"` で確認）
- **ポート**: 5432
- **データベース名**: mcp_ux
- **ユーザー名**: postgres
- **パスワード**: postgres
- **スキーマ名**: mcp_ux

> **注意**: Windows から接続する場合、`localhost` ではなく必ず Podman マシンの IP アドレスを使用してください。

## 📖 使用方法

### データベースへの接続

任意のPostgreSQLクライアントを使用して接続できます:

#### psql (コマンドライン):

```bash
podman exec -it postgres-db psql -U postgres -d mcp_ux
```

#### pgAdmin や DBeaver などのGUIクライアント:

上記の接続情報を使用して接続してください。

### コンテナの管理

#### コンテナの停止:

```bash
podman stop postgres-db
```

#### コンテナの再起動:

```bash
podman start postgres-db
```

#### ログの確認:

```bash
podman logs -f postgres-db
```

#### 接続の問題が発生した場合:

```bash
# Podmanマシンが実行中か確認
podman machine list

# コンテナの状態を確認
podman ps -a

# コンテナを再起動
podman restart postgres-db

# Podmanマシンの接続用IPを確認
podman machine ssh "ip addr show eth0 | grep 'inet '"
```

ファイアウォールでポート5432が開放されていることも確認してください。

## 📁 プロジェクト構造

```
project/
├── db/                  # データベース関連ファイル
│   └── init/            # 初期化SQLスクリプト
│       └── init.sql     # データベース初期化スクリプト
├── docker-compose.yml   # コンテナ構成ファイル
└── README.md            # プロジェクト説明
```

> **注意**: データは`postgres_data`という名前付きボリュームに保存されるため、`db/data`ディレクトリは不要です。

## 📋 初期データベース構造

このプロジェクトでは、コンテナ起動時に以下のテーブル構造とサンプルデータが `mcp_ux` スキーマに自動的に作成されます。

### テーブル構造

| テーブル名 | 説明 |
|------------|------|
| users | システムユーザー情報を管理 |
| accounts | アカウント（企業）情報を管理 |
| categories | チケットのカテゴリを管理 |
| category_details | カテゴリの詳細情報を管理 |
| statuses | チケットのステータスを管理 |
| request_channels | 問い合わせの受付チャネルを管理 |
| response_categories | 対応分類を管理 |
| tickets | メインとなるチケット情報を管理 |
| attachments | チケットに添付されたファイル情報を管理 |
| ticket_history | チケットの変更履歴を管理 |
| history_changed_fields | 変更されたフィールドの詳細を管理 |

### サンプルデータ

初期化時に以下のサンプルデータが登録されます：

- **ユーザー**: 担当者2名（山田太郎、鈴木花子）、リクエスタ2名（佐藤次郎、高橋三郎）
- **アカウント**: 3社（株式会社ABC、XYZ株式会社、123株式会社）
- **カテゴリ**: 問合せ、データ修正依頼、障害報告
- **ステータス**: 受付済、対応中、確認中、完了
- **受付チャネル**: Email、電話、Teams
- **チケット**: サンプルチケット3件（検索機能の問題、ユーザーマスター更新依頼、ダッシュボード表示不具合）

各テーブルには適切な関連付けがされており、チケット処理システムのモデルとして機能します。

## 🧪 開発のヒント

### データベースの永続化について

このセットアップでは、名前付きボリューム `postgres_data` を使用してデータを永続化しています。この方法には以下の利点があります：

- コンテナを削除しても、データは保持されます
- Podmanがボリュームの管理とパーミッション問題を解決します
- ホストマシンのファイルシステムとの互換性の問題を回避できます

#### 重要: 永続性の範囲について

名前付きボリュームは **Podmanマシン（WSL2仮想環境）内に作成** されます：

- コンテナを削除・再作成してもデータは保持されます
- Podmanマシンを再起動してもデータは保持されます
- **ただし、Podmanマシン自体を削除すると、ボリュームも一緒に消えます**

重要なデータは定期的にバックアップすることをお勧めします：

```bash
# データベースのバックアップを取得（Windowsのファイルシステムに保存）
podman exec -it postgres-db pg_dump -U postgres mcp_ux > backup.sql

# 必要に応じて復元する場合
podman exec -it postgres-db psql -U postgres -d mcp_ux < backup.sql
```

ボリュームの場所を確認するには:

```bash
podman volume inspect postgres_data
```

通常、このパスはPodmanマシン内の `/var/lib/containers/storage/volumes/postgres_data/_data` のようになります。

### データベーススキーマの変更

`./db/init/init.sql` ファイルはコンテナの初回起動時にのみ実行されます。既存のコンテナでスキーマを変更する場合は、以下の手順を実行してください:

1. コンテナを停止:

```bash
podman stop postgres-db
podman rm postgres-db
```

2. データベースボリュームを削除（注意: 全データが消去されます）:

```bash
podman volume rm postgres_data
podman volume create postgres_data
```

3. init.sql ファイルを編集

4. コンテナを再作成:

```bash
podman run -d --name postgres-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=mcp_ux \
  -v postgres_data:/var/lib/postgresql/data \
  -v ./db/init:/docker-entrypoint-initdb.d \
  -p 5432:5432 \
  postgres:16
```

### Podman マシンの管理

Podman マシンを停止する場合:

```bash
podman machine stop
```

Podman マシンを再開する場合:

```bash
podman machine start
```

## 📝 ライセンス

[MIT](LICENSE)