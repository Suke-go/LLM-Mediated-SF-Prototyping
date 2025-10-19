# MASTERDOCS — LLM-Mediated SF Prototyping

最終更新: 2025-10-19

## この文書の目的
- 本プロジェクトの「目的レイヤー」を明確化し、実装方針を合意するためのマスタードキュメント。
- ビジョン起点の研究を支援するために、SFプロトタイピングをLLMで媒介する仕組みの目的・スコープ・成果基準を定義する。
- 実装詳細（アルゴリズムやAPI仕様）は別章で拡張予定。本章では目的・仮説・検証計画にフォーカスする。

## 背景と問題意識
- HCIコミュニティではVision based Researchの重要性が繰り返し強調されている。
- しかしビジョンへの「共感ギャップ」が大きい。
  - 具体的すぎて「よくわからない」／抽象的すぎて「よくわからない」という両極端がWISS未来ビジョン等で発生。
  - 特にDigital NatureやTangibleなど、概念の想像が難しい層が一定数存在。
- アイディア自体は収集できても「世界観に没入できるストーリー化」に課題が残り、結果としてアイディアの面白さ／伝わりやすさが伸び悩む。

## 目的（目的レイヤー）
- 研究者・実践者が「共感可能な未来像」を共有できるよう支援する。
- WISS等の文脈で、未来ビジョンと具体的プロトタイピングとの行き来を円滑にする。
- SFプロトタイピングを核に、アイディアを「没入可能な世界観」へ翻訳・増幅し、共創を促進する。
- メディア考古学的視点（過去の文脈・技術史）を取り込み、発想の幅を拡張する。

## 成果のイメージ（アウトカム）
- 参加者・読者がビジョンに対して「わかる／面白い／やってみたい」を感じる頻度の増加。
- WISSのアイディア同士が結び付けられ、新たな共創が生まれるネットワーク効果。
- ワークショップ（WS）やプロトタイプの質的向上（評価指標・フィードバックで確認）。
- Digital NatureやTangibleなどの難解な概念に対する理解促進と再解釈。

## 成功指標（例）
- 共感度・理解度: 参加者アンケートの平均スコア向上（例: 20%以上）。
- 物語化の質: 生成されたSFシナリオに対する専門/非専門評価の向上。
- ネットワーク形成: WISS内でのアイディア間リンク数・新規協業件数の増加。
- 再利用性: ワークショップ・テンプレートの外部適用回数。

## 対象とステークホルダー
- 研究者・学生（WISS参加者、HCIコミュニティ）
- WSファシリテーター・企画者
- レビュアー・キュレーター（予稿集やセッション設計に関与）
- 一般の関心層（デザイン、アート、エンジニアリング）

## スコープ
- LLMを媒介にしたSFプロトタイピング支援の方法・ツール・テンプレート。
- 過去のWISS予稿集等のコーパス化（OCR）と、概念/テーマの整理（メタデータ・タグ付け）。
- アイディア間の関係性を可視化・探索する「ネットワーキング」機能。
- メディア考古学的参照（過去の技術・作品・議論）を導入する知識補助。

## 非スコープ（当面）
- 商用運用や長期保守を前提とした大規模システム。
- 研究コミュニティ外の包括的検索/推薦プラットフォーム。
- 新規HCI理論の提唱そのもの（ただし既存理論のブリッジは重視）。

## ソリューション仮説（高レベル機能ブロック）
1) ビジョン・ギャップ診断
   - 「具体的すぎる/抽象的すぎる」を検知し、言い換え・具体化/抽象化の方向性を提示。
2) 世界観生成アシスタント
   - 没入可能な設定・登場要素・制約条件を提案し、ビジョンと一貫したSFシナリオ雛形を提示。
3) メディア考古学ブリッジ
   - 過去の事例・概念・技術史を参照し、再解釈や逆照射を促す。
4) アイディア・ネットワーキング（WISS内共創）
   - 近縁/補完関係の発見、セッション設計やWS設計への活用。
5) コーパス構築（OCR + メタ化）
   - 過去WISS予稿のOCR抽出、トピック/キーワード/関係性の付与。
6) WSキット（テンプレート/プロンプト）
   - ファシリテーション手順、プロンプトセット、評価ルーブリックの雛形。
7) 評価ループ
   - 人手評価と自動指標を併用し、反復的に改善。

※ 本章では機能の名前と役割のみを示し、アルゴリズム詳細は別途「実装レイヤー」で定義する。

## 方法（エージェント分解とクラス設計）
本プロジェクトでは、手戻りを最小化しブラックボックス性を下げるため、LLMエージェントを役割ごとに分解し、入出力（クラス）を厳密に定義する。

### パイプライン概要
- Agent 1: Vision Extractor — 論文から「狙い／未来ビジョン」を抽出し、`VisionClass`を生成。
- Agent 2: Narrative Scaffolder — `VisionClass`を基にナラティブの構成案と、登場するシステム方向性を`ComponentsClass`として設計。
- Agent 3: Story Composer — `VisionClass`と`ComponentsClass`、任意の`AuthorClass`/`ArtistClass`を踏まえ、絵本SF向けの構成素（台本/指示）を生成（`ScenarioComponentsScriptsClass`と`IllustrationComponentsScriptClass`）。
- Agent 4: Scenario Realizer — それらの構成素を統合し、最終的なシナリオ`ScenarioClass`を出力。
- Optional Agent 0: Corpus/OCR Orchestrator — WISS予稿のOCR・整形・セクション抽出（「未来ビジョン」検出を補助）。
- Optional Agent 5: Critic/Evaluator — 整合性・独自性・可読性・倫理/安全を評価し、改善要求を発行。
- Optional Agent 6: Network Weaver — WISS内の他アイディアとのリンク提案（共創促進）。

### エージェントの役割（高レベル）
- Agent 1 — Vision Extractor
  - Input: ユーザー提供の論文テキスト/PDF（必要に応じてOCR後テキスト）
  - Focus: 「未来ビジョン」セクション、目的/狙い、社会実装の示唆、価値観。
  - Output: `VisionClass[]`、引用メタ情報（出典セクション/段落）、信頼度。
- Agent 2 — Narrative Scaffolder
  - Input: `VisionClass[]`
  - Task: 抽象狙いからPossible Applicationsを導出し、システム形態（例: マイクロチップ/AGI/ロボット/アバタ/VR/MR/その他）を決め、`ComponentsClass`を作成。
  - Output: `ComponentsClass`、ナラティブ骨子（Act/Beatレベルの素案）、未確定点。
- Agent 3 — Story Composer
  - Input: `VisionClass`+`ComponentsClass`+任意`AuthorClass`/`ArtistClass`
  - Task: 絵本SF小説の体裁に合わせ、台本とイラスト指示の2系統を構成。
  - Output: `ScenarioComponentsScriptsClass`、`IllustrationComponentsScriptClass`。
- Agent 4 — Scenario Realizer
  - Input: 上記すべて
  - Task: 一貫性・読者体験を最適化し、`ScenarioClass`（最終シナリオ）を生成。

### クラス定義（最小仕様）
以下は実装時のI/O契約として用いる最小キー。値の形式はJSON/YAML等で保持する。

```json
// VisionClass
{
  "id": "vision-001",
  "title": "",
  "summary": "論文から抽出した要旨（要約）",
  "intent": {"problem": "", "aspiration": ""},
  "horizon": "near|mid|far",
  "stakeholders": ["researchers", "users", "society"],
  "application_keywords": [""],
  "societal_values": ["accessibility", "sustainability", "privacy"],
  "constraints": [""],
  "citations": [{"section": "", "para": 0, "note": "引用の根拠（短文）"}],
  "confidence": 0.0
}

// ComponentsClass
{
  "id": "comp-001",
  "modality": "chip|AGI|robot|avatar|VR|MR|other",
  "capability_set": ["perception", "actuation", "generation"],
  "interaction_modes": ["voice", "gesture", "haptic", "immersive"],
  "environment": ["home", "public", "lab", "virtual"],
  "data_requirements": ["sensors", "corpus", "privacy"],
  "risks": ["misuse", "bias", "safety"],
  "affordances": [""],
  "open_questions": [""]
}

// AuthorClass（任意）
{
  "authors": ["SF作家名"],
  "influence_tags": ["hard-sf", "new-weird", "speculative"],
  "tone": "warm|clinical|lyrical|dark",
  "pacing": "slow|medium|fast",
  "theme_patterns": ["first-contact", "posthuman"]
}

// ArtistClass（任意）
{
  "artists": ["アーティスト名"],
  "style_tags": ["gouache", "ink", "flat", "painterly"],
  "color_palette": ["#..."],
  "composition_cues": ["rule-of-thirds", "centered"],
  "medium": "digital|watercolor|pencil|mixed",
  "texture": ["grain", "paper"],
  "reference_works": ["作品名/URL"]
}

// ScenarioComponentsScriptsClass（台本側の構成素）
{
  "act_structure": "three-act|four-act|journey",
  "pov": "first|third|omniscient",
  "narrative_voice": "",
  "beats": [
    {"id": "beat-1", "goal": "", "conflict": "", "turn": ""}
  ],
  "scenes": [
    {"id": "scene-1", "location": "", "characters": [""], "onstage_props": [""], "outcome": ""}
  ],
  "key_props": [""],
  "calls_to_action": [""]
}

// IllustrationComponentsScriptClass（絵本向けの画面指示）
{
  "page_scripts": [
    {
      "page": 1,
      "caption": "",
      "depiction": "主要被写体・関係・動き",
      "composition": "wide|close|bird-eye|worm-eye",
      "style_influence": {"authors": [""], "artists": [""], "tags": [""]},
      "camera": "focal-length/angle",
      "lighting": "", "color": "",
      "assets": ["prop/system/environment"],
      "safety_notes": ["肖像権/著作権/配慮事項"]
    }
  ]
}

// ScenarioClass（最終成果）
{
  "title": "",
  "logline": "1-2文の要約",
  "audience": "children|general|expert",
  "length_pages": 24,
  "page_map": [1,2,3],
  "prose": "本文（ページ割りの見出し付きでも可）",
  "illustration_briefs": "イラスト発注用まとめ",
  "asset_manifest": ["必要小物・舞台・キャラ"]
}
```

### I/O契約と運用
- 形式: JSON（UTF-8, LF）。段階ごとにファイルを保存し、差分を追跡可能にする。
- 根拠の可視化: `citations[].section/para`と短い`note`のみを保持（思考過程の逐語は保存しない）。
- 介入ポイント: 各段の`open_questions`をUIで編集→次段に再投入。

### 透明性・安全性（ブラックボックス低減）
- 追跡可能性: 各エージェントの入出力を「バージョン+日時」でアーカイブ。決定理由は箇条書き一行メモに限定。
- 出典厳格化: Agent 1は引用の無い主張を「仮説」タグで明示。偽の引用生成を禁止。
- 整合性検査: Agent 5（Critic）が`VisionClass↔ComponentsClass↔Scripts↔Scenario`の矛盾をチェックし、修正リクエストを発行。
- 倫理・安全: 登場人物表象、社会影響、プライバシ等の観点でフラグを付与。

### 評価（高レベルの更新）
- 可読性/共感度: 被験者評価（5段階）でScenarioの改善率を計測。
- 一貫性: Criticの自動スコア（用語一致、設定整合）と人手採点の相関。
- 活用度: `IllustrationComponentsScriptClass`の外部制作への発注実用性（修正指示回数）。

### 実装時のヒント
- 「未来ビジョン」検出: セクション見出しと典型語彙（例: 将来/応用/期待/社会実装）に基づくヒューリスティック+軽量分類器。
- 1→2の橋渡し: `intent.aspiration`→`modality`の対応表を運用で更新（例: 「身体拡張」→「ロボット/ウェアラブル」）。
- 絵本体裁: `length_pages`を24/32等の定型に合わせて最初に固定。奇数ページ=右開きのレイアウト指針を明示。

## 代表ユースケース
- 研究者のビジョン整理:
  - 粗いビジョンを投入 → ギャップ診断 → 世界観補強 → 論旨の可視化（登場要素、相互作用、制約）。
- WS設計支援（WISS文脈）:
  - 過去予稿コーパスからテーマ抽出 → 近縁アイディア束ね → シナリオ雛形と進行テンプレ生成。
- キュレーション/レビュー支援:
  - アイディア間の関係グラフを探索し、組合せや対比を発見。

## リスク・前提条件
- データ: OCR品質のばらつき、著作権・利用許諾の整備、言語多様性（主に日本語）。
- バイアス: コーパス/モデル由来の偏り、歴史的文脈の歪み。
- 妥当性: ハルシネーション対策、出力の検証可能性、出典トレーサビリティ。
- 受容性: 生成支援に対するコミュニティの期待値・透明性。

## 検証計画（高レベル）
- 指標: 共感度/理解度スコア、物語化の質評価、ネットワーク生成の有用性評価、再利用性。
- 手法: 小規模WSのパイロット、専門家/非専門家の二重評価、事例スタディ。
- データ: WISS予稿のサンプルOCR、メタデータ付与、匿名化や引用ルールの策定。

## ロードマップ（案）
1. 許諾・資料収集方針の確立（WISS予稿の利用条件整理）
2. ミニコーパス試作（OCR→タグ付け→要約）
3. ギャップ診断のルーブリック試作（言い換え規則・観点）
4. 世界観テンプレの作成（登場要素/制約/相互作用の型）
5. ネットワーク可視化のPoC（アイディア間リンク）
6. 小規模WSでの評価（指標の妥当性確認）
7. 統合デモと反復改善（フィードバック反映）

## 用語（簡易）
- Vision based Research: 未来像をドライバとして研究を進めるアプローチ。
- SFプロトタイピング: 物語を用いて技術/社会の可能性を検討する手法。
- メディア考古学: 過去のメディア・技術史を参照し、現在/未来を再解釈する視点。
- Digital Nature/Tangible: 特定の概念系。理解の障壁を低減する翻訳が必要。

## オープン課題 / 決めたいこと
- 参照データの範囲・許諾プロセス（WISS予稿、関連資料）。
- 評価ルーブリック（共感度、物語質、ネットワーク有用性）の具体化。
- 概念タクソノミ（トピック、タグ体系、関係タイプ）。
- 最初に提供するWSテンプレの対象（学生向け/研究者向け/ミックス）。
- UI/インタラクション方針（CLI/ノートブック/簡易Webなど）。

## 次アクション（提案）
- 本MASTERDOCSのレビュー: 目的・スコープ・指標の合意。
- データ方針の確認: OCR対象・許諾確認の打ち合わせ設定。
- 最小PoCの選定: ギャップ診断 or 世界観テンプレのいずれかを先行実装。
- WSユースケースの要件洗い出し: 60–90分の最小構成で試す。

---
この文書は「目的レイヤー」に特化した初版です。合意後、「実装レイヤー」（アルゴリズム、入出力、評価手順）を別章として追補します。
