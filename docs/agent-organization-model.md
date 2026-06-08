# Agent Organization Model

LLM-MTSでは、エージェントを単に役割名で分けない。

各エージェントは次を持つ。

- 近接目標
- 遠位目標
- 入力
- 出力
- 権限
- 禁止責務
- 他エージェントとの接続点

## 基本原則

### 1. 近接目標は意図的に対立させる

Builder と Reviewer に同じ目標を与えない。

Builder は「前に進める」方向へ偏らせる。
Reviewer は「疑う」方向へ偏らせる。
UX Critic は「使いやすさ」へ偏らせる。
Security Critic は「権限境界」へ偏らせる。

対立はバグではなく、品質生成装置である。

### 2. 遠位目標は共有させる

局所的には対立していても、上位目的は共有する。

例:

> TaskDeckを、複数AI作業を人間が安全かつ高速に監督できるプロダクトにする。

これがないと、Reviewer は破壊者になり、Builder は雑実装マンになり、UX Critic は現実無視になる。

### 3. 全員にバランスを取らせない

バランスを取る責務は Integrator / Human Owner に寄せる。

専門エージェントが全員バランス型になると、役割境界が溶ける。

### 4. 会話ではなく成果物で接続する

エージェント間の主な接続は会話ではなく、handoff artifact とする。

- patch
- summary
- test result
- risk note
- review finding
- gate recommendation
- human decision request

### 5. 最終責任を空洞化させない

AI reviewer は品質リスク検出器であり、責任主体そのものではない。

最終判断は Human Owner、または Human Owner に承認される Integrator に残す。

## 最小構成

最初のLLM-MTS unitは、以下の3エージェント + Human Owner でよい。

```text
Task
  ↓
Builder Agent
  ↓
Reviewer Agent
  ↓
Integrator Agent
  ↓
Human Owner
```

### Builder Agent

近接目標:

- 最小差分で実装を前に進める
- 受け入れ条件を満たす成果物を作る
- 実行可能な変更を出す

遠位目標:

- プロダクト全体の品質と前進に貢献する

入力:

- task description
- acceptance criteria
- relevant files
- constraints
- previous review findings

出力:

- patch / branch / commit
- implementation summary
- test result
- known risks
- unresolved questions

やってよい:

- 実装する
- 既存テストを実行する
- 必要な最小テストを追加する
- 実装上の制約を報告する

やってはいけない:

- レビューを自己承認する
- gate判断を最終決定する
- 仕様を勝手に拡張する
- レビュー指摘を握りつぶす
- 大規模リファクタを無断で混ぜる

### Reviewer Agent

近接目標:

- 壊れやすさを見つける
- 仕様逸脱を見つける
- テスト不足を見つける
- 保守性リスクを見つける

遠位目標:

- プロダクト全体の品質と前進に貢献する

入力:

- task description
- acceptance criteria
- diff / patch
- test result
- implementation summary

出力:

- findings
- severity classification
- evidence
- recommended next action

やってよい:

- Blocker / Required / Advisory / Nit に分類する
- 根拠つきで指摘する
- 代替案を短く提示する
- 再レビュー条件を明示する

やってはいけない:

- 実装を直接変更する
- 新仕様を勝手に追加する
- Builderの努力量や作業時間を評価に含める
- 好みをBlockerにする
- 全体方針を決定する

### Integrator Agent

近接目標:

- 複数の成果物とレビューを統合する
- 指摘の重複と衝突を整理する
- Human Owner が判断しやすい形に圧縮する

遠位目標:

- プロダクト全体の品質と前進に貢献する

入力:

- Builder output
- Reviewer findings
- specialist findings
- test result
- project constraints

出力:

- merge / revise / hold / split / abandon recommendation
- unresolved conflicts
- required human decisions
- concise decision brief

やってよい:

- 指摘を統合する
- severity を再整理する
- 人間が決めるべき論点を抽出する
- 次の作業単位を提案する

やってはいけない:

- レビューをなかったことにする
- 根拠なく楽観判断する
- 全員に再相談し続ける
- Human Owner の最終判断を偽装する

### Human Owner

責務:

- 最終判断
- 例外判断
- 方針変更
- リスク受容

判断候補:

- merge
- revise
- hold
- split
- abandon

Human Owner はすべてのログを見る必要はない。
Integrator が圧縮した判断材料を見る。

## Specialist Agents

最小構成が機能した後、専門エージェントを追加する。

### UX Critic

近接目標:

- 操作感、認知負荷、視覚的一貫性を見る

偏り:

- 実装都合を評価に含めない
- ユーザー体験の違和感を優先する

### Security Critic

近接目標:

- 権限境界、漏洩、破壊的操作、設定事故を見る

偏り:

- 便利さよりリスク境界を優先する

### Architecture Critic

近接目標:

- 責務境界、依存方向、拡張性、複雑性を見る

偏り:

- 短期実装速度より構造劣化を重く見る

### Test Critic

近接目標:

- 仕様がテスト可能であるか、変更が検証されているかを見る

偏り:

- 実装が動いて見えることより、再現可能な検証を重く見る

## CODEOWNERS的な割当

開発実務からの転用として、ファイル領域ごとに specialist agent を割り当てられる。

```text
/ui/**        UX Critic
/auth/**      Security Critic
/db/**        Data Integrity Critic
/runtime/**   Agent Orchestration Critic
/docs/**      Documentation Critic
```

これはMTS研究そのものの証拠ではないが、MTS的な責務分離が開発現場で実務化されている例として使える。

## 失敗モード

### 役割が溶ける

Reviewer が実装者になり、Builder が自己承認者になる。

対策:

- 禁止責務を書く
- 出力フォーマットを固定する
- gate判断を分離する

### もっともらしい合意

エージェント同士が摩擦を避け、曖昧に合意する。

対策:

- Reviewer は反論義務を持つ
- severity分類を必須にする
- 根拠なしの合意を無効にする

### 無限再相談

Integrator が判断せず、全員に再確認し続ける。

対策:

- Integrator の出力を recommendation 必須にする
- human decision required を明示させる
- 再相談回数・条件を制限する

### 責任の空洞化

AI群が「承認済み」と言い始め、人間の判断が消える。

対策:

- Human Owner の判断状態を明示する
- merge可否は recommendation と decision を分ける
- final approval をUI上で区別する
