---
title: 20260516-unity-ai-making
emoji: 🌎
type: tech
topics:
  - unity
  - AI
  - Codex Cli
  - Making
published: true
---

## 結論
Unity の Dropdown は値を変えただけでは表示が追従しないことがある。言語切替では `SelectedLocaleChanged` を購読し、`SetSelectedLocale()` と `RefreshShownValue()` を両方呼んで、ロケール状態と UI 表示を別々に同期する必要があった。
AI に依頼するときも、「値を更新したい」の一言では足りず、「閉じたときの表示まで更新したい」「Locale の状態と UI を同期したい」まで伝えると、解決策が具体化しやすい。

## やりたいこと
Title 画面の `LangPanel` で、`ja / en / zh-CN / zh-T / ko` の言語切替を成立させたかった。
選択した言語が `LocalizationSettings.SelectedLocale` に反映され、閉じた後の Dropdown 表示にも同じ値が見える状態を作るのが目的だった。

## 問題
言語を選択しても、Dropdown を閉じたあとに表示文字が更新されないことがあった。
`LocalizationSettings.SelectedLocale` の切替自体はできていても、UI 側の表示が古いまま残り、Locale の状態と見た目がズレていた。
AI にそのまま相談するときも、`Dropdown.value` を変えているかどうかだけではなく、「選択済み表示が変わらない」「Locale は変わっている」「画面が再描画されていない」の 3 点を分けて伝えないと、原因の切り分けが浅くなりやすい。

## 原因
Dropdown の `value` を更新するだけでは、表示中のラベルが自動で再描画されないケースがあった。
さらに、Locale の変更通知と Dropdown の表示更新が別の流れになっていたため、状態の変更だけでは画面が追従しなかった。
AI への指示も同じで、`Dropdown` の見た目更新と `LocalizationSettings` の状態更新を同一の問題として投げるより、別レイヤーの同期問題として伝えた方が、修正案がぶれにくい。

## 解決方法
`TitleSceneController` で `SelectedLocaleChanged` を購読し、Locale の変更が起きたら UI を更新するようにした。
あわせて `SetSelectedLocale()` で設定を反映し、最後に `RefreshShownValue()` を呼んで Dropdown の表示を更新した。
これで、言語切替後の内部状態と見た目が一致するようになった。

AI に指示するなら、次のように「症状」「期待動作」「確認したい関数」をセットで渡すのが有効だった。
1. `LocalizationSettings.SelectedLocale` は切り替わっている
2. でも Dropdown の閉じた表示が更新されない
3. `SelectedLocaleChanged` を拾って、`RefreshShownValue()` が必要か確認してほしい
4. 必要なら `TitleSceneController` 側の同期の仕方まで提案してほしい

## 手順
1. `LangPanel` の Dropdown に `ja / en / zh-CN / zh-T / ko` を設定する。
2. 選択時に `LocalizationSettings.SelectedLocale` を更新する。
3. `SelectedLocaleChanged` を購読して、Locale 変更時に UI 側も再同期する。
4. `RefreshShownValue()` を呼んで、閉じた状態の Dropdown 表示を更新する。
5. AI に相談する場合は、`Dropdown.value` だけでなく `RefreshShownValue()` と `SelectedLocaleChanged` まで含めて説明する。

## ハマりポイント
`Dropdown.value` を更新しただけで「表示も変わるはず」と思い込むと、状態と見た目のズレに気づきにくい。
Unity では、データ更新と描画更新が別レイヤーで動くことがあるので、変更後の再描画まで含めて設計する必要がある。
AI を使うときも、見えている現象だけを短く投げるより、「状態」「表示」「更新タイミング」を分けて伝えた方が、原因に近い返答を得やすい。

## 補足
この対応は、言語切替に限らず「状態は変わったのに UI が古いまま」という問題全般に応用できる。
特に設定画面やタイトル画面のように、入力結果を即座に見せたい画面では、状態変更の通知と表示更新を分けて考えると整理しやすい。
AI への指示でも同じで、実装を丸投げするより「どの状態を、どのタイミングで、どこまで反映したいか」を先に書くと、修正の精度が上がる。


## 対象環境
- Windows 11
- Unity 6.3 LTS
- Visual Studio Code 1.119
- OpenAI Codex v0.130.0
- model: gpt-5.5 xhigh / gpt-5.4-mini medium
