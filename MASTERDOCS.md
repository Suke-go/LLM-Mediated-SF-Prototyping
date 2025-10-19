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
- Optional Agent 3b: Illustration Selector & Prompt Strategist — イラスト対象ページの選定とプロンプト戦略の最適化。
- Orchestrator（実行管理）— 依存関係、品質ゲート、再試行、監査ログを統括し、正確な実行を保証。

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

// IllustrationComponentsScriptClass（絵本向けの画面指示; プロンプト多様性と選定方針を含む）
{
  "selection": {
    "budget": 12,                        // 例: 24ページ中、12ページをイラスト化
    "spread_ratio": 0.25,                // 見開き(2ページ)の割合
    "policy": "coverage-importance",     // 目的関数の方針
    "criteria_weights": {                // 重み例（調整可能）
      "importance": 0.4,
      "visual_salience": 0.3,
      "diversity": 0.2,
      "coverage": 0.1
    },
    "notes": ["核となる発明の初出、感情的転換点、舞台転換を優先"]
  },
  "prompt_method": "descriptive|structured|scene-graph|dsl|image-to-image|control-guided",
  "prompt_template": {
    "blocks": {
      "subject": "主要被写体",
      "action": "動き/関係",
      "context": "舞台/時刻/雰囲気",
      "style": "画風/質感/参考作品",
      "composition": "構図/距離/視点",
      "camera": "焦点距離/角度",
      "lighting": "光源/コントラスト",
      "color": "配色/パレット",
      "negative": "避けたい要素"
    }
  },
  "consistency": {                       // シリーズ内の統一性確保
    "character_anchors": [
      {"id": "char-1", "refs": [""], "descriptors": ["年齢/服装/髪型/シルエット"]}
    ],
    "prop_anchors": [
      {"id": "prop-1", "refs": [""], "descriptors": ["サイズ/材質/意匠"]}
    ],
    "palette": ["#112233", "#ddeeff"],
    "style_lock": ["flat", "clean-lines"]
  },
  "rendering_params": {
    "aspect_ratio": "3:2",
    "resolution": "1024x682",
    "seed": 123,
    "variations": 2
  },
  "pacing": {
    "page_tempo": "gradual|sudden|rhythmic|linger",
    "transition_policy": "hard-cut|dissolve|match-cut",
    "notes": ["ページ間のペーシングと遷移を統一語彙で指定"]
  },
  "control_inputs": [                    // 必要に応じて制御信号
    {"type": "pose|depth|edges|layout|mask", "source": "path-or-instruction"}
  ],
  "page_scripts": [
    {
      "page": 1,
      "illustrate": true,
      "type": "full|spread|spot",
      "priority": 0.9,                   // 選定スコア
      "selection_reason": "ビジョンの核/世界観の導入",
      "timing": "gradual|sudden|rhythmic|linger",
      "transition_from_prev": "hard-cut|dissolve|match-cut",
      "cause_effect": {"cause": "", "effect": ""},
      "caption": "",
      "depiction": "主要被写体・関係・動き",
      "composition": "wide|close|bird-eye|worm-eye",
      "style_influence": {"authors": [""], "artists": [""], "tags": [""]},
      "camera": "focal-length/angle",
      "lighting": "", "color": "",
      "assets": ["prop/system/environment"],
      "audio_cues": {"sfx": [""], "ambience": ""},
      "consistency_refs": ["char-1", "prop-1"],
      "control_refs": [0],               // control_inputsの参照
      "prompt": "freeform-or-templated",
      "negative": "避ける要素",
      "safety_notes": ["肖像権/著作権/配慮事項"]
    }
  ]
}

// ScenarioClass（最終成果）
{
  "title": "",
  "logline": "1-2文の要約",
  "audience": "children|general|expert",
  "timeline": {"year": 2042, "era": "past|present|near-future|far-future"},
  "length_chars": {"target": 1000, "tolerance": 0.15},
  "cast_limits": {"primary": 2, "supporting": 2, "max_total": 4},
  "length_pages": 24,
  "page_map": [1,2,3],
  "prose": "本文（ページ割りの見出し付きでも可）",
  "illustration_briefs": "イラスト発注用まとめ",
  "asset_manifest": ["必要小物・舞台・キャラ"]
}
```

### イラスト選定ポリシー（どの部分を描くか）
- 候補抽出: `ScenarioComponentsScriptsClass.beats/scenes/key_props`から、以下に該当するページ候補を列挙。
  - 発明/システムの初出、使用がわかる瞬間、舞台転換、感情の転換点（inciting, midpoint, climax, resolution）。
  - 世界観の手掛かりが多い場面（環境・小物・関係性が密）。
- スコアリング: `importance`（物語/概念の核）+ `visual_salience`（視覚的伝達力）+ `diversity`（舞台/構図の多様性）+ `coverage`（各Actの均衡）。
- 分配の目安（24ページ例）: 導入×2、きっかけ×1、展開×3、ミッドポイント×1、逆転/山場×3、解決×1、終章×1（見開きを要所に配置）。
- 種別の使い分け: `type=spread`は世界観提示・山場、`full`は主要シーン、`spot`は説明補助や静物・UI要素。
- 低視覚性の場面: 言語中心のページは象徴モチーフ/ダイアグラムで補完し、長文を避ける。
- アクセシビリティ: 代替テキストの方針、低視力向けコントラスト/文字サイズを設計に反映。

### プロンプト方式（複数アプローチの運用）
- 方式の型:
  - descriptive: 自然文で包括的に記述（迅速だが再現性はやや低い）。
  - structured: `prompt_template.blocks`に沿ったタグ構成（再現性と編集性が高い）。
  - scene-graph: 被写体/関係/空間をグラフで定義→文章化（複雑構図に有効）。
  - dsl: ショットリスト/ストーリーボードの簡易言語で記述（連作に強い）。
  - image-to-image: 参考画像/ラフ→変換（既存資産や作風合わせ）。
  - control-guided: pose/depth/edges/layout/mask等で構図や整合性を強制。
- 選択ガイド:
  - 一貫性を重視: structured or dsl + consistency anchors。
  - 複雑な群像/UI: scene-graph + control-guided。
  - 既存作風/資料あり: image-to-image + inpaint/mask。
  - 素早い試行: descriptiveで方向を探索→確定後structuredへ移行。
- 影響の統合: `AuthorClass/ArtistClass`の`style_tags/palette/reference_works`を`style_influence`へ反映。
- 安全/品質ノート: `negative`に不適切要素、アーティファクト、過度な写実/露出などの回避条件を明示。
- 成果物: 各ページに「プロンプト束（prompt, negative, control_refs, consistency_refs, rendering_params）」を保存し再現性を担保。

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

## 前処理（PDF抽出スタック）
日本語の学術PDFを安定抽出するための推奨パイプライン。Optional Agent 0（Corpus/OCR Orchestrator）が実行し、Agent 1へ`PaperIngestClass`を受け渡す。

### 使用コンポーネント
- 抽出: PyMuPDF or pypdfium2（テキスト/ブロック/座標/画像）
- 補助: pdfplumber（段組・表）/ Poppler `pdftotext`（比較用）
- OCR: OCRmyPDF + Tesseract（`jpn`, `jpn_vert`）/ PaddleOCR（難例）
- 構造化: pymupdf4llm（Markdown化）/ Unstructured（セクション・要素分割）

### フェーズ
1) 識別・品質計測
   - born-digital vs scanned の判定、`ToUnicode`有無、縦書き率、抽出文字率、ページ画像密度。
2) 第1層抽出
   - PyMuPDF/pypdfium2でblocks/spans/bbox/画像を取得。rawテキストと版面情報を保存。
3) レイアウト補強
   - pdfplumberで段組再構成・表検出。Poppler出力と突き合わせ、差異が大きい箇所をフラグ。
4) フォールバック（必要時）
   - OCRmyPDFでテキスト層を埋め込み（Tesseract `jpn`/`jpn_vert`）。PaddleOCRで困難ページを差し替え。
5) 構造化
   - pymupdf4llmでMarkdown化。Unstructured hi_resでセクション/段落/図表キャプションを要素化。
6) セクション検出
   - 「未来ビジョン」「将来展望」「今後の課題」「Societal Impact」等の同義見出しを検出し、`section_map`を作成。
7) 成果物出力
   - `PaperIngestClass`に本文・図版・レイアウト・品質指標・検出セクションを格納。

### 品質ゲートとフォールバック規則（例）
- `quality.char_coverage < 0.70` または `quality.cjk_ratio < 0.40` → OCRへフォールバック。
- `quality.vertical_text_ratio > 0.30` → `jpn_vert`モデルを使用。
- 連続する空白/改行異常やCID化け率が閾値超過 → 該当ページのみOCR差し替え（ページ単位ハイブリッド）。

### 出力クラス（前処理成果）
```json
// PaperIngestClass
{
  "id": "paper-001",
  "source": {"path": "", "hash": "", "pages": 12},
  "metadata": {"title": "", "authors": [""], "year": 2024, "venue": "WISS"},
  "extraction_method": ["pymupdf", "pdfplumber"],
  "quality": {
    "char_coverage": 0.92,           // 推定全文量に対する抽出率
    "cjk_ratio": 0.76,               // CJK比率
    "vertical_text_ratio": 0.12,
    "ocr_applied_pages": [3,4],
    "notes": ["p4-p5はOCR差し替え"]
  },
  "layout": {
    "pages": [
      {
        "page": 1,
        "blocks": [
          {"bbox": [0,0,100,100], "text": "...", "spans": 12}
        ],
        "images": ["page-1-fig-1.png"]
      }
    ]
  },
  "text": {
    "markdown": "...",               // pymupdf4llmの出力
    "by_section": {
      "abstract": "...",
      "introduction": "...",
      "future_vision": "..."
    }
  },
  "section_map": [
    {"name": "future_vision", "page": 8, "offset": 12345, "aliases": ["将来展望", "未来ビジョン"]}
  ],
  "assets": {
    "images": [{"id": "fig1", "path": "page-1-fig-1.png", "caption": "..."}]
  }
}
```

### 運用ノート
- 許諾/ライセンス: 収集・OCR・加工の範囲を明文化し、引用ルールを徹底。
- キャッシュ: ハッシュベースで成果物を再利用。ページ単位での再処理を許容。
- 国際化: 日本語を既定としつつ、英語/中韓の混在に備え言語検出を実装。
- ログ: 品質指標とフォールバック決定を保存（再現性）。

## 入出力チャネルとUI
本システムの主要アウトプットはWebアプリとSlack連携。インプットは「論文PDF」および「論文外の自由入力（意図・未来ビジョン・外部制約）」を受け付ける。

### 入力チャネル
- 論文PDF: `PaperIngestClass`としてAgent 1に供給。
- 自由入力（任意）: 研究の意図/未来ビジョン/論文化できない示唆/制約/希望作風を`UserPromptClass`で受け取る。
- 追加メタ: `AuthorClass`/`ArtistClass`（任意）、利用許諾/公開範囲、禁則事項。

```json
// UserPromptClass（論文外の自由入力）
{
  "id": "prompt-001",
  "research_intent": "この研究の狙い/背景/価値仮説",
  "future_vision": "理想的な将来像（論文化が難しい暗黙も可）",
  "constraints": ["できないこと/避けたい表現/倫理配慮"],
  "applications": ["想定ユースケース/適用領域"],
  "keywords": ["キーワード"],
  "author_pref": {"authors": [""], "tone": ""},
  "artist_pref": {"artists": [""], "style_tags": [""]},
  "privacy": {"share": "private|team|public", "expires": null},
  "notes": ["雑多なメモ"]
}
```

### セッション管理
複数入力を束ね、生成物を追跡するための最小単位。

```json
// SessionClass
{
  "id": "sess-001",
  "created_at": "2025-10-19T12:00:00Z",
  "owner": {"user": "", "workspace": ""},
  "inputs": {"paper": "paper-001", "prompt": "prompt-001"},
  "agents": ["0-preprocess", "1-vision", "2-scaffold", "3-composer", "3b-illustration", "4-scenario"],
  "status": "draft|running|needs-review|awaiting-user|done",
  "decisions": ["dec-vision-ok", "dec-structure", "dec-style", "dec-illustration", "dec-finalize"],
  "agreements": {
    "vision_ok": false,
    "structure": {"act_structure": null, "pov": null, "timeline": null},
    "style": {"tone": null, "narrative_voice": null},
    "illustration": {"budget": null, "policy": null},
    "final": {"length_ok": false, "cast_ok": false}
  },
  "artifacts": {
    "visions": ["vision-001"],
    "components": ["comp-001"],
    "scripts": ["scs-001", "ics-001"],
    "scenario": "scenario-001"
  }
}
```

### ユーザーフィードバックと合意フロー（必須チェックポイント）
- 1) Vision確認（Agent 1 後）
  - 内容: `VisionClass[]`の要旨/狙い/未来ビジョンが適切か。誤読・抜けの修正。
  - 入力: Yes/No（修正点メモ可）。決定: `agreements.vision_ok=true`。
- 2) 方向性の選択（Agent 2 前）
  - 内容: `ComponentsClass.modality`、想定オーディエンス、`timeline`（year/era）。
  - 入力: 単一/複数選択。決定: `agreements.structure.timeline`ほかに反映。
- 3) 構造/文体の合意（Agent 2→3）
  - 内容: `act_structure`（three/four/journey）、`pov`、`narrative_voice`、`AuthorClass.tone`。
  - 入力: ラジオ/セレクト。決定: `agreements.structure`/`agreements.style`を確定。
- 4) イラスト選定（Agent 3b）
  - 内容: `selection.budget`/`policy`、上位ページ案の採否、作風（`ArtistClass`）。
  - 入力: スライダー/チェック。決定: `agreements.illustration`確定。
- 5) 最終確認（Agent 4 後）
  - 内容: 文字数（1000±許容）、登場人物上限、引用/安全チェック。
  - 入力: Yes/No（修正依頼可）。決定: `agreements.final.length_ok`, `cast_ok`。

ユーザー応答が必要な場面では`Session.status=awaiting-user`とし、Orchestratorは次ステージを保留する。

### 多様な選択を引き出す設計（プロンプティング/フォーマット）
- 目的: 似通った選択肢を避け、異なる軸で十分に広がった候補を提示し、ユーザーが自分の意図に照らして比較・選択しやすくする。
- プロンプト方針（エージェントの指示例）
  - Axes-of-Variation: `modality, era, audience, act_structure, pov, tone, value_lens(privacy/inclusion/sustainability)`の少なくとも3軸以上が相互に異なるように`k`個の案を生成。
  - Diversity Penalty: 文章類似度/キーワード重複にペナルティ（例: bigram overlap < 0.4, cosine sim < 0.8）。
  - Coverage Constraints: 近未来/遠未来を各1案以上、保守的/大胆の二極を必ず含める、などの充足制約を明記。
  - Counterfactual/Wildcard: 1枠は「敢えて」反対解（大胆/異色）を要求。
  - Rationale Snippets: 各案に「なぜ多様で、ビジョンとどう整合するか」を1行で添える。
- フォーマット方針
  - Option Card: 各案をカード化（軸値、ワンライナー、短い抜粋、リスク/適合度、diversity_score）。
  - Comparative Matrix: 行=案, 列=軸（modality/era/tone/pov/act_structure/values）で差異を一望化。
  - Elicitation UI: 「Top-2を選び、理由を一言」→次世代案で反映（能動学習）。
  - Spectrum Toggles: Safe↔Bold, Pragmatic↔Speculative等のスライダーで意向を数値化し、生成ポリシーに反映。
  - Exploration Rate: `explore=0.0–1.0`で新奇性優先度を切替。

```json
// DiversityPolicyClass
{
  "k": 6,
  "axes": ["modality", "era", "audience", "act_structure", "pov", "tone", "value_lens"],
  "constraints": [
    {"key": "era", "coverage": "at_least_one", "values": ["near-future", "far-future"]},
    {"key": "modality", "coverage": "min_distinct", "count": 3},
    {"key": "risk_profile", "coverage": "include", "values": ["safe", "bold"]}
  ],
  "penalties": {"similarity": 0.6, "keyword_overlap": 0.5},
  "selection_method": "mmr|dpp|heuristic",
  "wildcard": 1,
  "explore": 0.4,
  "value_lenses": ["privacy", "inclusion", "sustainability"]
}

// OptionCard
{
  "id": "opt-001",
  "kind": "theme|scenario|style",            // 泡が表す世界の種別
  "axes": {"modality": "robot", "era": "near-future", "audience": "children", "act_structure": "three-act", "pov": "first", "tone": "warm", "value_lens": "inclusion"},
  "scenario_hook": "保健室ロボが秘密の悩みに寄り添う一夜",
  "world_glimpse": "白濁した廊下の光に、銀の耳だけがやわらかく瞬く。",
  "micro_excerpt": "わたしは銀色の耳を持つ友だちに話しかけた。",
  "rationale": "ビジョンのアクセシビリティ価値を強調しつつ児童向けに翻訳",
  "risks": ["過度な擬人化"],
  "diversity_score": 0.72,
  "novelty": 0.65,
  "alignment": 0.82
}

// OptionSetClass（Decisionに紐づく候補セット）
{
  "decision_id": "dec-structure",
  "policy": {"ref": "divpol-001"},
  "options": ["opt-001", "opt-002", "opt-003", "opt-004", "opt-005", "opt-006"],
  "coverage": {"axes": {"modality": ["robot", "VR", "AGI"], "era": ["near-future", "far-future"]}, "gaps": ["pov=omniscient"]}
}
```

実装ノート: 近似類似度は埋め込み＋MMR、もしくは単純なn-gram重複で十分。UIはカード/マトリクス/スライダーのいずれかを最低1つ提供。

### Webアプリ（アウトプット）
- 主要機能
  - アップロード（PDF）+自由入力フォーム（`UserPromptClass`）
  - エージェント駆動の進行表示（各段のI/Oプレビューと介入点）
  - イラスト選定ビュー: `IllustrationComponentsScriptClass.selection`と`page_scripts`を可視化・編集
  - シナリオエディタ: 1000字目安、年代/登場人物のチェック、脚注・引用プレビュー
  - エクスポート: JSONバンドル/Markdown/画像アセット
- 最低限のナビゲーション
  - セッション一覧→詳細→各段のタブ（Paper/Vision/Components/Scripts/Scenario）
  - 差分表示（前回との変更点）

### Slack連携（アウトプット）
- 想定ユースケース
  - 研究チャンネルでPDFを投げる→ボットが要約/`VisionClass`を返信
  - スラッシュコマンドで進行を操作（抽出→構成案→シナリオ生成）
  - スレッドで選定ページの投票/修正
- コマンド例
  - `/sfproto extract` PDF添付 or リンク → `PaperIngestClass`を生成
  - `/sfproto vision` セクション検出と`VisionClass`出力
  - `/sfproto scaffold` `ComponentsClass`/構成案を出力
  - `/sfproto compose` シナリオ素案（1000字目安）
  - `/sfproto illustrate` イラスト候補一覧を提示（ボタンで採否）
- 実装ノート
  - Slack OAuth（bot token, event subscription）
  - event: `app_mention`, `message.channels`, `file_shared`, slash commands
  - 返信はスレッド、長文はファイル添付/外部URLで配布
  - インタラクティブ: Block Kitのボタン/セレクト/モーダルで決定事項を収集（例: 構造/文体/イラスト予算）。
  - 多様化UI: 6枚のOption Cardをカルーセル/チェックボックスで提示、`Top-2 + 理由`を必須入力にして学習反映。

### API（最小）
- `POST /api/uploads/pdf` → `paper_id`
- `POST /api/prompts` → `prompt_id`
- `POST /api/sessions` {paper_id, prompt_id}
- `POST /api/sessions/{id}/run` {stages:["1","2",...]} → 非同期ジョブ
- `GET /api/sessions/{id}` → セッション/成果物
- `GET /api/sessions/{id}/export` → Zip(JSON+MD+assets)
- `GET /api/sessions/{id}/decisions` → 未決定事項一覧
- `POST /api/decisions/{decision_id}/respond` {choice, comment?}
 - `GET /api/decisions/{decision_id}/options` → `OptionSetClass`
 - `POST /api/decisions/{decision_id}/options/regenerate` {policy_overrides?}

## 実行オーケストレーション（正確な実行保証）
Orchestratorは、ステージ依存/DAG、品質ゲート、再試行、監査ログ、通知を一元管理する。Criticは内容品質、Orchestratorは手続きと規約遵守を担う。

### ステージDAGとステート
- 依存: 0-preprocess → 1-vision → 2-scaffold → 3-composer → 3b-illustration → 4-scenario → 5-critic（任意）
- ステート: `pending | running | succeeded | failed | skipped`
- ポリシー: `failed`時はゲートにより`retry`/`rollback`/`skip`を選択。

### 品質ゲート（例）
- 0→1: `PaperIngestClass.quality.char_coverage ≥ 0.70`。未満はOCRフォールバック後に再抽出。
- 1→2: 各`VisionClass`に`citations`が1件以上 or `仮説`タグ付け必須。
- 1→2+: `agreements.vision_ok=true`（ユーザー承認が必要）。
- 2→3: `ComponentsClass.modality`の未設定不可。`open_questions`があればUIで解決 or 暫定で進行を明示。
- 2→3+: `agreements.structure.*`（`act_structure/pov/timeline`）が選択済み。
- 3→3b: `page_scripts`の候補が`selection.budget`±20%以内。
- 3b→4: `selection.policy`に基づくスコア上位が採択されていること。
- 4完了: `ScenarioClass.length_chars`が`target±tolerance`、`cast_limits`を超過しない、参照整合（anchors/props）が取れている。

### 決定要求スキーマ
```json
// DecisionClass（ユーザー合意が必要な項目）
{
  "id": "dec-structure",
  "session_id": "sess-001",
  "stage": "2-scaffold",
  "type": "choice|confirm|text",
  "title": "シナリオ構造の選択",
  "prompt": "Act構成/視点/語り口を選んでください",
  "options": [
    {"key": "act_structure", "values": ["three-act", "four-act", "journey"], "required": true},
    {"key": "pov", "values": ["first", "third", "omniscient"], "required": true},
    {"key": "narrative_voice", "values": ["neutral", "lyrical", "clinical"], "required": true}
  ],
  "multi_select": false,
  "required": true,
  "status": "pending|resolved|expired",
  "due_at": null,
  "created_at": "...",
  "resolution": {"by": "user-id", "at": "...", "choices": {"act_structure": "three-act"}, "comment": ""}
}
```

### 再試行/決定性
- リトライ: エラー種別に応じて`retries.max`と`backoff`を設定。
- 決定性: 生成時に`seed`を固定（Illustration/LLM）。同一`inputs + seed`→同一出力を指向。
- 中断/再開: ステージ単位のチェックポイント。成功済みの成果物は再利用。

### 監査・観測
- 監査: 全ステージの入出力摘要、ゲート判定、再試行履歴を保存。
- 観測: メトリクス（所要時間、失敗率、ゲート不通過率）を可視化。通知（Slack）で重要イベントを配信。

### ラン実体
```json
// RunClass（オーケストレーション単位）
{
  "id": "run-001",
  "session_id": "sess-001",
  "created_at": "2025-10-19T12:00:00Z",
  "status": "running|succeeded|failed|canceled",
  "stages": [
    {"name": "0-preprocess", "state": "succeeded", "started_at": "...", "ended_at": "...", "retries": 0},
    {"name": "1-vision", "state": "running",   "started_at": "...", "retries": 1}
  ],
  "gates": [{"stage": "1-vision", "gate": "citations_min", "pass": true}],
  "params": {"seed": 123, "retries": {"max": 2, "backoff": "exp"}},
  "errors": [],
  "logs": ["...summary only..."]
}
```

### Orchestrator API 追加
- `POST /api/sessions/{id}/runs` → `run_id`を返す（新規Run作成）
- `GET /api/runs/{run_id}` → Run状態/ゲート/ログ
- `POST /api/runs/{run_id}/retry` {stage?:"2-scaffold"}
- `POST /api/runs/{run_id}/cancel`
- `POST /api/runs/{run_id}/resume`

### Slack通知（例）
- 重大: ゲート不通過/連続失敗/最終成功をチャンネルに投稿。
- 操作: スレッドで`Retry this stage`/`Skip gate`アクションボタン。

### セキュリティ/プライバシ
- 権限: セッションはデフォルトprivate。共有は明示設定。
- データ保持: 期限/削除API。アップロードPDFのハッシュ化キャッシュ。
- ログ/トレース: Bot/WEBの操作履歴を監査可能に。

## UIコンセプト（三層）— ALTER EGOインスパイア
目的: モノクロ×哲学的テキストの“ALTER EGO”の没入感に、SF的な泡世界（胡蝶の夢）を重ねる。孵化の待機演出ではなく、「複数の世界（シナリオ/テーマ）が現れ、その中から選ぶ」体験を中心に据える（神の視点で世界を選ぶ）。

### イメージ改装レイヤー（Skin/Art Direction）
- ビジュアル言語
  - モノクロ基調に一点蛍光色（シアン/マゼンダ）で状態を示す（待機=微光、処理中=脈動、完了=点灯）。
  - 余白重視の活字UI（明朝/セリフ系）と幾何学アイコン（円環・ルーン）。
  - 泡世界（Bubble Shader）: 半透明の球体の内部に流体ノイズと紙片。各泡は一つの「可能な世界」。
- 演出
  - 暗号化の儀式: PDF投入直後に短いグリッチ/サイファーレイン（<1.5秒）→「Encrypted」バッジ点灯（待機で魅せない）。
  - 生成: シャボン玉が一斉に立ち現れ、その内部にミニチュア世界が見える。装丁アニメは選択後に進行。
  - 名言/抜粋: ALTER EGO的な短い引用を時折挿入（UserPrompt/論文から抽出した一文）。
- 実装トークン
  - Three.js + ShaderMaterial（ray-marching, SDF, simplex noise）。
  - 低刺激モード: CSS/Canvasへのフォールバック、アニメ速度低減、ハイコントラスト配色。

### 体験レイヤー（Interaction/Flow）
- フロー
  1) 提出 Submit: PDFドロップ→短い暗号化演出→即座に選択フェーズへ。
  2) 生成 Generate: 3つ（〜6つ）の泡（Option Cards）が現れ、それぞれ「シナリオ/テーマ/文体」の世界が可視化される。ホバーで“呼吸”、内部の抜粋や価値レンズを覗ける。
  3) 選択 Choose: 「ここに3つの世界がある。さぁ選べ。」Top-2＋理由を入力（能動学習）。
  4) 合意 Agree: Act構成/POV/トーンのスライダー/セレクト。未合意の項目は泡が曇る演出で促す。
  5) 織上 Weave: ページが束ねられ、イラスト候補（点灯したページ）のミニ窓が光る。
  6) 発行 Release: 書き出し+Slackへ儀式報告（進捗GIF/画像）。
- マイクロインタラクション
  - 泡の大きさ= `alignment`、脈動= `uncertainty`、境界色= `era`、表面紋様= `modality`。
  - ページめくりの慣性/反り。決定時にルーンが一つずつ点灯（品質ゲート通過）。
- Slack等価表現
  - 泡世界のサムネイル（GIF/静止）＋Option CardをBlock Kitで横並び提示。チェック＋理由をモーダル入力。

### 処理レイヤー（Logic Mapping）
- 泡⇔OptionCard: `kind`で泡の種類（theme/scenario/style）を分け、`axes`/`scores`を視覚属性にマッピング（色相=era, 輝度=novelty, 太さ=diversity）。
- 儀式⇔Run/Gates: 0→1→2→3→3b→4→5の通過に応じてリングのルーンが点灯。失敗は点滅+再試行UI。
- 合意⇔Decisions: `DecisionClass`の未解決→`Session.status=awaiting-user`。解決時に泡の曇りが晴れる演出。
- 織上⇔Artifacts: `ScenarioClass`/`IllustrationComponentsScriptClass`の更新毎に装丁が一段進む（背表紙→帯→小口染め）。

### 実装ヒント（短期PoC）
- Web: React + Three.js（泡コンポーネント）+ Framer Motion（UIトランジション）+ XState（オーケストレーション状態同期）。
- Slack: Block Kitでカード群、モーダルでTop-2＋理由。儀式GIFはWeb側でレンダリング→URL添付。
- アセット: 1色差しのアイコン/ルーンSVG。引用は`PaperIngestClass.text.by_section`から安全に抽出。

## 処理アーキテクチャ（Processing Architecture）
目的: 各ステージの責務・入出力・品質ゲート・再試行・保存先を明確化し、Orchestratorが確実に流せる実装骨子を提示する。

### ランタイム構成（参考）
- Web App: React/Next.js（任意）
- API: FastAPI/Express（任意）。認可ミドルウェア、Rate Limit。
- Worker: 非同期実行（Celery/RQ/BullMQ等）。
- Orchestrator: ステート機械（XState/自前FSM）+ Jobキュー連携。
- Storage: オブジェクト（S3/ローカル）、DB（PostgreSQL/SQLite）、Search/Vector（任意）。
- Cache/Queue: Redis（idempotency・レート・短期キャッシュ・ワークキュー）。

### キューと同時実行
- キュー名: `q-preprocess`, `q-vision`, `q-scaffold`, `q-compose`, `q-illustration`, `q-scenario`, `q-critic`。
- 同時実行: `vision/scaffold/compose`はCPU/LLM、`illustration`はGPU負荷。並列度は別々に制御。
- 冪等性キー: `idempotency = sha1(session_id + stage + input_hash + model_version)`。
- 出力のバージョン: `artifacts/*/v{n}.json`で保存。RunClassで参照バージョンを固定。

### データ配置（例）
- `sessions/{sess}/inputs/{paper|prompt}.json`
- `sessions/{sess}/artifacts/vision/{id}.v1.json`
- `sessions/{sess}/artifacts/components/{id}.v1.json`
- `sessions/{sess}/artifacts/scripts/{scs|ics}.v1.json`
- `sessions/{sess}/artifacts/scenario/{id}.v1.json`
- `sessions/{sess}/previews/{gif|png}`（UI/Slack用の可視化）
- `sessions/{sess}/runs/{run}.json`（RunClassのスナップショット）

### ステージ仕様（I/O・手順・ゲート）
- 0 Preprocess（Paper → PaperIngestClass）
  - Input: PDF
  - Steps: born-digital判定→PyMuPDF/pypdfium2抽出→pdfplumber補強→必要に応じOCRmyPDF/PaddleOCR→pymupdf4llm/UnstructuredでMarkdown/要素化→未来ビジョン見出し検出。
  - Output: `PaperIngestClass`
  - Gates: `char_coverage≥0.70`、縦書き率に応じモデル選択、OCR差し替えログ必須。
- 1 Vision Extractor（PaperIngest → VisionClass[]）
  - Input: `PaperIngestClass`, `UserPromptClass?`
  - Steps: 見出し/段落の候補抽出→LLMで要旨/狙い/価値仮説→応用キーワード抽出→引用位置（section/para）付与→信頼度付与。
  - Output: `VisionClass[]`
  - Gates: `citations>=1` or `仮説タグ`、重複visionの類似度<0.85。
- 2 Narrative Scaffolder（Vision → Components）
  - Input: `VisionClass[]`, `agreements.vision_ok`
  - Steps: 価値レンズ/志向→`modality`候補（対応表+LLM）→`capability_set/interaction/environment`推定→`open_questions`抽出。
  - Output: `ComponentsClass`
  - Gates: `modality`必須、`open_questions`はDecisionとして提示可。
- 2' Options（多様案）
  - Input: Vision/Components、`DiversityPolicyClass`
  - Steps: エージェントがOptionCardをk件生成（MMR/DPP/ヒューリスティックで多様化）→OptionSetを保存。
  - Output: `OptionSetClass`
  - Gates: `coverage`と`penalties`を満たす。
- 3 Story Composer（Components → Scripts）
  - Input: `VisionClass`, `ComponentsClass`, `Author/ArtistClass?`, `agreements.structure/style`
  - Steps: Act構造を決定→ビート/シーン/小道具→絵本向け台本断片→イラスト指示の雛形（ページ候補含む）。
  - Output: `ScenarioComponentsScriptsClass`, `IllustrationComponentsScriptClass`(候補)
  - Gates: POV/Act/Voiceが合意済み、台本とイラスト候補の整合。
- 3b Illustration Selector（Scripts → IllustrationComponents）
  - Input: `IllustrationComponentsScriptClass`
  - Steps: スコアリング（importance/visual_salience/diversity/coverage）→`budget`内で採択→プロンプト方式選択（structured推奨）→一貫性アンカー作成。
  - Output: 採択済み`IllustrationComponentsScriptClass`
  - Gates: 予算・分布・一貫性anchorの充足。
- 4 Scenario Realizer（Scripts → Scenario）
  - Input: `ScenarioComponentsScriptsClass`, `IllustrationComponentsScriptClass`, `agreements.*`
  - Steps: 1000字±許容で本文生成→`timeline/cast_limits`遵守→`page_map`整備（24/32のレイアウト原則）→`illustration_briefs`集約。
  - Output: `ScenarioClass`
  - Gates: 文字数/登場人物/参照整合（anchors/props）、不適切表現のnegativeチェック。
- 5 Critic/Evaluator（任意）
  - Input: 全成果物
  - Steps: 用語整合/矛盾検出/スタイル逸脱/倫理フラグ→改善提案パッチを生成。
  - Output: 批評レポート、修正案
  - Gates: 重大フラグで保留→ユーザー確認。

### ページ割り/配置の指針（シナリオ×絵本）
- 文字数とページ: 本文は約1000字、絵本は24/32ページを基準。テキストはページ見出し/段で分解せず、重要ページのみキャプション付。
- `page_map`ヒューリスティック: 導入→きっかけ→展開→山場→解決→終章で配点。見開きは世界観提示・山場に割当。
- 低視覚シーン: spot図版・象徴モチーフで補完し、長文の塊を避ける。

### 失敗モードと対処
- PDF固有: ToUnicode欠落/縦書き/画像混在→OCRフォールバック、ページ単位差し替え。
- 多様化失敗: 類似度高すぎ→`explore↑`/`penalties↑`/軸の強制カバレッジ。
- 文字数逸脱: 反復生成でtolerance内に収束（長い→縮約、短い→増補）。
- 整合性崩れ: Criticの差分パッチ適用→再評価。

### 観測・計測（最小）
- パフォーマンス: ステージ時間、失敗率、再試行回数、OCR率。
- 品質: 可読性/共感度のユーザー評価、Criticスコア、採択率（Top-2→最終反映率）。
- コスト: LLMトークン/画像生成コスト、1セッション当たりの目安。

### 決定性・再現性
- `seed`固定、モデル/プロンプトのバージョン固定（RunClassに記録）。
- 成果物は参照のみで再生成しない（idempotencyキーが一致する限り）。

### セキュリティ/プライバシ（処理系）
- アップロード直後に暗号化保存（at-rest）。伝送はTLS。
- 権限境界: Slack/WEBのスコープ最小化、共有スコープ厳格化。
- ログには本文を保存しない。要約/ハッシュ/メタのみ。

### シナリオ制約（長さ・年代・登場人物数）
- 文字数: 日本語約1000字を目安（`length_chars.target=1000`、許容±10–20%を`tolerance`で指定。推奨は0.15）。
- 年代/時代設定: `timeline.year`と`timeline.era`を明示。用語・小物・技術水準が年代と矛盾しないかCriticで検査。
- 登場人物数: 認知負荷を抑えるため`cast_limits.max_total`≦4を推奨（主要2+助演2）。
- 整合性: `ScenarioComponentsScriptsClass.scenes[*].characters`の重複/別名を正規化し、`consistency.character_anchors`と同期。

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
