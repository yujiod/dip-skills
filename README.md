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
```

---

## インストール・セットアップ

AIエージェント用パッケージマネージャー `npx skills`（Vercel Labs提供）を用いてインストールします。

### 1. プロジェクトへのインストール（推奨）

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

### 2. グローバル利用（ユーザーレベル）

マシン上の全プロジェクトから利用したい場合は `-g` フラグを付与します。

```bash
npx skills add yujiod/dip-skills -g
```

### 3. チーム開発・CI/CD運用（ロックファイル復元）

`npx skills add` 実行時に `.skills.json` および `skills-lock.json` が生成されます。これらを Git 管理に含めることで、チーム開発や CI 環境で同一バージョンのスキルを確実に再現できます。

```bash
# ロックファイルから復元（npm ci 相当）
npx skills experimental_install

# インストール済みスキルの一覧表示
npx skills list

# 全スキルを最新版に更新
npx skills update
```

### 4. 手動登録（ローカル開発 / Antigravity 直接設定）

本リポジトリをクローンして直接開発する場合や、Antigravity の設定ファイルから参照する場合は以下のように指定します。

- **プロジェクト設定 (`skills.json` または `.agents/skills.json`)**:
  ```json
  {
    "entries": [
      {
        "path": "skills"
      }
    ]
  }
  ```

- **グローバル設定 (`~/.gemini/config/skills.json`)**:
  ```json
  {
    "entries": [
      {
        "path": "/path/to/dip-skills/skills"
      }
    ]
  }
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
