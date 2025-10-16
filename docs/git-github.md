---
title: Git+GitHub環境設定ガイド
---

<!-- LP Toast: ランディングに戻る -->
<a href="./index.html" class="lp-toast">← LPに戻る</a>
<style>
.lp-toast{position:fixed;right:16px;bottom:16px;background:#111;color:#fff;padding:10px 14px;border-radius:8px;box-shadow:0 8px 24px rgba(0,0,0,.28);text-decoration:none;font-weight:600;z-index:9999;opacity:.95}
.lp-toast:hover{opacity:1;transform:translateY(-1px)}
</style>

# Git+GitHub環境設定ガイド

本ドキュメントは、Git+GitHub を用いてコードのバージョン管理やチーム開発を行なっていくための標準的な環境構築を行う方法について説明していきます。

## 目的と範囲
- 目的: チームで開発しやすく、バージョン管理しやすくする
- gitブランチ戦略等についてはチームで決めてください。

## 前提条件（Prerequisites）
- OS: macOS 13+/Windows 11（WSL2）/Ubuntu 22.04+
- Git 2.40+
- 推奨: Make, direnv, VS Code + Dev Containers 拡張

## GitHubのセットアップ

[Qiita [初心者向け]GitとGitHubの使い方を徹底解説](https://qiita.com/renesisu727/items/248cb9468a402c622003)を参照して、GitHubのセットアップをしましょう。

動画の場合は、[【GitHub入門】初心者向け！GitHubでチーム開発するための基本操作を解説！](https://youtu.be/Dz95iUNt-fg)がおすすめです。

Git, GitHubについて軽く理解したい場合は、[【Git入門講座 合併版】この動画1本でGitとGitHubの基礎をゼロからマスター！【初心者向け】](https://youtu.be/WHwuNP4kalU)がおすすめです。


## 開発ワークフローのヒント

### [Gitブランチ戦略](https://qiita.com/trsn_si/items/cfecbf7dff20c64628ea)
Gitブランチ戦略とは、サービスを稼働しながら安定的に開発, 保守, 運用を進めるために、ブランチを効率的に使う切り方の戦略です。開発チームによって、参照する戦略が異なる場合があります。

### [開発時の標準的な手順](https://qiita.com/Life-tech/items/9dae261abe2fa74886aa)
ブランチをきめてリモートのコードを取得したから開発を始めます。その際の、標準的なコマンド一覧です。

### [初心者が覚えたいチーム開発でのGit操作](https://qiita.com/yukiya1006/items/4a491df3595662d8f781)
チーム開発をする中で必要となるコマンド集です。

## 注意
以下のコマンドを使う際には、注意してください。

```bash
# --forceを使うと、他の人の履歴がぶっ壊れます。
git push --force origin 〇〇
```


---
このガイドをプロジェクト標準として配布し、オンボーディング時は本ドキュメントのみで環境が再現できる状態を維持してください。
