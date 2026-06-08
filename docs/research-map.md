# Research Map

このメモは、LLMエージェントによる開発を Multiteam System (MTS) として設計するための根拠地図である。

## 1. Multiteam Systems

MTS は、人間組織・産業組織心理学・チーム研究の領域で発展してきた考え方である。

ここで重要なのは、単なる大規模チームではなく、次の構造を持つこと。

- 複数のチームが存在する
- 各チームはそれぞれ異なる近接目標を持つ
- それらは少なくとも一つの遠位目標を共有する
- チーム間に相互依存がある
- 全体成果は、個別チームの能力だけでなく、チーム間調整に強く依存する

LLMエージェント開発に転用する場合、人間組織に固有の感情・評判・政治・心理的安全性をそのまま移植するのではなく、次のように変換する。

| 人間MTSの要素 | LLMエージェントMTSでの対応物 |
| --- | --- |
| 信頼 | ログ、再現性、テスト結果、根拠 |
| 責任感 | Human Owner / Integrator の明示的な判断責任 |
| 心理的安全性 | 不都合な指摘を出せるrole specとgate protocol |
| 組織文化 | AGENTS.md、review policy、handoff format |
| リーダーシップ | Integrator / Owner による調整と意思決定 |

AIエージェントには人間独自の面倒さがないため、MTSの構造部分をより機械的に適用しやすい可能性がある。

## 2. 大規模アジャイル開発との接続

ソフトウェア開発では、大規模アジャイルや team-of-teams の文脈で、MTS的な問題がすでに現れている。

代表的な論点は以下。

- 複数開発チーム間の依存関係
- feature team / platform team / enabling team などの職能差
- Scrum of Scrums などの調整メカニズム
- 複数チーム間での知識共有と意思決定
- タスク不確実性と相互依存の増大

ここでの根拠は、「ソフトウェア開発も複数チームの調整問題として扱われてきた」という点にある。

ただし、これは基本的に人間チーム対象の研究であり、AIエージェント設計にそのまま適用できるわけではない。

## 3. 開発統治実務との接続

MTSという名前を使わなくても、ソフトウェア開発実務にはMTS的な構造が多く存在する。

- CODEOWNERS
- code ownership
- QA / test team
- SRE
- Security review
- Architecture review
- Release engineering
- Platform team
- Enabling team

これらは、明示的にMTS理論から導かれたものとは限らない。

しかし、次の点でLLM-MTS設計の実務的な橋になる。

- 領域ごとに責任者を割り当てる
- 変更に対して専門レビュアーを要求する
- 実装速度と品質・安定性・セキュリティの近接目標が対立する
- 最終判断をレビュー・ゲート・オーナー承認に分ける

したがって、CODEOWNERSやSREは「MTS研究のエビデンス」ではなく、「MTS的な責務分離が開発現場で実務化されている証拠」として扱うのが適切。

## 4. LLM multi-agent software development

既存のLLM multi-agent開発には、以下のような流れがある。

### MetaGPT

- 人間のソフトウェア会社のワークフローを模倣する
- Product Manager / Architect / Project Manager / Engineer などの役割を使う
- SOPをプロンプト列としてエンコードする
- 中間成果物を明示し、協調的にソフトウェアを作る

### ChatDev

- 仮想ソフトウェア会社として、CEO / CTO / Programmer / Tester などのエージェントを置く
- design / coding / testing などのphaseに分ける
- multi-turn dialogue によって成果物を作る

### AutoGen

- 複数のカスタマイズ可能なエージェントが会話しながらタスクを遂行する
- LLM、人間入力、ツールを組み合わせられる
- 開発者が相互作用パターンを定義できる

これらは重要な先行例だが、多くは「協調」に寄っている。

LLM-MTSで掘りたい差分は以下。

| 既存multi-agent開発 | LLM-MTS |
| --- | --- |
| ソフトウェア会社の模倣 | MTSとしてのエージェント組織設計 |
| 協調中心 | 調整された対立中心 |
| 会話中心 | handoff artifact中心 |
| 役割名中心 | 近接目標・禁止責務・権限境界中心 |
| 最終成果物生成 | 人間判断キューとgate設計 |

## 5. ちょうどよい湯加減

このアプローチは、完全に新規で根拠がないわけではない。

- MTS研究は人間組織で確立した土台がある
- 大規模アジャイル開発にもMTS的分析がある
- 開発実務にはCODEOWNERSやSREなどの責務分離がある
- LLM multi-agent開発もすでに存在する

一方で、これらを明示的に統合して、

> 相反する近接目標を持つLLMエージェント群を、MTSとして設計し、成果物・レビュー・ゲート・統合・人間判断キューに落とす

という形は、まだ十分に一般化されていない。

そのため、説得力と新規性のバランスがよい可能性がある。

## References / Starting Points

- DeChurch, L. A., & Marks, M. A. Leadership in Multiteam Systems.
- Scheerer et al. Large-scale agile software development as a multiteam system.
- Dingsøyr et al. Coordination in multi-team programmes in large-scale agile software development.
- MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework.
- ChatDev: Communicative Agents for Software Development.
- AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation.
- CODEOWNERS / code ownership literature.
