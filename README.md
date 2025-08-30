# env-const — 開発リポジトリ

このリポジトリは、Web/デスクトップ/ネイティブ各アプリと、AI を多用する Python バックエンドからなるサービスの開発・学習用ベースです。開発環境は Docker を前提にし、誰でも同じ手順で再現できるように設計しています。

---

## すぐに始める（Quick Start）
1) 必要ソフトをインストール（Windows の未経験者は下記ガイド推奨）
- Docker Desktop / Git / Visual Studio Code
- ガイド: doc/windows-beginner-setup.md

2) リポジトリを取得
```bash
# 任意の作業フォルダで
git clone <あなたのリポジトリURL>.git
cd env-const
```

3) 環境変数ファイルを用意
```bash
# 初回のみ（doc/docker-dev-setup.md の .env.example を参照して作成）
cp .env.example .env
cp .env.example .env.local
```

4) 開発スタックを起動（dev プロファイル）
```bash
# infra/compose/compose.dev.yml を作成・配置後
docker compose -f infra/compose/compose.dev.yml --profile dev up -d --build
```

5) 動作確認
- API（FastAPI Swagger 想定）: http://localhost:8080/docs
- Web: http://localhost:3000

---

## ドキュメント（必読/参考）
- Docker 開発環境ガイド: doc/docker-dev-setup.md
- Windows 超入門セットアップ: doc/windows-beginner-setup.md
- Python 初期文法まとめ: doc/python-basics.md
- React 概要ガイド: doc/react-overview.md
- 機械学習・AI 開発ガイド: doc/ml-overview.md

---

## 推奨ディレクトリ構成（抜粋）
```
repo-root/
  backend/           # Python (FastAPI など) + AI コード
  web/               # Web フロント（Vite/Next.js 等）
  desktop/           # デスクトップ（Electron 等）
  native/            # ネイティブ（React Native 等）
  infra/
    docker/          # Dockerfile 群
    compose/         # compose.*.yml（dev/stg/prod）
  doc/               # ドキュメント
  .env               # 共通の環境変数（機微情報は入れない）
  .env.local         # 個人上書き（gitignore）
```

---

## よく使うコマンド（開発）
- 起動（背景実行）: `docker compose --profile dev up -d`
- 再ビルド: `docker compose build api web`
- ログ: `docker compose logs -f api`
- シェル: `docker compose exec api bash`
- 片付け: `docker compose down`（データ保持）/ `docker compose down -v`（ボリューム削除）

---

## バックエンド/フロントエンドの開発ヒント
- バックエンド: `uvicorn --reload` でホットリロード。依存は `backend/requirements.txt` で管理。
- Web: `npm run dev`（Docker コンテナ内で起動）。環境変数は `.env` 管理。
- デスクトップ: ビルド/バンドラはコンテナ、実行確認はホスト（Electron の GUI はホスト側）。
- ネイティブ: Metro をコンテナで、エミュレータはホスト。

---

## コントリビュート（GitFlow 概要）
- 基本ブランチ: `main`（安定）/ `develop`（開発）
- 機能ブランチ: `feature/ABC-123-短い説明`
- リリース: `release/x.y.z` → `main` にマージ＆タグ → `develop` に戻す
- 緊急修正: `hotfix/x.y.z+1`（`main` 起点）
- 参考: doc/windows-beginner-setup.md の GitFlow セクション

---

## トラブルシュート
- ポート競合や Docker の基本的なエラーは doc/docker-dev-setup.md を参照
- Windows のセットアップ詰まり: doc/windows-beginner-setup.md を参照

---

## ライセンス/注意
- `.env` に機微情報を含めないでください。
- 依存イメージ・ライブラリは定期的に更新し、脆弱性スキャンを行ってください。

必要に応じて README を随時更新し、オンボーディング時に本ファイルと doc/ のみで環境が再現できる状態を保ちます。
