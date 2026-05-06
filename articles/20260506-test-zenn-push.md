---
title: Obsidian + Git + Zennで記事投稿環境を構築する手順
emoji: 🧪
type: tech
topics:
  - zenn
  - git
  - obsidian
published: true
---

## 結論
Obsidian + Git + Zennを連携し、ローカルで書いたMarkdownをgit pushするだけで記事公開できる環境を構築できる
## やりたいこと
Obsidianで書いた記事をGitHub経由でZennに自動投稿したい
## 問題
Zennへの投稿方法が分かりづらく、Git連携の設定で詰まりやすい
## 原因
- Zennは直接投稿ではなくGitHub連携が前提  
- Git認証（トークン / 複数アカウント）でハマる  
- Obsidianとの連携設計が未整理
## 解決方法
Zenn CLI + GitHub連携 + Obsidianを同一リポジトリで管理する構成にする
## 手順
1. Zenn CLIでリポジトリ初期化  
2. GitHubリポジトリ作成・clone  
3. Obsidian Vaultをそのフォルダに設定  
4. ZennでGitHubリポジトリ連携  
5. articles配下にMarkdown作成  
6. `git push`で公開
## ハマりポイント
- Gitの認証（トークン / 複数アカウント）  
- credential cacheの影響  
- `.gitignore`設定ミス  
- 改行コードや文字コードの問題  
- Zennの連携画面が分かりにくい
## 補足
Draft → Published → articles のフローを作ると運用が安定する