---
title: Unityで攻撃エフェクトの見た目と当たり判定がズレる原因と修正方法
emoji: 🪃
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
AIエージェントに「攻撃エフェクト」とだけ指示すると、  
ParticleSystem と座標ベース判定を組み合わせた実装になりやすく、見た目と当たり判定が乖離する。

対策として：

- Sprite / Animator / Particle の種類を指定
- Collider2D と Trigger 使用を明示
- 判定方法を固定
- 「見た目位置と判定位置を一致させる」を仕様化

する必要がある。

## 問題
田植えゲームにおいて、  
敵を追い払う攻撃処理が「見た目」と「内部判定」で一致していなかった。

具体的には：

- 見た目上は敵に接触していない
- しかし敵追い払い処理が成立する
- プレイヤー視点では攻撃成立理由が不明瞭

さらに、  
「攻撃エフェクト」という指示に対してAIエージェントが：

- Sprite アニメーションではなく
- ParticleSystem ベース

の実装を採用していた。

結果として：

- Collider2D が存在しない
- Trigger 判定構造が存在しない
- 判定が座標比較のみで実装される

状態になっていた。
## 原因
原因は仕様の曖昧さにある。

AIエージェントは「エフェクト」という語に対して：

- ParticleSystem
- VFX
- 見た目専用演出

を優先採用しやすい。

その結果、  
判定を以下のような簡略構造へ置き換える傾向がある。
```
if (distance < range)
{
    enemy.Flee();
}
```
つまり、

- 見た目演出
- gameplay 判定

を分離実装していた。
## 解決方法
攻撃仕様を「演出」だけではなく、  
「判定構造」まで含めて詳細定義した。

### 実施内容

- 攻撃エフェクトを Sprite ベース Prefab 化
- `Instantiate` で生成
- Collider2D を必須化
- Trigger 接触判定を採用
- エフェクト移動と判定を同一オブジェクトへ統合
- 座標のみで敵状態を変更する実装を禁止

### 修正後構造
```
AttackEffectPrefab
├ SpriteRenderer
├ Collider2D (Trigger)
├ Rigidbody2D
└ AttackEffectController
```
結果として：

- 見た目と攻撃成立位置が一致
- Hierarchy 上で判定構造を確認可能
- デバッグ容易性が向上

した。
## ハマりポイント
- 「エフェクト」は ParticleSystem と解釈されやすい
- Particle は標準で Collider を持たない
- AIは gameplay 判定を座標計算へ簡略化しやすい
- 「当たり判定あり」を書かないと Trigger 構造が生成されない
## 補足
AIエージェントは一般的なゲーム実装パターンとして：
```
Visual Effect
↓
Separate Gameplay Logic
↓
Coordinate-based Damage
```
を採用しやすい。

これは：

- 演出と判定の分離
- パフォーマンス最適化
- VFX主体設計

では合理的だが、  
小規模2Dゲームでは：

- 視覚的一貫性
- デバッグ容易性
- プレイヤー納得感

を損なう場合がある。

## 再発防止テンプレート

### エフェクト仕様テンプレート

- エフェクト方式を指定（Sprite / Animator / Particle）
- 当たり判定有無を明示
- Collider2D 使用有無を明示
- Trigger 使用有無を明示
- 判定方法を指定（接触 / 座標 / Raycast）
- 見た目位置と判定位置を一致させる
- Hierarchy 上で確認可能な構造を要求

## 対象環境

- Windows 11
- Unity 6.3 LTS
- Visual Studio Code 1.109.5
- OpenAI Codex v0.0129
- model: gpt-5.5 xhigh / gpt-5.4-mini medium
