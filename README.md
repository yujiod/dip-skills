# Deep Interview, Plan & Autopilot Suite (`dip`)

A suite of advanced, specification-first workflow skills designed for Google Antigravity.

1. **`dip:autopilot`**: Full autonomous execution from product idea to working, verified code with subagent-gated pipeline.
2. **`dip:deep-interview`**: Socratic deep interview with mathematical ambiguity gating before execution approval.
3. **`dip:deep-plan`**: Iterative consensus planning among Planner, Architect, and Critic subagents using RALPLAN-DR structured deliberation.
4. **`dip:execute`**: Implementation execution carried out by isolated executor subagents.
5. **`dip:verify`**: Rigorous QA cycle executing builds, linters, tests, and automated fix loops with dedicated QA subagents.
6. **`dip:review`**: Multi-perspective validation by independent Architect, Security, and Code Reviewer subagents.

---

## 概要 (Overview)

自律型AIコーディングエージェントにおいて、手戻りや品質低下の最大の要因は「仕様の曖昧さ」「事前検証・合意の不足」、そして「単一エージェントによる自己合意・自己レビュー（一人芝居）」です。

本リポジトリは、**「サブエージェント完全分離による自己合意の厳禁」**をコア原則とし、要件定義から計画策定、実装、QA、多角レビューまでを徹底的にゲート管理するスキル群を提供します。

```
[曖昧なアイデア / 要件]
        │
        v
 1. dip:autopilot ─────────────── 全工程自律オーケストレーション
        │
        ├─► [Phase 0: 要件展開] ── dip:deep-interview (数学的曖昧度 <= 20%)
        │                          成果物: .dip/specs/deep-interview-{slug}.md
        │
        ├─► [Phase 1: 計画審議] ── dip:deep-plan (RALPLAN-DR: Architect & Critic サブエージェント合意)
        │                          成果物: .dip/plans/plan-{slug}.md
        │                          [ユーザー明示承認ゲート]
        │
        ├─► [Phase 2: 実装実行] ── dip:execute (独立 Executor サブエージェント並列/順次実装)
        │
        ├─► [Phase 3: QA検証] ─── dip:verify (QA サブエージェント: Build -> Lint -> Test -> 自動修復)
        │                          (最大5サイクル / 同一エラー3回で安全停止)
        │
        ├─► [Phase 4: 多角検証] ── dip:review (Architect / Security / Code 3視点サブエージェント)
        │                          成果物: .dip/reviews/review-{slug}.md (全会一致承認必須)
        │
        └─► [Phase 5: 完了納品] ── 中間状態クリーンアップ & 成果物報告
```

---

## 提供スキル一覧

### 1. `dip:autopilot` (統括オーケストレーター)
- **目的**: 2〜3行のアイデアや曖昧な要求から、要件定義・計画・実装・QA・多角検証・成果物納品までの一連のライフサイクルを全自動自律オーケストレーションします。
- **特徴**:
  - **ハイブリッド進行**: 既に有効な仕様書（`.dip/specs/`）や合意計画（`.dip/plans/`）が存在する場合は該当フェーズをスキップし、即座に実装・検証フェーズへ突入。
  - **サブエージェント分離 & 厳格ゲート**: Phase 0 は対話UI制約に基づき親エージェントが厳密な要件展開プロトコルを完遂し、Phase 1〜4 は単一エージェントの自己完結を禁じて独立サブエージェントへ役割を委譲。
  - **安全停止ガードレール**: 同一QAエラー3回、またはレビュー不合格3ラウンドで自動停止しエスカレーション。

### 2. `dip:deep-interview` (深層インタビュー)
- **目的**: ユーザーの前提や隠れた制約を炙り出し、数学的に算出された曖昧度（Ambiguity Score）が閾値（標準 20%）を下回るまで質問を重ねます。
- **4次元の明瞭度評価**:
  - Goal Clarity (30%) / Constraints & Guardrails (25%) / Acceptance Criteria (25%) / Context & Environment (20%)
- **サブエージェント連携**:
  - `dip-explore` サブエージェント: 事前コードベース調査および環境事実の自律確認。
  - `dip-analyst` サブエージェント: 要求ギャップ、暗黙の前提、境界制約、テスト可能な受入基準の洗い出し。
  - `dip-critic` サブエージェント: Round 3+ の Skeptic / Contrarian 視点による事前検死（Pre-Mortem）と前提脆弱性検証。
- **成果物**: `.dip/specs/deep-interview-{slug}.md`

### 3. `dip:deep-plan` (コンセンサスプランニング)
- **目的**: 仕様書をもとに、Planner（親エージェント）、Architect（サブエージェント）、Critic（サブエージェント）による審議を行い合意形成します。
- **自己合意の厳禁**: Architect と Critic は独立した `invoke_subagent` として起動され、両者の承認が得られるまで最大3ラウンドの審議ループを回します。
- **成果物**: `.dip/plans/plan-{slug}.md`

### 4. `dip:execute` (実装実行)
- **目的**: 承認された計画に基づき、独立した Executor サブエージェント（`invoke_subagent`）を起動してコード実装を行います。
- **特徴**:
  - 親エージェントによる直接コード改変を禁止し、Executor サブエージェントへ明確なタスク定義を渡して委譲。
  - 独立タスクの並列ディスパッチに対応。

### 5. `dip:verify` (QA・テスト自動修復)
- **目的**: 「動くはず」という主観を排し、独立した QA サブエージェントがビルド、リント、テストを実行して客観的動作証拠を収集します。
- **特徴**:
  - テスト失敗時の自動修復ループ（最大5サイクル）。
  - **同一エラー安全停止ガード**: 同一エラーが3回連続した場合、根本的障害として即時中断・報告。

### 6. `dip:review` (3視点並列レビュー)
- **目的**: 納品前に、3つの独立したサブエージェントによる多角的な品質検証を並列実施します。
  1. **Architect Reviewer**: 仕様・計画との機能的一致、責務境界の維持。
  2. **Security Reviewer**: OWASP、認証認可、入力バリデーション、機密情報漏洩。
  3. **Code Reviewer**: 可読性、保守性、エッジケース、AI-Slop（冗長な定型コード）の排除。
- **全会一致承認**: 全レビュアーの `APPROVE` が揃うまで完了と認めず、指摘事項は自動修正・再検証（最大3ラウンド）。
- **成果物**: `.dip/reviews/review-{slug}.md`

---

## 成果物のディレクトリ構成

生成されるすべての成果物・中間ログは、プロジェクト内の `.dip/` 配下に一括管理されます。

```text
.dip/
├── specs/     # deep-interview で確定した仕様書 (.md)
├── plans/     # deep-plan で合意された計画書 (.md)
├── reviews/   # review で記録された多角レビュー判定書 (.md)
└── state/     # セッション実行状態・中間進行状況 (.json)

agents/        # サブエージェント定義 (.md)
├── dip-analyst.md            # 要件定義・スコープ分析・受入基準具体化
├── dip-explore.md            # 高速コードベース探索・構造把握
├── dip-critic.md             # 批判的評価・前提検証・リスク及び失敗シナリオ検証
├── dip-architect.md          # システムアーキテクチャ・設計・トレードオフ分析
├── dip-planner.md            # 実装計画・タスク分解・RALPLAN-DR策定
├── dip-executor.md           # コード実装・最小差分・タスク遂行
├── dip-qa-tester.md          # QA・対話的動作テスト・実行時検証
├── dip-test-engineer.md      # テスト戦略設計・カバレッジ検証・テスト実装
├── dip-debugger.md           # 不具合根本原因分析・スタックトレース解析・最小修正
├── dip-verifier.md           # 受入基準検証・客観的エビデンス照合
├── dip-security-reviewer.md  # セキュリティ脆弱性検証・OWASP Top 10・機密情報保護
├── dip-code-reviewer.md      # コード品質・可読性・SOLID原則・AIスロップ排除
├── dip-code-simplifier.md    # コード単純化・不要複雑性排除・保守性向上
├── dip-document-specialist.md# 外部仕様・公式ドキュメント・ライブラリ参照調査
├── dip-writer.md             # 技術文書・README・仕様書作成
├── dip-git-master.md         # Git操作・ブランチ管理・アトミックコミット・PR作成
├── dip-designer.md           # UI/UXデザイン・デザインシステム設計
├── dip-scientist.md          # データサイエンス・ML実験・データ分析
└── dip-tracer.md             # 実行トレース・コールグラフ解析・データフロー追跡
```

---

## 移植サブエージェント一覧と担当スキル

本リポジトリでは、各ライフサイクル工程に特化した19種類のサブエージェント定義（原文英語仕様）を標準配備しています（`agents/` および `.agents/agents/`）。

| エージェント名 | 種別 | 役割・責務 | 主な担当スキル |
| :--- | :--- | :--- | :--- |
| **`dip-analyst`** | 読取専用 | 要件のギャップ抽出、暗黙の前提の炙り出し、受入基準の具体化 | `dip:deep-interview`, `dip:autopilot` |
| **`dip-explore`** | 読取専用 | 高速コードベース探索、Brownfield事前調査、環境事実の自律確認 | `dip:deep-interview`, `dip:deep-plan`, `dip:autopilot` |
| **`dip-critic`** | 読取専用 | 批判的評価、Skeptic / Contrarian 視点、事前検死（Pre-Mortem）、前提脆弱性検証 | `dip:deep-interview`, `dip:deep-plan`, `dip:review` |
| **`dip-architect`** | 読取専用 | アーキテクチャ健全性評価、対立仮説（Steelman）構築、トレードオフ分析 | `dip:deep-plan`, `dip:review`, `dip:autopilot` |
| **`dip-planner`** | 読取専用 | 実行計画策定、タスク依存グラフ分解、RALPLAN-DRサマリー作成 | `dip:deep-plan`, `dip:autopilot` |
| **`dip-executor`** | 実装実行 | 最小差分（Smallest Viable Diff）によるアトミックなコード実装 | `dip:execute`, `dip:autopilot` |
| **`dip-qa-tester`** | 動作検証 | 対話的CLI/サービス起動テスト、コマンド出力キャプチャ、動作エビデンス収集 | `dip:verify`, `dip:autopilot` |
| **`dip-test-engineer`** | テスト作成 | テストピラミッド設計、カバレッジギャップ補完、単体/結合/E2Eテスト実装 | `dip:verify` |
| **`dip-debugger`** | 原因分析 | エラー根本原因分析、スタックトレース解析、最小差分での障害解消 | `dip:verify`, `dip:execute` |
| **`dip-verifier`** | 判定照合 | 主観を排した受入基準の客観的検証、完了判定（PASS/FAIL）の監査 | `dip:verify`, `dip:review` |
| **`dip-security-reviewer`** | 監査 | OWASP Top 10、認証認可、入力バリデーション、機密情報漏洩スキャン | `dip:review`, `dip:autopilot` |
| **`dip-code-reviewer`** | 監査 | 仕様適合性検証、ロジック正確性、SOLID原則、AI-Slop（冗長コード）排除 | `dip:review`, `dip:autopilot` |
| **`dip-code-simplifier`** | 最適化 | 振る舞いを維持したネスト平坦化、過剰抽象化排除、保守性向上 | `dip:execute`, `dip:review` |
| **`dip-document-specialist`** | 調査 | 外部公式ドキュメント、最新API仕様、バージョン互換性調査 | `dip:deep-interview`, `dip:deep-plan` |
| **`dip-writer`** | 文書化 | 仕様書、README、APIドキュメント、ユーザーガイド作成 | `dip:deep-interview`, `dip:deep-plan` |
| **`dip-git-master`** | VCS操作 | 論理的アトミックコミット分割、ブランチ管理、PR概要作成 | `dip:execute`, `dip:autopilot` |
| **`dip-designer`** | UI/UX | UI設計・デザインシステム・アクセシビリティ検証 | `dip:deep-interview`, `dip:deep-plan` |
| **`dip-scientist`** | ML/分析 | データ分析・MLモデル設計・実験追跡 | `dip:verify`, `dip:deep-plan` |
| **`dip-tracer`** | トレース | 実行トレース・コールグラフ解析・複雑データフロー追跡 | `dip:verify`, `dip:review` |

---

## インストール・セットアップ

本リポジトリの機能（スキル群およびサブエージェント群）を完全に利用するには、**1. スキル群のインストール** と **2. サブエージェント群のインストール** の2ステップを実行します。

### ステップ 1: スキル群のインストール (`npx skills`)

AIエージェント用パッケージマネージャー `npx skills`（Vercel Labs提供）を用いてスキルをインストールします。

#### プロジェクトへのインストール（推奨）

対象プロジェクトのルートディレクトリで実行します。

```bash
# スキルを一括インストール（対話式選択または全スキル）
npx skills add yujiod/dip-skills

# 常に最新バージョンのCLIを使用してインストール
npx skills@latest add yujiod/dip-skills
```

特定のエージェント（Claude Code、Cursor、GitHub Copilot等）に限定してインストールする場合：

```bash
npx skills add yujiod/dip-skills --agent claude-code cursor
```

特定のスキルのみを個別にインストールする場合：

```bash
npx skills add yujiod/dip-skills --skill dip-autopilot
```

#### グローバル利用（ユーザーレベル）

マシン上の全プロジェクトから利用したい場合は `-g` フラグを付与します。

```bash
npx skills add yujiod/dip-skills -g
```

---

### ステップ 2: サブエージェント群のインストール (`npx degit`)

各スキルが実行時に起動する全19種の専門サブエージェント定義（`dip-*`）を配備します。`npx degit` を用いることで、Git clone を行うことなく必要なエージェント定義ディレクトリのみをワンライナーで配備できます。

#### プロジェクトへの配備（推奨）

対象プロジェクトのルートディレクトリで実行し、プロジェクトの Git 管理に含めます。

```bash
# プロジェクト直下の .agents/agents に展開
npx degit yujiod/dip-skills/agents .agents/agents

# Git でコミットしてチーム共有
git add .agents/agents
```

#### グローバル配備（マシン共通利用）

マシン上のすべてのプロジェクトから共通してエージェントを利用可能にします。

```bash
# ユーザーのホームディレクトリ配下の ~/.agents/agents に展開
npx degit yujiod/dip-skills/agents ~/.agents/agents
```

---

### チーム開発・CI/CD運用（ロックファイル復元）

`npx skills add` 実行時に `.skills.json` および `skills-lock.json` が生成されます。これらを Git 管理に含めることで、チーム開発や CI 環境で同一バージョンのスキルを確実に再現できます。

```bash
# ロックファイルから復元（npm ci 相当）
npx skills experimental_install

# インストール済みスキルの一覧表示
npx skills list

# 全スキルを最新版に更新
npx skills update
```

---

### 手動登録（ローカル開発 / Antigravity 直接設定）

本リポジトリをクローンして直接開発する場合や、Antigravity の設定ファイルから参照する場合は以下のように指定します。

- **プロジェクト設定 (`skills.json` または `.agents/skills.json`)**:
  ```json
  {
    "entries": [
      {
        "path": "skills"
      },
      {
        "path": "~/.agents/skills"
      }
    ]
  }
  ```

- **グローバル設定 (`~/.gemini/config/skills.json`)**:
  ```json
  {
    "entries": [
      {
        "path": "~/.agents/skills"
      },
      {
        "path": "/path/to/dip-skills/skills"
      }
    ]
  }
  ```

- **エージェントの手動シンボリックリンク配備**:
  ```bash
  # グローバル配備（クローン元の更新を自動反映）
  mkdir -p ~/.agents/agents
  ln -sf /path/to/dip-skills/agents/*.md ~/.agents/agents/
  ```

---

## 利用方法

### 1. フルオート自律実行（推奨）
```text
dip:autopilot "TypeScript と Fastify を用いたセッション認証付き REST API"
```
- オプション:
  - `--quick`: 高速モード（曖昧度閾値 35%）
  - `--deep`: 深層モード（曖昧度閾値 10%）
  - `--skip-qa`: QA検証スキップ
  - `--skip-review`: 最終多角レビュースキップ

### 2. 各工程ごとの個別実行
```text
# 1. 要件定義インタビュー
dip:deep-interview "認証機能の追加"

# 2. コンセンサス計画策定
dip:deep-plan .dip/specs/deep-interview-auth.md

# 3. 実装実行
dip:execute .dip/plans/plan-auth.md

# 4. QAテスト・ビルド検証
dip:verify

# 5. 3視点品質レビュー
dip:review
```

---

## ライセンス

[MIT License](LICENSE)
