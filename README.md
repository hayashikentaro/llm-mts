# LLM-MTS

LLMエージェントによる開発を、単なる multi-agent collaboration ではなく、**Multiteam System (MTS)** として設計するための研究・設計メモ。

## 中心仮説

従来の LLM multi-agent software development は、しばしば「ソフトウェア会社」や「開発チーム」を模倣する。

このリポジトリでは、そこから一歩進めて、LLMエージェント群を次のように扱う。

> 異なる近接目標を持つエージェント群が、共有された遠位目標のために、成果物・レビュー・ゲート・統合判断を通じて調整されるシステム。

つまり、焦点は「仲良く協調するエージェント」ではなく、**調整された対立を品質生成装置として使うエージェント組織**にある。

## 用語

| 用語 | 意味 |
| --- | --- |
| エージェント | 目的・責務・権限・道具を持つAI作業主体 |
| 近接目標 | そのエージェントが局所的に最適化する目標 |
| 遠位目標 | エージェント群全体が共有する上位目標 |
| MTS unit | Builder / Reviewer / Integrator などからなる作業単位 |
| handoff artifact | エージェント間で受け渡す成果物、レビュー、判断材料 |
| gate | マージ・再作業・保留などを決める通過条件 |

## なぜMTSか

MTS研究では、複数のチームがそれぞれ異なる近接目標を持ちながら、少なくとも一つの遠位目標を共有する構造が扱われる。

これは、LLMエージェントによる開発にも自然に対応する。

- Builder は実装を前に進める
- Reviewer は壊れやすさ・仕様逸脱・テスト不足を疑う
- UX Critic は実装都合を無視して体験品質を見る
- Security Critic は便利さより権限境界を見る
- Integrator は衝突を整理し、人間が判断しやすい形に圧縮する
- Human Owner は最終判断を行う

重要なのは、全員にバランスを取らせないこと。バランスを取るのは Integrator / Human Owner の責務であり、各専門エージェントは意図的に偏ってよい。

## 既存研究・実務との位置づけ

このアプローチは、以下の中間にある。

1. 人間組織研究としての Multiteam Systems
2. 大規模アジャイル開発における team-of-teams / coordination 研究
3. CODEOWNERS、QA、SRE、Security Review などの開発統治実務
4. MetaGPT / ChatDev / AutoGen などの LLM multi-agent 開発

既存のLLM multi-agent開発は、主に「役割分担されたAIたちが協調して成果物を作る」方向に寄る。

このリポジトリでは、より明示的に次を設計対象にする。

- 相反する近接目標
- 役割境界の維持
- 成果物ベースの受け渡し
- レビュー分類
- 統合判断
- 人間の意思決定キュー

## 初期ドキュメント

- [Research Map](docs/research-map.md)
- [Agent Organization Model](docs/agent-organization-model.md)
- [Handoff and Gate Protocol](docs/handoff-and-gate-protocol.md)

## 現時点の結論

LLMエージェント開発をMTSとして見ると、設計の焦点は「会話で協調させること」から「責務対立を成果物とゲートで処理すること」へ移る。

これは TaskDeck のような複数AI作業監督UIにおいて、単なるターミナル監視ではなく、**エージェント組織の状態と人間の判断キューを扱うUI**へ進むための理論的な足場になりうる。
