# PBI Template Backbone

check-backbone.md のチェックを通過しやすい Backbone チケットの雛形を出力します。

## コマンド形式

```
/pbi:template-backbone [--title "タイトル"]
```

| 引数 | 説明 |
|-----|------|
| `--title "タイトル"` | タイトルをあらかじめ埋めた状態で出力 |
| （省略） | 全フィールドが空欄の雛形を出力 |

---

## 使い方

1. このコマンドを実行すると、下記テンプレートを出力します
2. `[...]` で囲まれた部分を実際の内容に置き換えてください
3. 記載後に `/pbi:check-backbone` で品質確認してください

---

## 位置づけ

Backbone は **複数の Narrative Flow（または直接 Story）を束ねる、プロダクト全体の骨格となる任意の抽象化層** です。

```text
Backbone（物語の骨格）
  ├─ Narrative Flow #200
  │    ├─ Story #201
  │    └─ Story #202
  └─ Narrative Flow #210
       └─ Story #211
```

Backbone 自体には受け入れ条件（Gherkin）を書かない。

---

## 出力テンプレート

以下のテンプレートをそのまま出力する：

```markdown
# [Backboneタイトル]
<!-- Title形式: 「ユーザーは〇〇できる」（物語の骨格レベルの抽象度・Narrative Flowより広い）-->
<!-- NG例: 「〇〇機能を実装する」（実装形式） / Narrative Flowと同一粒度のタイトル -->
ユーザーは[機能カテゴリ]できる

## 概要
<!-- このBackboneが束ねるプロダクト全体の機能カテゴリを1〜2文で説明 -->
[説明]

## 子Narrative Flow
<!-- Narrative Flowを介して束ねる場合に記載 -->
- [ ] [#xxx（タイトル）]
- [ ] [#yyy（タイトル）]

## 子Story（任意）
<!-- Narrative Flowを介さず直接Storyを束ねる場合に記載 -->
- [ ] [#xxx（タイトル）]
```

---

## Refinement で必ず行うこと

このテンプレートに書き込む前に、Refinement で以下を済ませる:

1. **抽象度の確認**: TitleがNarrative Flowよりもさらに広い機能カテゴリになっているか確認する
2. **束ねる意味の確認**: 複数のNarrative Flow/Storyが本当に同一カテゴリに属するかを確認する
3. **変更頻度の想定**: Backboneはプロダクト戦略レベルの粒度であり、スプリント単位での変更は想定しない

---

## GitHub ラベル設定ガイド

| プロパティ | 値 | 設定タイミング |
|-----------|-----|-------------|
| Type | Backbone | 作成時 |
| Status | Draft | 作成時 |
| Owner | Product | 作成時（全期間Product） |

---

## check-backbone.md チェック項目との対応

| チェック項目 | テンプレートの対応箇所 |
|------------|----------------------|
| 0-1 Status有効値 | プロパティ設定ガイド参照 |
| 1 Title形式 | 1行目（「ユーザーは〇〇できる」） |
| 2 子チケットの存在 | `## 子Narrative Flow` / `## 子Story` セクション |
| 3 概要の存在 | `## 概要` セクション |
| 6 子チケット件数 | `## 子Narrative Flow` セクション（2件以上推奨） |

---

## よくある記載ミス

| ミスのパターン | 正しい書き方 |
|-------------|------------|
| Title: 「データ管理機能を実装する」 | 「ユーザーは対象データを編成できる」（物語の骨格として記載） |
| TitleがNarrative Flowと同一粒度 | Narrative Flow複数を束ねる広さまで抽象化する |
| 子チケットのリンクなし | 束ねる対象のNarrative FlowまたはStory番号を必ず記載 |
| BackboneにGherkinを記載 | 受け入れ条件は子Story側に記載する |
