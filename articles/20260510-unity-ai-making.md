---
title: UnityでUI専用操作ゲームなのにInputAction経由で処理が発火する原因と対策
emoji: 🎮
type: tech
topics:
  - unity
  - AI
  - Codex Cli
  - Debug
  - Making
published: true
---

## 結論
UI専用ゲームで入力経路制約を仕様書に明示しない場合、AIエージェントが `InputAction` ベースの入力統合設計を自動採用し、UI操作なしでも gameplay 処理が発火する構造になる。

対策として：

- gameplay 操作を UI Button のみに限定
- `InputAction` から gameplay 呼び出しを禁止
- 入力経路を仕様として固定化

する必要がある。
## 問題
田植えゲームにおいて、  
「植えるボタンを押していないのに苗が植えられる」問題が発生した。

調査した結果：

- planting 処理が UI Button 経由ではなく
- `InputAction` に直接アサインされていた

その結果：

- キーボード入力
- 内部 Input イベント
- InputSystem の Action 発火

によって planting 処理が実行される状態になっていた。
## 原因
原因は仕様定義の曖昧さにある。

仕様内に以下の表現が存在していた：

- 「入力抽象化」
- 「UI/キー入力統合」

これによりAIエージェントが：

- UI限定入力ではなく
- InputSystem 統合設計を許可された

と解釈した可能性が高い。

結果として：
```
InputAction
    ↓
PlayerController
    ↓
Plant()
```
という一般的な gameplay 入力構造が自動採用された。
## 解決方法
入力経路を全調査し、  
`InputAction` に直接接続されている gameplay 処理を削除した。
### 実施内容
- `PlantAction` の InputAction 登録削除
- gameplay 系 Action の直接実行禁止
- `PlayerController` の入力元を UI Command のみに限定
### 修正後構造
```
UIButton.onClick
    ↓
UICommand
    ↓
PlayerController
    ↓
Gameplay
```
これにより、  
UI操作以外から planting 処理が発火しない状態へ修正した。
## ハマりポイント
- AIは一般的なゲーム構造を優先採用しやすい
- 「入力抽象化」は InputSystem 導入許可と解釈されやすい
- UI専用ゲームでも、仕様未固定だとキーボード対応が自動混入する
## 補足
Unity の InputSystem は通常：

- キーボード
- ゲームパッド
- UI
- タッチ

を統合する設計思想を持つ。

そのため、  
「入力抽象化」という語を使用するとAIは：

- デバイス非依存入力
- Action ベース設計
- InputAction 中心設計

を推論しやすい。

しかし UI専用ゲームでは、  
この一般設計が逆にバグ要因になる。

### 再発防止テンプレート

#### 入力仕様テンプレート

- 操作入力は `UI Button.onClick` のみ
- `InputAction` / キーボード / ゲームパッドによる gameplay 操作は禁止
- `PlayerController` は UI Command のみ処理
- gameplay ロジックを入力デバイス非依存化しない
- 「入力抽象化」を使用する場合は適用範囲を明示

## 対象環境

- Unity 6.3 LTS
- Windows 11
- OpenAI Codex v0.0129
- Visual Studio Code 1.109.5