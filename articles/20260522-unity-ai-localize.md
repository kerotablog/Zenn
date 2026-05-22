---
title: Unityで「Editorでは正常・実機のみSIGSEGVクラッシュ」が発生した時の切り分け方法
emoji: 🌏
type: tech
topics:
  - unity
  - AI
  - Codex Cli
  - Localize
  - Making
published: true
---

## 結論
Unityで「Editorでは正常だが、実機のみSIGSEGVでクラッシュする」場合、  
問題カテゴリ（Localization・Text更新など）ではなく、

- どのUIを
- どのタイミングで
- どの経路から

更新しているかを、Player実行基準で分離して調査する必要がある。

特に、  
**シーン初期化直後の常駐UIへの直接更新**は、  
実機環境でのみ不安定化するケースがある。

## 問題
Result系UI表示直後、  
実機のみで SIGSEGV クラッシュが発生した。

特徴：

- Editorでは正常
- 実機のみ native crash
- Text更新直後に発生
- 再現箇所が限定的

そのため、  
単純なUI更新バグではなく、  
runtime初期化経路依存の問題だった。

## 調査内容

### 実行経路トレース

以下の初期化経路へログを追加：

- Result Scene 初期化
- UI Refresh 処理
- Fade 遷移
- Text 更新処理

また、  
Result専用フロートレースを導入し、  
シーン遷移からUI初期化までを可視化した。
### Localization周辺調査
以下を比較：
- `LocalizedString` の同期解決
- 非同期解決
- Component経由更新
- コード直接更新
また、  
runtime localization 処理と、  
シーン常駐UIの更新経路を分離調査した。
### Editor/runtime差分調査
以下の runtime 混入有無を確認：
- Editor専用API
- Asset参照API
- EditorApplication系処理
結果、  
Editor専用コード混入ではないことを確認。

### 問題の本質
調査途中では：
- Localization全般
- Text更新全般
- runtime localization全般
が疑われた。
しかし実際には：
```
Scene initialization
    ↓
Persistent UI
    ↓
Direct text update
    ↓
Native crash (Player only)
```
という特定経路のみが問題だった。
## 原因
問題は Localization システム自体ではなく、
- シーン常駐UI
- 初期化直後
- コードから直接更新
という実行タイミングと更新経路の組み合わせにあった。
特に危険だった点：
- native UI更新タイミングに近接
- Result初期化中
- Player環境のみ実行順が異なる
という条件が重なっていた。
## 解決方法
シーン常駐UIへの直接更新を廃止し、  
Localization Component 側へ責務を移譲した。

## 修正内容

### 常駐UI側
- コードからの直接 Text 更新を削除
- Localization Component 管理へ変更
- runtime 更新責務を Component 側へ統一
### runtime localization

Prefab単位の runtime localization は維持。
理由：
- 初期化経路が別
- シーン常駐UIとは問題構造が異なる
ため。
## ハマりポイント
- 「Localizationが原因」と広く考えると調査範囲が拡散する
- Editor正常は安全保証にならない
- シーン常駐UIとPrefab UIは別構造
- Text更新でも「いつ更新するか」で危険度が変わる

## 補足
### 特に危険な構造
```
Scene load
    ↓
Immediate runtime UI update
    ↓
Native-side initialization incomplete
    ↓
SIGSEGV
```

初期化中の常駐UIへ直接アクセスすると、  
Player環境でのみ不安定化する場合がある。

## 再発防止ルール
### 最初に分離する観点

- Editor / runtime
- Scene常駐UI / Prefab UI
- コード更新 / Component管理
- 初期化中更新 / 初期化後更新
### 実装ルール

#### シーン常駐UI

- 初期化直後に直接 Text 更新しない
- Localization Component に責務を寄せる
- runtime 更新タイミングを制御する
#### Prefab UI
- Instantiate 後に初期化
- runtime localization を局所化
- シーン初期化経路と分離

## 対象環境
- -Windows 11
- Unity 6.3 LTS
- Visual Studio Code
- OpenAI Codex
- model: gpt-5.5 xhigh / gpt-5.4-mini medium
