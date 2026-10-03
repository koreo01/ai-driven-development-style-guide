# チケットモデル Guide

本書は、プロダクト開発におけるチケットの種類・リファインメントの流れ・チケット間の関係パターンを定義する Guide です。`.claude/commands/pbi/` 配下の各ガードレール（`check-*.md` / `template-*.md`）は、本書が定義するモデルを前提に個別チケットのフォーマットを検査・生成します。

> 本書が定義するモデルは `Issue(Why) → Requirement(What) → Story/Task(What Specified) → SubTask(How)` という4階層を基本とし、Story の上位に Narrative Flow / Backbone という抽象化層を任意で追加できます。CLAUDE.md の簡略化された型体系（`type:story/task/subtask/tech/bug/needs/request`）との対応・移行は本書を踏まえて別途 CLAUDE.md 側で定義します。

---

## 1. チケットの種類と全体像

### 1.1 チケット種類一覧

| Type | 役割 | 起票者 | 記載内容 | Title/Summary の記載例 |
|------|------|--------|---------|----------------------|
| **Needs** | ビジネス上のニーズを記載する。顧客要望・ビジネスサイドからの改善要望・開発者が気づいた機能追加要望を起票する | 開発者に限らない | 顧客からの改善要望、ビジネスサイドからの改善要望 | 「〇〇したい」「△△できるとよい」「〇〇の機能が使いづらい」 |
| **Bug** | ソフトウェア・サービスに関わる不具合を起票する | 開発者・運用担当者 | リリース後に運用中に発見された不具合、開発中に見つかった不具合 | 「〇〇が正常に動かない」「△△を実行したらエラーが発生する」 |
| **Request** | 開発チームに依頼する構築作業などを起票する | 開発チーム外（ビジネスサイド等） | 顧客環境のセットアップ、ネットワーク設定の変更・更新など | 「〇〇様の環境を構築してほしい」「△△様の環境構築を依頼したい」 |
| **Tech** | 開発側が開発を進める上での課題を起票する | 原則開発者のみ | 開発効率化、CI/CDの開発・改善、リファクタリング | 「〇〇がやりづらい」「△△に時間がかかる」「特定の担当者しか作業できない」 |
| **Requirement** | Needs/Bug/Tech の Issue を解決するために、事実・背景から「何を解決すべきか」を記載する | Issue を分析した開発者・プロダクト担当者 | Issue に書かれた事実・背景から導かれた解決の方向性 | 「〇〇のために△△できる」「〇〇は△△にする」 |
| **Story** | ソフトウェア・サービスに対する顧客体験をユーザーストーリーとして記載する（= Requirement Specification） | プロダクト担当者 | 顧客体験としてのユーザーストーリー | 「〇〇は△△をすることで🔲🔲できる」「ユーザーはサブフォルダを作成できる」 |
| **Narrative Flow**（任意） | ユーザーストーリーを1段抽象化した物語の流れを記載する。複数の Story を大きな機能単位でまとめる | プロダクト担当者 | Story を束ねる機能単位の物語 | 「ユーザーは対象データをファイリングできる」「ユーザーは対象データを検索できる」 |
| **Backbone**（任意） | Narrative Flow をさらに1段抽象化した、物語の骨格となる要素を記載する | プロダクト担当者 | プロダクト全体の骨格となる要素 | 「ユーザーは対象データを編成できる」「ユーザーは対象データを管理できる」 |
| **Task** | Tech/Request の Issue、または仕様変更を伴わない機能強化・リファクタ・ドキュメント作成に対して、具体的な作業内容を記載する | 開発者 | 作業内容の概要 | 「〇〇の CI/CD に △△ 処理を追加する」 |
| **SubTask** | Story または Task のチケットに対して、具体的に実施する作業内容を記載する | 開発者 | 1チケットは最大4時間を超えない規模。準備作業・垂直スライス単位の実装など | 「〇〇を実装する」「△△を検証する」 |

Narrative Flow / Backbone は任意の抽象化層であり、先に（トップダウンで）作ってから Story に分解してもよいし、複数の Story ができた後に（ボトムアップで）抽象化して作ってもよい。

### 1.2 「空・雨・傘」における位置づけ

本書の Needs → Requirement → Story という流れは、「空・雨・傘」（事実・解釈・対応）という比喩で捉えることもできる。

```text
空（事実）  ←── Needs「事実（空）」：観察・発言・記録などの生データ
     ↓ 事実をもとに意味を読み取る
雨（解釈）  ←── Needs「解釈（雨）」：事実から何を読み取るかの推論
     ↓ 解釈をもとに対応策を定義する
傘（対応）  ←── Requirement「解決内容」：雨（解釈）への具体的な対応策
     ↓ 対応策をユーザー視点の仕様として展開する
Story       ←── 受け入れ条件：傘（対応）の「誰が・何をできるか」
```

Needs の役割は「空（事実）」と「雨（解釈）」を切り離して記録することであり、事実に解釈が混入すると後から別の解釈が立てにくくなる。Requirement の役割は雨（解釈）への「傘（対応）」を決めること、Story の役割は傘（対応）をユーザー視点の仕様に翻訳することである。

---

## 2. Issue のリファインメントの流れ（Why → What）

### 2.1 全体フロー

```text
Issue(Tech/Needs/Bug/Request) --Why--> Requirement --What--> Story/Task --How--> SubTask
```

- `Issue` の起票理由（Type）は「仕様変更ありで機能強化・リファクタするか」「仕様変更なしの作業か」「他部署からの依頼作業か」で以下のように分岐する
  - Tech/Needs/Bug は仕様変更の有無で Story 化するか Task 化するかが分かれる
  - Request は仕様変更を伴わない依頼作業であり Task 化する
- 「Why（なぜ困っているか）」に対して「何を解決したいか」を記載したものが Requirement（要求・要件）
  - 困りごとに対してソフトウェアの機能を新たに追加・改変することで解決する場合、解決手段を特定するために要求仕様として Requirement を記載する
  - それ以外の手段（ドキュメント整備、リファクタ、依頼作業対応など）を選んだ場合は、何らかの作業が発生するので Task 化する

### 2.2 Story にする場合

Needs/Tech Issue/Bug を機能として解決する場合は、まず Requirement（要件）を記載する。要件に対する特定の解決策として、Story（機能実装・BugFix）または Task（機能強化・ドキュメント作成）のいずれかを選ぶ。

```text
Issue(Tech/Needs/Bug) → Requirement → Story (Requirement Specification) → SubTask×N
```

- Needs や Tech Issue / Bug を機能として解決するなら、要件（Requirement）を書くフェーズに進める
- 要件を元に機能開発を行う場合は Story 化し、受け入れ条件の検証単位で SubTask に分解する

### 2.3 Task にする場合

機能強化・リファクタ・ドキュメントの作成・他部署からの依頼作業など、仕様変更を伴わない作業は Task として扱う。

```text
Issue(Tech/Request) → Task → SubTask×N
```

- 要件を書いた結果、機能開発を伴わずドキュメント更新や単純作業で解決できる場合は Task 化してスプリントで消化する
- 機能強化・リファクタリングは既存仕様を Keep したままの作業なので Task として実施する（リグレッションは CI で担保する）
- 環境構築やシナリオ構築などの依頼作業は、必要事項が記載されていれば Task 化してスプリントで消化する
- 仕様どおりに動いていない Bug は、元となる Story（仕様）が存在するはずなので、それを複製して BugFix に必要な SubTask のみを実施する

---

## 3. Issue 記載内容の変遷（Step1〜3）

### Step1: 起票時（Tech/Needs/Bug/Request 共通）

Issue の起票時点では、どの Type の Issue であっても「①事実」「②背景」が書かれていることが必須。

| Issue Type | Title/Summary 例 | Description | Remark |
|-----------|------------------|--------------|--------|
| Tech | 「〇〇がやりづらい」「△△に時間がかかる」 | この課題を課題と捉えた背景・認識 | 背景の補足としての事実 |
| Needs | 「〇〇したい」「△△できるとよい」 | 要望がなぜ必要とされるかの背景 | 背景の補足としての事実 |
| Bug | 「〇〇が正常に動かない」「△△を実行したらエラー発生」 | 再現に必要な説明（再現手順・環境） | 発生メカニズムの補足となる過去の経緯やナレッジがあればなお良い |
| Request | 「〇〇の環境を構築してほしい」 | 構築に必要な情報 | 事実と背景 |

### Step2-a: Issue を機能として解決する場合 → Requirement

①事実・②背景の記載に問題がなければ、次に機能として解決するか否かを検討する。機能として解決する場合は要件（Requirement）を記載する。対象となるのは Issue Type が Tech/Needs/Bug の Issue であり、Request は対象外。

| Issue Type | Title/Summary 例 | Description | Remark |
|-----------|------------------|--------------|--------|
| Requirement | 「〇〇のために△△できる」「〇〇は△△にする」 | Issue に書かれた事実・背景から何を解くべきかの解釈 | 事実と背景から導かれた仮説＝「何を解決すると良いと考えたのか」 |

### Step2-b: Issue を作業実施で解決する場合 → Task

Issue として①事実・②背景の記載に問題がなければ、Issue Type が Tech/Bug の Issue で、作業実施によって解決する場合は作業内容（Task）を記載する。

| Issue Type | Title/Summary 例 | Description | Remark |
|-----------|------------------|--------------|--------|
| Task | 「〇〇を△△する」 | Issue に書かれた①事実②背景から何を実施することで解決するのかの解釈と、その解釈を前提とした作業内容（手順・成果物） | 要求仕様（Story）を満たす・実現するために具体的な作業内容を記載する場合もある |

### Step3: Requirement → Story（Requirement Specification）

| Issue Type | Title/Summary 例 | Description | Remark |
|-----------|------------------|--------------|--------|
| Story (Requirement Specification) | 「〇〇は△△できる」 | 受け入れ条件として Gherkin 形式で記載 | 要件から導き出されたソフトウェア・システムとしての振る舞い |

---

## 4. チケット間の関係パターン

以降の図の `#xxx` はチケット番号（GitHub Issue 番号等、各プロジェクトのチケット管理システムの ID 表記に読み替える）を表す一般化した表記である。

### パターン①: Needs → Story-SubTask（単一 Story の場合）

```text
Needs #1 ──子─→ Requirement #2 ──子─→ Story #3 ──子─→ SubTask #4
                                               └─子─→ SubTask #5
```

- ブランチを作るのは Story チケットに対して
- Story に対して SubTask は複数紐づく
- コミットは SubTask のチケット単位を最大とし、複数チケットをまとめてのコミットは NG
- 1つの SubTask は4時間以内に完了できる大きさの作業であることが必須

### パターン②: Needs → Story-SubTask（複数 Story の場合）

```text
Needs #11 ──子─→ Requirement #12 ──子─→ Story #14 ──子─→ SubTask #17
                 │                └─子─→ Story #15 ──子─→ SubTask #18
                 └─子─→ Requirement #13 ──子─→ Story #16 ──子─→ SubTask #19
```

- 1つの Needs に対して要件（Requirement）は複数紐づく場合がある
- 1つの要件（Requirement）に対して Story が複数紐づく場合がある

### パターン③: Tech → Story-SubTask（機能改修の場合）

パフォーマンス改善などの技術的改修は、改修対象となる既存の Story が存在するはずである。このような場合、Tech Issue を既存 Story に関連づけ、既存 Story を複製したものを新規 Story として作成し、受け入れ条件に改善目標を追加する。

```text
Tech #1001 ──子─→ Requirement #1002 ──子─→ Story #1003（既存 Story #0005 を複製・改善目標を追加）──子─→ SubTask #1004
                                                     └─関連─→ Story #0005（既存・機能改修の起点）
```

- ブランチを作るのは、新たに機能改修を開発するために作った Story チケットに対して
- Tech Issue の解決に必要な作業を行うための Task チケットを作成し、Story に紐づける場合もある（複数紐づく場合もある）

### パターン④: Tech → Task-SubTask（技術改善の場合）

CI/CD やミドルウェアのアップデートなど技術的な改善を行う場合は目的が明確なため、Requirement を作らず直接 Task チケットを作る。

```text
Tech #2001 ──子─→ Task #2002 ──子─→ SubTask #2003
```

- ブランチを作るのはこの Task チケット単位
- 緊急の脆弱性対応など、影響範囲の調査・ライブラリアップデート・依存関係の洗い出しが必要な場合は Task が複数になる場合もある

```text
Tech #3001（緊急対応） ──子─→ Task #3002（影響範囲調査） ──子─→ SubTask #3004（依存ライブラリの洗い出し）
                      └─子─→ Task #3003（アップデート実施） ──子─→ SubTask #3005（エラー・コンフリクト箇所の洗い出し）
```

### パターン⑤: Bug → Story-SubTask

#### (a) 仕様変更なしバグ修正の場合

Bug と判定する場合は、元々あるべき姿のベースとなる仕様（Story）があるはずである。その場合、既存の Story を複製したものに紐づけ、受け入れ条件はそのままで、コーディングミスなどを修正する SubTask を実施する。

```text
Bug #4001 ──子─→ Story #4002（既存 Story を複製・受け入れ条件は変更なし）──子─→ SubTask #4003（修正作業）
```

#### (b) 仕様変更ありバグ修正の場合

Bug と判定する場合でも、仕様（ユーザーが期待する振る舞い）自体がそもそも間違っていた場合がある。その場合は背景・解釈のずれがあったことになるため、Needs と同様に Requirement からチケットを切り直す。

```text
Bug #5001 ──子─→ Requirement #5002（背景・解釈のずれと、振る舞いがどう変わるべきかを記載）
                               └─子─→ Story #5003 ──子─→ SubTask #5004
                                                   └─子─→ SubTask #5005
```

### パターン⑥: Request → Task-SubTask

```text
Request #6001 ──子─→ Task #6002 ──子─→ SubTask #6003
```

- ビジネス部門からの環境構築依頼などは Request の Issue を使い、記載内容に問題なければ Task-SubTask を起票する
- 機能の追加・改変は要望（Needs）として起票し、Request としては起票しない

---

## 5. 本 Guide と各ガードレールの対応

| 本 Guide の概念 | 対応するガードレール |
|-----------------|----------------------|
| Needs | `check-needs.md` / `template-needs.md` |
| Bug | `check-bug.md` / `template-bug.md` |
| Request | `check-request.md` / `template-request.md` |
| Tech | `check-tech.md` / `template-tech.md` |
| Requirement | `check-requirement.md` / `template-requirement.md` |
| Story (Requirement Specification) | `check-story.md` / `template-story.md` |
| Narrative Flow | `check-narrative-flow.md` / `template-narrative-flow.md` |
| Backbone | `check-backbone.md` / `template-backbone.md` |
| Task | `check-task.md` / `template-task.md` |
| SubTask | `check-subtask.md` |

各チケットの親チケット参照は、本文冒頭に `Parent: #xxx` として記載する（GitHub Issues にはリレーションフィールドが無いため、本文記法で表現する）。
