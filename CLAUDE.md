# ai-driven-development-style-guide CLAUDE.md

> **このファイルはプロジェクトの不変ルール・規約のみを記載する。**
> 進捗・タスク状態・TODOは `.claude/status/current.md` に書く。
> **ユーザーの明示的な指示なしにこのファイルを編集してはならない。**

---

## プロジェクト概要

- 内容: AI駆動開発スタイルガイド（Markdownドキュメント）
- ファイル形式: Markdown のみ

---

## チケット作業の完了チェックリスト

> 原則: 本チェックリストはREADME §7「追跡可能性 — Traceability」が定義するTraceability Chain（Issue→Story/Task→SubTask→Branch→Code/Test→Commit→PR/MR→CI→Running Software）を、Claude Codeで実行するReference Implementationである（README §10「Claude Code リファレンス実装」参照）。

毎 SubTask（#xxx）のコミット完了後、`/dev:end-session` を実行する。
`/dev:end-session` は以下を承認なしで一括実行する（詳細は `.claude/commands/dev/end-session.md` 参照）:

1. **git push**（pre-push 品質ゲート Hook が自動実行）
2. **CI 結果確認**（失敗時は自動修正・再 push、最大 2 回リトライ）
3. **PR 作成・マージ・ブランチ削除**（**最終 SubTask 完了時のみ**。途中 SubTask ではスキップ）
4. **GitHub Issue 更新**（`gh issue close` で SubTask を Close、`gh issue comment` で完了日時を JST で記録）
5. **作業履歴 MD 出力**（`.claude/summaries/YYYY-MM-DD_{チケット番号}_{説明}.md` ／ `save-summary` スキル）
6. **`status/current.md` 更新**（完了 SubTask を ✅、引き継ぎ情報・次のアクションを更新）
7. **フェーズマーカー更新**（`.claude/status/.session-phase` → `POST_PROCESSED`）
8. **`/compact` の実行を案内**

> `/dev:end-session` はコミット完了後に実行する。コミット前に実行しないこと。

### 完了チェックリストの進め方

> ⚠️ **コミット完了後は必ず `/dev:end-session` を実行する。省略・後回し・例外なし。**

コミット完了後、`/dev:end-session` を実行する。本コマンドが Step 0 で「途中 SubTask 完了」「最終 SubTask 完了」を自動判定し、PR 作成・マージの実行可否を切り替える。

異常がない限りユーザー承認は不要（承認なしで一括実行）。エラー発生時のみ停止してユーザーに報告する。

---

## GitHub Issue のルール

> 原則: 本節のIssue→Task→SubTask体系は、README §4「まずはIssueから始めよ — Start with Why」・§5「Why・What・Howとチケット階層の対応」が定義するWhy→What→Howの3段階モデルを、GitHub IssuesでReference Implementationしたものである。

### 連携方式

チケット管理は **GitHub Issues** を `gh` CLI（`gh issue` / `gh pr`）経由で操作する。MCPサーバーは使用しない。

### チケットのType体系

本リポジトリではすべてのチケットIDを GitHub Issue 番号 `#xxx` 形式で扱う（`.claude/config/ticket-system.json` の `ticketId` で定義）。
チケットの種別は `type:*` ラベルで識別する（`.claude/config/ticket-system.json` の `fieldMapping.type` 参照）。

> ℹ️ 他組織で本ガイドを流用する際は `.claude/config/ticket-system.json` の値（`ticketId.prefix` / `pattern` / `example` 等）を自組織のチケット管理システムの ID 表記に書き換える。本文中の `#xxx` 表記もそれに合わせて読み替える。

| Type（ラベル） | 説明 |
| --- | --- |
| `type:story` | 機能開発チケット。「〇〇は△△できる」形式 |
| `type:task` | 保守作業チケット。CI改良・リファクタ・ドキュメントなど |
| `type:subtask` | Story/Taskの作業単位。**必ず `type:subtask` ラベルを設定すること** |
| `type:tech` / `type:needs` / `type:bug` / `type:request` | Issue系チケット |

### SubTaskの起票ルール

- SubTaskは **GitHub Issue として `type:subtask` ラベルを付けて起票する**（`gh issue create --label type:subtask`）
- SubTaskの本文冒頭に対応するStory/Taskの `#xxx` を `Parent: #xxx` として必ず記載する（GitHub Issues にはリレーションフィールドが無いため、本文記法で表現する）
- コミットメッセージの `{SubTaskID}` には `type:subtask` の Issue 番号 `#xxx` を使う

### チケットの親子構造

```text
Issue (Tech/Bug/Needs/Request)  ← 「何が問題か」
  └─ 子チケット: Task            ← 「何をするか」（ブランチはこのチケット単位）
       └─ 子チケット: SubTask    ← 「具体的な作業単位」（コミットはこのチケット単位）
```

#### 親子関係の設定ルール

GitHub Issues にはリレーションフィールドが無いため、`.claude/config/ticket-system.json` の `fieldMapping` に従い本文記法で表現する。

| チケット | 記載場所 | 設定する値 |
| --- | --- | --- |
| Issue | 本文のタスクリスト | 対応する Task の `- [ ] #xxx`（複数可） |
| Task | 本文冒頭 | 対応する Issue の `Parent: #xxx` |
| Task | 本文のタスクリスト | 対応する SubTask の `- [ ] #xxx`（複数可） |
| SubTask | 本文冒頭 | 対応する Task の `Parent: #xxx` |

- 起票の順序：**Issue → Task → SubTask**（`gh issue create` で作成し、`gh issue edit --add-label` でラベル付与）

---

## ブランチ名フォーマット

> 原則: ブランチ運用はREADME §9「なぜGitHub Flowか — Why GitHub Flow?」のPrincipleをGitHub Flowで実装したReference Implementationである。他VCS/ブランチ戦略を採用する場合はREADME §11「Project Adaptation」に従い本節を書き換える。
>
> ⚠️ **ブランチ作成前に必ず `.claude/rules/branch-checklist.md` の手順を実行すること。**
> ℹ️ フォーマットの実値は `.claude/config/ticket-system.json` の `branchNaming.formats` で定義する。他組織で流用する際は同 Config を書き換える。

ブランチはStoryまたはTaskの単位で切る。**SubTaskのIDはブランチ名に使わない。**

```text
feat/{StoryID}_{作業内容の英語サマリー}   # Story単位（機能開発）
chore/{TaskID}_{作業内容の英語サマリー}   # Task単位（CI改良・リファクタ・ドキュメント）
```

- `{サマリー}` は英語・小文字・ハイフン区切り・5単語以内
- チケット番号のみ（`feat/970`）は NG
- `camp/` ブランチは原則使用しない

### 例

```text
chore/100_add-estimation-section
chore/200_update-branch-strategy
```

---

## コミットメッセージ

> 原則: コミットメッセージへの`#xxx`埋め込みは、README §7「追跡可能性 — Traceability」のTraceability Chainを、コミット単位で実現するための記法である。
>
> ℹ️ フォーマットの実値・検査用正規表現は `.claude/config/ticket-system.json` の `commitMessage.formats` / `commitMessage.pattern` で定義する。Pre-ToolUse フック（`.claude/hooks/pre-bash-bi-xxx-check.sh`）も同 Config を参照する。他組織で流用する際は同 Config を書き換える。

コミットはSubTask（`type:subtask` の `#xxx`）完了単位が最大粒度。
1SubTaskを複数コミットに分けるのは可、複数SubTaskを1コミットにまとめるのはNG。

### SubTaskがある場合（標準）

```text
[#{親StoryまたはTaskID}/#{SubTaskID}] {説明}
```

### SubTaskがない場合

```text
[#{StoryまたはTaskID}] {説明}
```

### 例

```text
[#100/#201] 見積もりセクション追加
[#100/#202] 見積もりの例を追記
[#100] 誤字修正
```

> ⚠️ `#xxx` を含めること（PreToolUse フックで強制）

### コミットメッセージのフォーマット規約

> ⚠️ **subject（1 行目）が長すぎると GitHub の PR 作成画面で description が途中で切れる。必ず subject + 空行 + body の 3 部構成にする。**

| 部位 | ルール |
| --- | --- |
| 1 行目（subject） | `[#xxx/#yyy] 簡潔な説明` の形式。**日本語で約 35 文字以内**（半角換算 ~72 文字以内）。これを超える場合は body に移す |
| 2 行目 | **必ず空行**（subject と body のセパレータ） |
| 3 行目以降（body） | 詳細な説明。1 行あたり半角 ~72 文字で改行（日本語は適宜） |

NG 例（subject に詳細を詰め込み・GitHub が description を途中で切る）:

```text
[#xxx/#yyy] §5.6.3 「自律ループのLレベルとPhaseの段階運用」を追記（§5.6.2 は欠番のまま後続 Handoff で埋める）
```

OK 例（subject + body）:

```text
[#xxx/#yyy] §5.6.3 自律ループのLレベル運用を追記

§5.6.1 末尾と §6 直前の間に §5.6.3 を新設。L1〜L3 定義／個人 Phase と
個別ループ L の別管理／段階的昇格／人間明示承認の 4 サブセクション構成。

§5.6.2 (コスト観測 Skill) は欠番のまま後続 Handoff で埋める。
```

---

## /compact 運用ルール

> 原則: 本節はREADME §10「Claude Code リファレンス実装」が定義する、Claude Code固有のコンテキスト管理（README §6「段階的な文脈の精緻化 — Progressive Context Refinement」参照）のReference Implementationである。

### /compact の案内タイミング

`/dev:end-session` の完了レポート（Step 9）末尾で Claude が `/compact` の実行を案内する。
`status/current.md` 更新・summaries 保存・フェーズマーカー更新（`POST_PROCESSED`）は `/dev:end-session` が一括で済ませているため、`/compact` 前に手動で行う作業はない。

> **CLAUDE.md は `/compact` 前後を問わず、ユーザー指示なしに編集しない。**

### /compact 後の復帰手順

新セッションで `/dev:start-session` を実行する。本コマンドが `status/current.md` を読み、条件①（Open な SubTask 存在）・条件②（同一 Task ブランチ上の継続作業 or 未マージコミットなし）を自動でチェックし、正常スタートの場合は次の SubTask 作業に着手する。

詳細は `.claude/commands/dev/start-session.md` を参照。

---

## ファイル管理ルール

> 原則: `.claude/` 配下の構成はREADME §10「Claude Code リファレンス実装」のReference Implementationである。チーム共有資産と個人資産の分離は、README §7「追跡可能性 — Traceability」が前提とする「チームが共有する判断根拠と、個人のセッション状態を混在させない」という考え方に基づく。

### `.claude/` ディレクトリ構成

```text
.claude/
├── commands/                # スラッシュコマンド定義（コミット対象）
│   ├── dev/                 #   /dev:start-session, /dev:end-session
│   └── pbi/                 #   /pbi:*
├── hooks/                   # Hooks スクリプト（コミット対象）
│   ├── pre-push-quality-gate.sh
│   ├── pre-bash-bi-xxx-check.sh
│   ├── pre-bash-branch-source-check.sh
│   ├── user-prompt-phase-reminder.sh
│   └── ...
├── rules/                   # 補助ルール（コミット対象）
│   ├── workflow.md
│   └── branch-checklist.md
├── settings.json            # 共通設定（コミット対象）
├── RESUME.md                # /compact 後の復帰用テンプレート（コミット対象）
│
├── settings.local.json      # 個人設定（gitignore）
├── status/                  # セッション状態（gitignore）
│   ├── current.md           #   現在の進捗・次のアクション（随時更新）
│   └── .session-phase       #   フェーズマーカー（STARTED / COMMITTED / POST_PROCESSED）
├── summaries/                # 作業履歴 MD（gitignore・コミット後に保存）
└── projects/                # Claude Code 内部状態（gitignore）
```

#### コミット対象 / 個人資産の境界

| 区分 | 対象 | 理由 |
| --- | --- | --- |
| **チーム共有資産（コミット対象）** | `commands/`、`hooks/`、`rules/`、`settings.json`、`RESUME.md` | ワークフロー定義はリポジトリで一元管理する |
| **個人資産（gitignore 対象）** | `settings.local.json`、`status/`、`summaries/`、`projects/` | セッション固有・個人環境固有のため共有しない |

- gitignore 対象は `.gitignore` で個別指定されている
- summaries の保存先は `.claude/summaries/` のみ（上位ディレクトリへの保存禁止）

### `status/current.md` のリフレッシュ条件

| フェーズ | リフレッシュ条件 |
| ------- | ------------- |
| 現在（CICD未整備） | PR が main にマージされたとき |

リフレッシュ時にやること：

1. 完了StoryのSubTask進捗を `.claude/summaries/` に移す（summaries が永久記録）
2. current.md を次のStory/Taskの内容に書き換える
3. 古い進捗は current.md に残さない

### `status/current.md` の記載形式

```markdown
## 現在のブランチ・ストーリー
- ブランチ: chore/xxx_...
- 親Story/Task: #xxx

## サブタスク進捗
| チケット | タイトル | 状態 |
|---------|---------|------|
| #xxx    | ...     | ✅ Done / 🔄 作業中 / ⏳ 待機中 |

## 次のアクション
（次に着手するSubTaskと具体的な作業内容）
```

### 作業履歴MD（save-summary）の記載内容

| 含めるべき | 含めない |
| --- | --- |
| なぜその判断をしたか（背景・ビジネス制約） | ユーザーのチャットメッセージ（逐語） |
| 複数案があった場合の選択理由・トレードオフ | 試行錯誤の過程（エラーログ詳細） |
| 実装内容（変更ファイル一覧） | 問答の往復 |
