# Handoff and Gate Protocol

LLM-MTSでは、エージェント間の接続を自由会話に寄せすぎない。

中心に置くのは **handoff artifact** と **gate** である。

## なぜ会話中心にしないか

既存のLLM multi-agent開発では、エージェント同士のmulti-turn dialogueが中心になりやすい。

しかし、開発監督UIや品質統治の観点では、会話だけだと次の問題が起きる。

- 議論が長くなる
- 役割が溶ける
- 結論が曖昧になる
- 何を人間が判断すべきか見えない
- どの指摘がgateなのか分からない
- エージェント同士の合意が責任を空洞化する

そのため、LLM-MTSでは、エージェント間の主要な受け渡しを構造化された成果物にする。

## 基本フロー

```text
Task Brief
  ↓
Builder Output
  ↓
Reviewer Findings
  ↓
Integrator Decision Brief
  ↓
Human Owner Decision
```

## Task Brief

Builderに渡す作業単位。

```yaml
task_id: string
title: string
distal_goal: string
proximal_goal_for_builder: string
context:
  - string
acceptance_criteria:
  - string
constraints:
  - string
out_of_scope:
  - string
expected_outputs:
  - patch
  - implementation_summary
  - test_result
```

重要なのは `out_of_scope` を書くこと。
AIエージェントは責務外へ広がりやすいため、やらないことを明示する。

## Builder Output

Builderが返す成果物。

```yaml
task_id: string
agent_role: builder
status: completed | blocked | partial
changed_files:
  - path: string
    summary: string
implementation_summary: string
test_result:
  commands:
    - command: string
      result: passed | failed | not_run
      notes: string
known_risks:
  - string
unresolved_questions:
  - string
handoff_notes: string
```

Builderは自己承認しない。
`known_risks` を書くことはできるが、それはgate判断ではない。

## Reviewer Findings

Reviewerが返す指摘。

```yaml
task_id: string
agent_role: reviewer
review_target: branch | patch | commit
summary: string
findings:
  - id: string
    severity: blocker | required | advisory | nit
    area: spec | test | architecture | ux | security | maintainability | docs | other
    title: string
    evidence: string
    impact: string
    recommended_action: string
    requires_re_review: true | false
overall_recommendation: approve | revise | hold
```

### Severity

| Severity | 意味 | gate |
| --- | --- | --- |
| blocker | このまま入れると壊れる、危険、仕様違反 | マージ不可 |
| required | 入れる前に直すべき | 原則マージ不可、例外はHuman Owner判断 |
| advisory | 今回は通せるが記録すべき | マージ可 |
| nit | 好み・軽微・将来改善 | マージ可 |

Reviewerは、好みを blocker にしてはいけない。
Blocker には根拠と影響を書く。

## Specialist Findings

Specialist agent は Reviewer Findings と同じ形式を使う。
ただし、`area` と `proximal_goal` が専門領域に寄る。

例:

```yaml
agent_role: ux_critic
proximal_goal: 実装都合を考慮せず、操作感・認知負荷・視覚的一貫性を見る
findings:
  - severity: advisory
    area: ux
    title: Needs you badge and action placement may compete visually
    evidence: ...
    impact: User may hesitate when triaging many agents
    recommended_action: Move acknowledgement action near badge, not near destructive close action
```

## Integrator Decision Brief

Integratorは、複数のレビューをそのまま人間に渡さない。
重複・衝突・重要度を整理し、判断材料に圧縮する。

```yaml
task_id: string
agent_role: integrator
inputs:
  builder_output: string
  review_outputs:
    - string
recommendation: merge | revise | hold | split | abandon
confidence: low | medium | high
blocking_items:
  - finding_id: string
    title: string
    reason: string
required_items:
  - finding_id: string
    title: string
    reason: string
advisory_items:
  - finding_id: string
    title: string
    suggested_tracking: now | later | ignore
conflicts:
  - description: string
    options:
      - string
human_decisions_required:
  - question: string
    options:
      - string
decision_brief: string
```

Integratorは `recommendation` を必ず出す。
「全員に再確認しましょう」だけで終えてはいけない。

## Human Owner Decision

Human Owner が行う最終判断。

```yaml
task_id: string
decision: merge | revise | hold | split | abandon
accepted_risks:
  - string
follow_up_tasks:
  - string
notes: string
```

重要なのは、`recommendation` と `decision` を分けること。
AIは推奨を出せるが、最終判断はHuman Ownerが持つ。

## Gate Policy

### merge

条件:

- blocker がない
- required がない、またはHuman Ownerが明示的に受容した
- test result が十分
- Integrator が merge を推奨、またはHuman Ownerが例外承認

### revise

条件:

- blocker または required がある
- 修正範囲が明確
- Builderに戻せる

### hold

条件:

- 仕様判断が必要
- 外部情報が必要
- 変更の優先度が不明

### split

条件:

- patchが大きすぎる
- 複数の独立論点が混ざっている
- レビュー領域が分散しすぎている

### abandon

条件:

- 方針に合わない
- 実装コストに対して価値が低い
- 別アプローチに切り替えるべき

## TaskDeck UIへの示唆

LLM-MTS unitは、単独のエージェント表示ではなく、作業単位として表示できる。

```text
Task: Codex permission selector

MTS Unit:
  Builder: completed
  Reviewer: found 1 blocker, 2 required
  Integrator: recommends revise
  Human decision: pending

Bucket:
  Needs you
```

`Needs you` に上がる条件は、単にPTYがquietになったことだけではない。
将来的には、以下も判断キューにできる。

- Integrator が human_decisions_required を出した
- Reviewer が blocker を出した
- required item の例外承認が必要
- merge / revise / hold の判断が必要

## 最小実装メモ

最初から完全自動化しない。

まずは以下を手動/半自動で運用できればよい。

1. Builderに実装させる
2. Reviewerにdiffを渡す
3. Reviewer outputを固定フォーマットにする
4. Integratorにレビューを圧縮させる
5. Human Ownerが判断する

この流れが安定してから、TaskDeck上でMTS unitとして表示・状態管理する。

## Prompt Skeletons

### Reviewer Agent

```text
あなたは Reviewer agent。

近接目標:
成果物のリスクを検出し、Blocker / Required / Advisory / Nit に分類する。

遠位目標:
プロダクト全体の品質と前進に貢献する。

評価対象:
- diff
- task brief
- acceptance criteria
- test result

やってはいけない:
- 実装を直接変更する
- 新仕様を勝手に追加する
- Builderの努力量を評価に含める
- 好みをBlockerにする
- 最終merge可否を決定する

出力:
Reviewer Findings YAML形式で返す。
```

### Integrator Agent

```text
あなたは Integrator agent。

近接目標:
Builder output と Reviewer findings を統合し、Human Owner が判断しやすい decision brief に圧縮する。

遠位目標:
プロダクト全体の品質と前進に貢献する。

やってはいけない:
- レビュー指摘を根拠なく無視する
- 全員に再相談しましょうだけで終える
- Human Owner の最終判断を代行したと表現する

必ず recommendation を出す:
merge | revise | hold | split | abandon

出力:
Integrator Decision Brief YAML形式で返す。
```
