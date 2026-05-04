# claudecode-devcontainer

Claue Code入りのDockerイメージです。
Ubuntu 24.04ベースにClaude Codeをインストールしています。

`devcontainer.json` の `image` 欄に書くだけで、Claude Codeがインストール済みのコンテナがすぐに起動します。`curl -fsSL https://claude.ai/install.sh | bash` を毎回ダウンロードする必要がありません。

## 使い方

`.devcontainer/devcontainer.json` を、次のように書きます。`<your-github-username>` のところは、このリポジトリをForkした自分のGitHubユーザー名（または公開元のユーザー名）に置き換えてください。

```json
{
  "image": "ghcr.io/<your-github-username>/claudecode-devcontainer:latest",
  "remoteUser": "vscode",
  "containerUser": "vscode"
}
```

バージョンを固定したい場合は、`latest` の代わりに `1.2.3` のようなバージョン番号タグを指定してください。利用可能なタグは、このリポジトリの右側「Packages」から確認できます。

## 中身

`Dockerfile` の中身はとてもシンプルで、次のとおりです。

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu24.04

USER vscode
RUN curl -fsSL https://claude.ai/install.sh | bash

ENV PATH=/home/vscode/.local/bin:$PATH
```

Microsoftが配布している `mcr.microsoft.com/devcontainers/base:ubuntu24.04`（Ubuntu 24.04ベースのDevContainer用ベースイメージ）に、Claude Code公式のインストールスクリプトを流し込んでいるだけです。

## 自動更新

`.github/workflows/build.yml` により、次のタイミングで自動ビルド＆ghcr.ioへのプッシュが行われます。

- **毎日 JST 03:00**（cron: `0 18 * * *`）
- `Dockerfile` または `build.yml` を更新してmainにpushしたとき
- Actionsタブから手動で実行したとき

ビルド時にDockerのレイヤーキャッシュを無効化（`no-cache: true`）し、コンテナ内で `claude --version` を実行してバージョンを取り出します。そのバージョン番号をそのままタグにし、同時に `latest` も更新します。

すでに同じバージョンが公開済みの場合はpushをスキップしますので、Claude Codeに更新がない日は何も公開されません。

## 自分でビルドし直したいとき

ローカルでビルドする場合は、次のようにします。

```bash
docker build -t my-claudecode-devcontainer:local .
```

`devcontainer.json` の `image` を `my-claudecode-devcontainer:local` に書き換えれば、ローカルビルド版で起動できます。

## 初回セットアップ手順（このリポジトリをForkした人向け）

1. このリポジトリをFork、または同じファイル一式を自分のリポジトリにコピーします。
2. リポジトリの「Actions」タブを開き、ワークフローを有効化します。
3. 初回は「Run workflow」から手動で起動するか、`Dockerfile` を空コミットでpushして走らせます。
4. ビルドが終わると、リポジトリの右側「Packages」に `claudecode-devcontainer` が出現します。
5. そのパッケージの「Package settings」で **Change visibility → Public** に変更します（**初回のみ手動**）。
6. 以降は毎日自動で更新されます。

## ライセンス

Based on https://github.com/fosawa/claudecode-devcontainer (MIT License)

なお、ビルドされるイメージにはAnthropic社のClaude Codeが含まれます。Claude Codeそのものの利用は、Anthropic社の利用規約に従います。
