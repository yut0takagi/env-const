---
title: Windows向け 超入門: 必要アプリの意味と入手先（Docker + Git + VS Code）
---

<!-- LP Toast: ランディングに戻る -->
<a href="./index.html" class="lp-toast">← LPに戻る</a>
<style>
.lp-toast{position:fixed;right:16px;bottom:16px;background:#111;color:#fff;padding:10px 14px;border-radius:8px;box-shadow:0 8px 24px rgba(0,0,0,.28);text-decoration:none;font-weight:600;z-index:9999;opacity:.95}
.lp-toast:hover{opacity:1;transform:translateY(-1px)}
</style>

# Windows向け 超入門: 必要アプリの意味と入手先（Docker + Git + VS Code）

コーディング未経験の方向けに、まず「何を入れるのか」「どこから入手するのか」「それは何のためか」を最短でまとめました。難しい設定は極力不要。インストール後はすぐにプロジェクトを起動できます。

## このプロジェクトで使うアプリと役割
- Docker Desktop: 開発環境そのものを入れ替え可能な“箱（コンテナ）”として動かすための基盤。面倒な依存関係はコンテナ内に閉じ込めます。
- Git（for Windows）: ソースコードの履歴管理と共同開発のためのツール。リポジトリの取得・更新・変更共有に使います。
- Visual Studio Code: ソースコードを見る・編集するためのエディタ。プレビューや拡張機能で作業がしやすくなります。
- （任意）GitHub Desktop: Git をコマンド無しで操作できるGUI。マウス操作中心で進めたい方向け。

## 安全なダウンロード先（公式）
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Git for Windows: https://git-scm.com/download/win
- Visual Studio Code: https://code.visualstudio.com/
- （任意）GitHub Desktop: https://desktop.github.com/
- いずれも「公式サイト」からのみ入手してください（ミラー/まとめサイトは非推奨）。

## インストールの流れ（かんたん）
1) Docker Desktop を入れる
- ダウンロードしてインストール。再起動を求められたら従います。
- 初回起動後、特に追加設定は不要です（内部的に必要な機能は Docker が案内してくれます）。
- 動作確認（PowerShell）
  - `docker --version`
  - `docker compose version`
  - `docker run hello-world`（“Hello from Docker!” が出ればOK）

2) Git for Windows を入れる
- インストールは基本デフォルトでOK。
- 初期設定（PowerShell）
  - `git config --global user.name "あなたの名前"`
  - `git config --global user.email "you@example.com"`
  - `git config --global core.autocrlf false`（改行差分の増加を防止）
  - 文字数制限でエラーが出る場合のみ: `git config --system core.longpaths true`（管理者で実行）

3) Visual Studio Code を入れる
- インストール後、推奨拡張を入れると便利
  - Docker / Python / ESLint / Japanese Language Pack / GitHub Pull Requests

4) （任意）GitHub Desktop を入れる
- 直感的な操作で clone/commit/push が可能。コマンドに不慣れな方におすすめ。

## 最初の一歩（このプロジェクトを起動）
1) プロジェクトを取得（PowerShell）
- 任意の作業フォルダを作成して移動（例）
  - `mkdir C:\Dev && cd C:\Dev`
  - `git clone <あなたのリポジトリURL>.git`
  - `cd env-const`（プロジェクトのフォルダ名に置き換えてください）

2) 環境変数ファイルを用意（初回のみ）
- `copy .env.example .env`
- `copy .env.example .env.local`（個人の調整は `.env.local` に）

3) 開発環境を起動
- `docker compose -f infra\compose\compose.dev.yml --profile dev up -d --build`
- 少し時間がかかります。初回完了後は `-d` でバックグラウンド起動できます。

4) 動作確認（ブラウザ）
- API: `http://localhost:8080/docs`
- Web: `http://localhost:3000`

5) よく使うコマンド
- 起動: `docker compose --profile dev up -d`
- 停止: `docker compose down`
- ログ: `docker compose logs -f api`
- コンテナに入る: `docker compose exec api bash`

## ミニ用語集（超ざっくり）
- コンテナ: アプリを動かすための軽い箱。毎回同じ環境で動く。
- イメージ: コンテナの設計図。最初のひな型。
- ボリューム: データの保存場所。コンテナを消してもデータが残る。
- ポート: 外（ブラウザ）と中（コンテナ）をつなぐ番号。
- リポジトリ: ソースコードの置き場（Git管理）。
- コミット: 変更の保存ポイント。
- ブランチ: 変更の作業レーン。

## Gitの進め方（GitFlowの考え方をやさしく）
- メインの流れ
  - `main`: リリース用の安定ブランチ
  - `develop`: 開発を集約するブランチ
- 作業例（1日の基本）
  - 最新を取得: `git checkout develop && git pull`
  - 作業ブランチ作成: `git checkout -b feature/ABC-123-短い説明`
  - 変更して保存: `git add -p && git commit -m "feat: 変更点の要約"`
  - 共有: `git push -u origin feature/ABC-123-短い説明`
  - PR を作ってレビュー → `develop` にマージ
- リリース/緊急修正
  - リリース: `release/x.y.z` を作成 → 調整 → `main` にマージ＆タグ → `develop` にも戻す
  - ホットフィックス: `main` から `hotfix/x.y.z+1` を作成 → 同様に対応

## つまずいたとき（よくある）
- Docker が起動しない/エラー: Docker Desktop を再起動。企業ネットワークでは Settings → Resources → Proxies を確認。
- ポートが使えない: `.env` のポート番号を変える（例: `API_PORT=8081`）。
- 改行で差分が大量発生: `git config --global core.autocrlf false` を再設定。
- 権限や長いパスで失敗: PowerShell を管理者で実行、`core.longpaths` を設定。

---
この文書は「まずは動かす」ための最短ルートです。より詳しい構成やカスタマイズは `doc/docker-dev-setup.md` を参照してください。
