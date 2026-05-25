---
name: design-setup
description: "TEKION Slide Generator v4 のブランドデザインを対話形式でカスタマイズするセットアップスキル。branding.md の Asset 層・Guideline 層（ロゴ／配色／フォント／トーン／ブランドムード／参考デザイン／禁止事項）を 1 問ずつ対話で設定し、次回のスライド生成に自動適用されるプリセットを生成する。「デザインを設定したい」「ブランドをカスタマイズしたい」「テーマを変えたい」「自社カラーに設定して」「ロゴを設定して」で発動。"
---

# TEKION Slide Generator — デザインセットアップ

## 概要

このスキルは TEKION Slide Generator v4 のブランドデザインを **対話形式** でカスタマイズし、
プリセットファイルを生成して次回のスライド生成に自動適用する。

設定対象は `${V4_SKILL_DIR}/docs/branding.md`（インストール後の絶対パスで参照）の：

- **Asset 層**: ロゴ
- **Guideline 層**: 配色 / フォント / トーン / ブランドムード（キーワード・使用ルール・参考デザイン）/ 禁止事項
- **デフォルトスタイル**: balanced / visual

**Template 層**（`templates/*.j2` の Fork）は対象外。Step 13 で「閲覧専用の案内」のみ提供する（非推奨）。

## 起動メッセージ（最初に必ず表示）

実行前チェックの直前に、以下を1度だけ表示してユーザに全体像を伝える：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEKION Slide Generator — デザインセットアップ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

このスキルは対話形式でブランドデザインを設定し、
次回のスライド生成に自動適用するプリセットを生成します。

【全 13 ステップ】（任意ステップはスキップ可）
  Step 0  : 既存設定の確認
  Step 1  : ブランド名
  Step 2  : ロゴ設定(任意)               [Asset]
  Step 2.5: ロゴから色推測(ロゴ指定時のみ)[Asset → Color]
  Step 3  : メインカラー Primary         [Color]
  Step 4  : アクセントカラー(任意)       [Color]
  Step 5  : セマンティック・グレー(任意) [Color]
  Step 6  : フォント雰囲気               [Font]
  Step 7  : トーン                       [Mood]
  Step 8  : ブランドキーワード(任意)     [Mood]
  Step 9  : 使用ルール(過去スライド分析対応) [Mood]
  Step 10 : 参考デザイン(任意)           [Mood]
  Step 11 : 禁止事項(任意)               [Forbidden]
  Step 12 : デフォルトスタイル           [Style]
  Step 13 : 上級設定の確認(任意)         [Advanced]
  → サマリー確認 → プリセット生成

所要時間目安: 5-10 分
詳細解説: project/docs/skill/design-setup-skill.md

【ルール】
- 質問は 1 問ずつ。前の回答を受けてから次へ
- 任意項目は「スキップ」「後で」「なし」でデフォルト採用
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 定数

```bash
DESIGN_SKILL_DIR="<このSKILL.mdが置かれているディレクトリの絶対パス>"
V4_SKILL_DIR="${DESIGN_SKILL_DIR}/../tekion-slide-generator-v4"
PRESETS_DIR="${V4_SKILL_DIR}/references/presets"
ASSETS_DIR="${V4_SKILL_DIR}/assets"
ACTIVE_PRESET_FILE="${PRESETS_DIR}/.active_preset"
ACTIVE_STYLE_FILE="${PRESETS_DIR}/.active_style"
TEMPLATE_FILE="${PRESETS_DIR}/example-preset.md"

# Python 実行コマンドを環境に応じて自動選択
# macOS/Linux は通常 python3、Windows は python のことが多い
command -v python3 >/dev/null 2>&1 && PY=python3 || PY=python
```

## 実行前チェック

V4スキルディレクトリと Python 実行環境の存在を確認する：

```bash
ls "${V4_SKILL_DIR}/references/presets/" 2>/dev/null || echo "ERROR: v4スキルが見つかりません"
"${PY}" --version 2>/dev/null || echo "ERROR: Python が見つかりません（python3 または python が PATH 上に必要）"
```

見つからない場合はユーザに `tekion-slide-generator-v4` のパスまたは Python インストール状況を確認して修正する。

---

## 対話フロー

**ルール：質問は必ず 1 問ずつ。次の質問は前の回答を受け取ってから行う。**
ユーザが「スキップ」「後で」「なし」と答えた場合はデフォルト値を使って次へ進む。

各ステップの末尾に **[Asset]** / **[Color]** / **[Font]** / **[Mood]** / **[Forbidden]** / **[Style]** / **[Advanced]** タグを付け、ユーザが現在どの層を設定しているか分かるようにする。

---

### Step 0: 既存設定の確認

```bash
if [ -f "${ACTIVE_PRESET_FILE}" ]; then
  CURRENT_PRESET=$(cat "${ACTIVE_PRESET_FILE}")
  echo "現在のアクティブプリセット: ${CURRENT_PRESET}"
else
  echo "アクティブプリセット: 未設定（example-preset.md を使用中）"
fi

if [ -f "${ACTIVE_STYLE_FILE}" ]; then
  CURRENT_STYLE=$(cat "${ACTIVE_STYLE_FILE}")
  echo "現在のアクティブスタイル: ${CURRENT_STYLE}"
else
  echo "アクティブスタイル: 未設定（balanced を使用中）"
fi

echo ""
echo "=== 利用可能なプリセット ==="
for f in "${PRESETS_DIR}"/*.md; do
  [ -e "$f" ] || continue
  basename=$(basename "$f")
  case "$basename" in
    "example-preset.md")
      echo "📘 example-preset.md（汎用・balanced系）"
      echo "   用途: 営業資料、提案書、社内説明資料、Pitch.com / Figma Slides 風"
      echo "   判断基準: テキスト＋ビジュアル両立、3-5項目の箇条書きで詳細を伝えたい"
      ;;
    "example-preset-presentation.md")
      echo "🎤 example-preset-presentation.md（登壇・visual系）"
      echo "   用途: ピッチデッキ、登壇プレゼン、TED Talk / Apple Keynote 風"
      echo "   判断基準: 1スライド1メッセージ、ビジュアル60%以上、文字数厳格制限"
      ;;
    *)
      # カスタムプリセット: H1 から BRAND_NAME を抽出
      brand=$(head -1 "$f" | sed -n 's/^# Design Specifications — \(.*\)/\1/p')
      [ -z "$brand" ] && brand="（ブランド名不明）"
      echo "🏷  ${basename}（カスタム）"
      echo "   ブランド: ${brand}"
      ;;
  esac
  echo ""
done
```

表示後、「新規作成しますか？それとも既存のプリセットを選択しますか？」と聞く。

- **既存を選択** → ファイル名を `.active_preset` に書き込み、Step 12（スタイル選択）へジャンプ。
  - 以降は `FLOW_MODE=existing` として扱う（後段の「サマリー確認」「プリセット生成」セクションはスキップし、Step 12 のスタイル書き込み直後に「完了メッセージ」へ直行する）。
  - `BRAND_NAME` / `SLUG` / `PRIMARY` などの新規作成用変数は未定義のままで OK（後段で参照しない）。
- **新規作成** → `FLOW_MODE=new` として Step 1 へ進む。Step 12 完了後は「サマリー確認 → プリセット生成 → 完了メッセージ」を順に通る。

---

### Step 1: ブランド名

> 「会社名またはブランド名を教えてください。（例: TEKION、Vibe Coder、株式会社◯◯）」

入力を受け取ったら：
- `BRAND_NAME` = 入力値（表示用、原文のまま保持）
- `SLUG` はスクリプトで決定的に生成する（LLM の文字列変換は揺らぐので避ける）：

```bash
SLUG=$("${PY}" "${DESIGN_SKILL_DIR}/scripts/slugify.py" "${BRAND_NAME}")
# 例: "TEKION" → tekion / "Vibe Coder" → vibe-coder
#     "株式会社ABC" → abc / "TEKION株式会社" → tekion
#     "あいうえお"（ASCII 文字なし）→ brand（フォールバック）
```

- `PRESET_FILE="${PRESETS_DIR}/${SLUG}.md"`

同名のプリセットが既に存在する場合は「上書きしますか？」と確認する。
ASCII 文字を含まないブランド名で `SLUG=brand` にフォールバックした場合は、ユーザーに「ファイル名は `brand.md` になります。変更しますか？」と確認する。

---

### Step 2: ロゴ設定（任意） **[Asset]**

> 「ロゴファイルのパスを教えてください。（例: ~/Downloads/logo.png）
> PNG 推奨（透過対応）、横長比率（3:1〜6:1）が理想です。
> 推奨解像度: 1200px 以上（幅）。
> スキップすると現在のロゴをそのまま使います。」

- パスが指定された場合:

ユーザー入力は `~/Downloads/...` のようにチルダを含むことが多い。クォート内では `~` がシェル展開されないので、`eval` か `${variable/#~/$HOME}` で展開してから扱う。
また「既存ロゴを退避 → コピーに失敗 → ロゴ無し」という事故を防ぐため、**先に入力ファイルの存在を確認し、退避と上書きをアトミックに行う**。

```bash
# 1) チルダ展開（クォート内では展開されないため明示的に処理）
USER_LOGO_PATH="${USER_LOGO_PATH/#\~/$HOME}"

# 2) 入力ファイルの存在を確認してから破壊的操作に進む
if [ ! -f "${USER_LOGO_PATH}" ]; then
  echo "ERROR: ロゴファイルが見つかりません: ${USER_LOGO_PATH}"
  echo "（既存ロゴは変更されていません）"
else
  # 3) 一時ファイルにコピー成功してから既存ロゴを退避・差し替え（失敗時もロゴを失わない）
  TMP_LOGO="${ASSETS_DIR}/logo.png.new.$$"
  if cp "${USER_LOGO_PATH}" "${TMP_LOGO}"; then
    [ -f "${ASSETS_DIR}/logo.png" ] && mv "${ASSETS_DIR}/logo.png" "${ASSETS_DIR}/logo.png.bak.$(date +%s)"
    mv "${TMP_LOGO}" "${ASSETS_DIR}/logo.png"
    echo "ロゴをコピーしました: ${ASSETS_DIR}/logo.png"
  else
    rm -f "${TMP_LOGO}"
    echo "ERROR: ロゴのコピーに失敗しました: ${USER_LOGO_PATH}"
    echo "（既存ロゴは変更されていません）"
  fi
fi
```

- スキップ → `LOGO_STATUS="変更なし"`

---

### Step 2.5: ロゴ画像の色解析（自動・ロゴ指定時のみ） **[Asset → Color]**

Step 2 でロゴが指定された場合（スキップでない場合）のみ自動実行する。
スキップされた場合は本ステップを飛ばして Step 3 へ進む。

**Claude が実行する処理：**

1. Read tool で `${ASSETS_DIR}/logo.png` を画像として読み込む
2. ロゴ上の主要色（背景以外で彩度が高く、占有面積が大きい色）を 1-3 個抽出
3. 抽出結果を以下の形式で表示：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ロゴから検出した主要色
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1) #XXXXXX  （◯◯系、最も占有面積が大きい）   ← Primary 候補
  2) #YYYYYY  （△△系）                          ← Accent Gold 候補（あれば）
  3) #ZZZZZZ  （◇◇系）                          ← 参考
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

4. これらは **Step 3 の Primary 候補・Step 4 の Accent 候補** として記憶し、
   それぞれのステップ冒頭で「ロゴから推測すると `#XXXXXX` 系が候補です」と提示する

**注意事項：**
- 視覚判断であり厳密なカラーピッカーではない（誤差あり）
- ユーザの確認・上書きを必ず受け付ける
- 検出に失敗した場合（モノクロロゴ等）は「色を抽出できませんでした」と表示して Step 3 へ進む

---

### Step 3: メインカラー（Primary） **[Color]**

> 「ブランドのメインカラーを教えてください。
> HEX コード（例: #EA5514）でも、言葉（例: オレンジ系、深い青）でも OK です。
> ※スキップすると青系（#2563EB）をデフォルトで設定します。」

入力を受け取ったら：
- HEX コード（`#XXXXXX` 形式）→ `PRIMARY` にそのまま格納
- 自然言語 → Claude が適切な HEX コードに変換し「`#XXXXXX`（◯◯系）でよいですか？」と確認
- スキップ → `PRIMARY=#2563EB`

`PRIMARY` が確定したら派生色をスクリプトで算出する（LLMの暗算より決定的・再現可能）：

```bash
"${PY}" "${DESIGN_SKILL_DIR}/scripts/derive_palette.py" "${PRIMARY}"
# 例: {"primary": "#EA5514", "primary_light": "#FDEEE8", "primary_dark": "#BB4410"}
```

返り値の JSON から `PRIMARY_LIGHT`（白に90%寄せ・黒テキスト前提の淡背景）と `PRIMARY_DARK`（黒に20%寄せ・白テキスト前提の濃背景）を取り出す。

> なぜ 90% / 20%: `#EA5514` → `#FEF0E8`（example-preset.md の Primary Light）から逆算した比率。
> Pattern C のカード背景・60-30-10 の30%領域・注目ボックスの背景など、**黒テキストを載せる前提の薄背景**として使うため、ほぼ白に近い淡さが必要。

算出値を表示し「この自動算出値でよいですか？個別指定したい場合は HEX を入力してください」と確認。

---

### Step 4: アクセントカラー（任意） **[Color]**

> 「アクセントカラー（データ可視化・差別化用）を設定します。
> 自動算出値で OK ならスキップしてください。
> ・Accent Teal: {ACCENT_TEAL_AUTO}（Primary の補色方向）
> ・Accent Gold: {ACCENT_GOLD_AUTO}（ハイライト用ゴールド）」

- 入力あり → HEX 変換して上書き
- スキップ → 自動算出値（Teal: `#0D7C72` 系 / Gold: `#C4880D` 系）を採用

---

### Step 5: セマンティック・グレースケール（任意） **[Color]**

> 「セマンティックカラー（Success / Warning / Error / Info）とグレースケールはデフォルト値を使いますか？
> カスタマイズしたい場合は HEX を順に入力してください（例: `success=#16A34A, error=#DC2626`）。
> スキップでデフォルト採用。」

デフォルト値：

| 色 | デフォルト HEX | 用途 |
|---|---|---|
| Success | `#1A7A3D` | 完了、正解、推奨 |
| Warning | `#C4880D`（Accent Gold と共用） | 注意、要確認 |
| Error | `#C52B2B` | エラー、禁止 |
| Info | `#2D6CB4` | 参考情報、補足 |
| Gray-900 | `#1A1A1A` | 見出しテキスト |
| Gray-700 | `#4A4A4A` | 本文テキスト |
| Gray-400 | `#9CA3AF` | キャプション |
| Gray-200 | `#E5E7EB` | 罫線 |
| Gray-100 | `#F3F4F6` | カード背景 |

---

### Step 6: フォント雰囲気 **[Font]**

> 「フォントの雰囲気を教えてください。具体的なフォント名ではなく "雰囲気ワード" で指定すると効きやすいです。
>
> 1) 太字の丸みのあるゴシック（親しみやすい・現代的）— デフォルト
> 2) シャープな極太サンセリフ（先進的・力強い）
> 3) クラシックなセリフ系（伝統的・知的）
> 4) 細身の現代的サンセリフ（ミニマル・洗練）
> 5) 自由記述（"角ばった厳格な" など）」

- 番号 → 対応する `HEADING_FONT_MOOD` / `BODY_FONT_MOOD` を採用
- 自由記述 → そのまま格納
- スキップ → 1) デフォルト

---

### Step 7: トーン **[Mood]**

> 「スライドのトーンを 1-2 文で教えてください。
>
> 例:
> ・「プロフェッショナルだが親しみやすい。シニア世代に安心感を与える温かみ」
> ・「先進的でクリーン。テック企業の洗練されたイメージ」
> ・「大胆でエネルギッシュ。ベンチャーの推進力を感じさせる」
>
> または以下から選択:
> 1) プロフェッショナル・信頼感
> 2) スタートアップ・モダン
> 3) 親しみやすい・温かみ
> 4) クール・先進的
> 5) スキップ（デフォルト: プロフェッショナル）」

入力を受け取ったら `TONE` を生成（自由記述ならそのまま、番号なら branding.md の例文を採用）。

---

### Step 8: ブランドキーワード **[Mood]**

> 「ブランドを表すキーワードを 3-5 個、カンマ区切りで教えてください。
> （例: energetic, confident, cutting-edge, human-warm）
> AI がモデルにブランドの肌触りを伝える際のヒントになります。
> スキップ可。」

- 入力あり → `KEYWORDS` にカンマ区切りリストで格納
- スキップ → `KEYWORDS=""`（セクション自体を省略）

---

### Step 9: 使用ルール **[Mood]**

> 「ブランドカラーの使い方や哲学を教えてください。
> 60-30-10 ルールはデフォルトで含まれます。
>
> 入力方法は 3 種類から選べます：
>
> (a) 自由記述で直接ルールを書く
>     例: 「Primary 色は『行動のエネルギー』のメタファー。大胆に使って良い」
>
> (b) 過去のスライドから自動分析（推奨）
>     既存のスライド画像から共通の視覚パターンを Claude が抽出します。
>     入力方法はさらに 2 つから選択可：
>       ・ファイルパス指定: `~/Desktop/slide1.png, ~/Desktop/slide2.png`（カンマ区切り、複数可）
>       ・フォルダパス指定: `~/Downloads/past-slides/`（フォルダ内の画像を読む）
>
> (c) スキップ → デフォルトルールのみ採用」

**(a) 自由記述の場合：** デフォルトルールの後に `USE_RULES_CUSTOM` として追記。

**(b) 過去スライド分析の場合：**

1. ユーザがパス（ファイル列 or フォルダ）を入力したら、画像ファイルを列挙：
   - フォルダ指定: `find <path> -maxdepth 1 -type f \( -iname "*.png" -o -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.webp" \) | head -10`
   - ファイル列指定: カンマ区切りでパース、それぞれの存在確認
2. 各画像を Read tool で読み、以下の観点で **共通パターン** を抽出：
   - カラーの使い分け方（Primary をどの面積・どの位置で配置しているか）
   - レイアウト傾向（テキスト/ビジュアル比、余白、配置の固定要素：ロゴ位置・ページ番号位置）
   - 装飾の傾向（線・図形・写真・アイコンの種類、グラデーション帯の有無）
   - 強調手法（巨大数字、バナー、引用枠、テキスト背景の塗りなど）
   - タイポグラフィ（フォントウェイト、サイズの相対的傾向）
   - 背景処理（白ベース／ダーク／グラデーション）
3. 抽出結果を箇条書きの草案として提示し「これでよいですか？／修正箇所を指定してください」と確認
4. ユーザの修正があれば反映し、確定したら `USE_RULES_CUSTOM` として格納

**(c) スキップの場合：** デフォルトルールのみ採用。

デフォルトルール:
```
- Primary色はブランドのメインカラーとして大胆に使用
- 60-30-10 ルール: 60% ニュートラル、30% Primary Light、10% Primary
```

---

### Step 10: 参考デザイン（任意） **[Mood]**

> 「参考にしたいデザインを **固有名詞** で教えてください。
> モデルがそのスタイルを想起しやすくなります。
>
> 例:
> ・Apple Keynote（2015-2020年代の基調講演スライド）
> ・Stripe Sessions のメインビジュアル
> ・Figma Config のスピーカースライド
> ・Linear.app のランディングページ
>
> 複数指定可、カンマ区切り。スキップ可。」

- 入力あり → `REFERENCE_DESIGN` にリスト形式で格納
- スキップ → `REFERENCE_DESIGN=""`（セクション自体を省略）

---

### Step 11: 禁止事項 **[Forbidden]**

> 「禁止したいビジュアル要素はありますか？
> 以下のデフォルト禁止事項に **追加で** 指定したい項目があれば入力してください。
>
> デフォルト禁止事項:
> ・ステレオタイプな IT 系アイコン（歯車・クラウドの汎用イメージ）
> ・ベタ塗り単色の背景（質感のない塗り）
> ・カラーコード文字列の画像内描画
>
> 追加例: "他社ブランドを想起させる色（競合の青/赤）"、"AI が描いたとわかるイラスト"
> スキップ可。」

- 入力あり → デフォルト禁止事項の後に追記
- スキップ → デフォルトのみ採用

---

### Step 12: デフォルトスタイル選択 **[Style]**

> 「次回スライド生成時のデフォルトスタイルを選んでください。
>
> 1) **balanced**（営業資料・提案書風）— デフォルト
>    Pitch.com / MorningBrew / Figma Slides 系。
>    見出し + 3-5 項目の箇条書き、テキスト 40-60% / ビジュアル 40-60% のバランス。
>
> 2) **visual**（ピッチデッキ風）
>    Apple Keynote / TED Talk 系。
>    タイトル 15 文字以内 + キーメッセージ 1 文、ビジュアル 60-80%。
>
> ※ スライド毎に JSON 内 `_style` で個別オーバーライドも可能です。」

- 1 → `ACTIVE_STYLE="balanced"`
- 2 → `ACTIVE_STYLE="visual"`
- スキップ → `ACTIVE_STYLE="balanced"`

決定したら即座に書き出す：

```bash
echo "${ACTIVE_STYLE}" > "${ACTIVE_STYLE_FILE}"
```

**分岐**:
- `FLOW_MODE=existing` の場合 → サマリー確認・プリセット生成をスキップして、直接「完了メッセージ」へ進む
- `FLOW_MODE=new` の場合 → Step 13（任意） → サマリー確認 → プリセット生成 → 完了メッセージ の通常フロー

---

### Step 13: 上級設定の確認（任意・閲覧専用） **[Advanced]**

> 「上級設定（テンプレート Fork）について確認しますか？ (y/N)
> ※ design-setup ではサポートしません。情報表示のみです。」

- `y` → 以下を表示して完了フローへ:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ⚠️  上級：テンプレート Fork（非推奨）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

スライドの「演出の質」を自社特化したい場合、
${V4_SKILL_DIR}/templates/prompt_template_*.j2 を
手動で Fork することで以下を制御できます：

  - mood / reference_design / visual_effects / title_treatment
  - Typography 演出（極大タイトル / 広い字間 等）
  - フルブリード写真処理（デュオトーン 等）

⚠️ 注意事項
  - design-setup のサポート対象外（自己責任）
  - 余白・座標・font size まで触ると、Visual/Balanced の
    スタイル一貫性が崩れる可能性があります
  - 手順は ${V4_SKILL_DIR}/docs/branding.md の「上級」セクション参照

${V4_SKILL_DIR}/templates/ への直接編集が必要です。
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- `N` or 何も入れず → サマリーへ

---

## サマリー確認

**前提**: このセクション以降（プリセット生成まで）は `FLOW_MODE=new`（Step 0 で「新規作成」を選んだ場合）のときのみ実行する。`FLOW_MODE=existing` の場合は Step 12 で既に `.active_preset` と `.active_style` を書き終えているため、サマリー確認とプリセット生成をスキップして「完了メッセージ」へ進む（未定義の `${BRAND_NAME}` / `${SLUG}` / `${PRIMARY}` などを参照しない）。

全ステップ完了後、以下の形式で表示して「この内容でプリセットを生成しますか？」と確認する：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  デザイン設定サマリー
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ブランド名     : {BRAND_NAME}
プリセット名   : {SLUG}.md
デフォルトスタイル: {ACTIVE_STYLE}

[Asset]
ロゴ           : {コピーしたパス or 変更なし}

[Color]
Primary        : {PRIMARY}
Primary Light  : {PRIMARY_LIGHT}  （自動算出）
Primary Dark   : {PRIMARY_DARK}   （自動算出）
Accent Teal    : {ACCENT_TEAL}
Accent Gold    : {ACCENT_GOLD}
Semantic+Gray  : {デフォルト or カスタム}

[Font]
雰囲気         : {HEADING_FONT_MOOD} / {BODY_FONT_MOOD}

[Mood]
トーン         : {TONE}
キーワード     : {KEYWORDS or なし}
使用ルール     : {USE_RULES_SUMMARY}
参考デザイン   : {REFERENCE_DESIGN or なし}

[Forbidden]
禁止事項       : デフォルト{追加 N 項目 or のみ}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

→「この内容で生成しますか？[y/変更箇所を指定]」

- `y` または「はい」 → プリセット生成へ
- 変更箇所を指定 → 該当ステップを再実行してサマリーを再表示

---

## プリセット生成

確認 OK 後に実行。

### 1. プリセットファイル書き出し

`${PRESETS_DIR}/{SLUG}.md` を以下の構造で生成する。
`${V4_SKILL_DIR}/references/presets/example-preset.md` と `${V4_SKILL_DIR}/docs/branding.md` の構造を融合させる。

```markdown
# Design Specifications — {BRAND_NAME}

## 配色パレット

### ブランドカラー（Primary）
- Primary: {PRIMARY} - ブランドのメインカラー。見出し装飾、アイコン、矢印、ボタン、強調に使用
- Primary Light: {PRIMARY_LIGHT} - 薄い背景。ハイライトエリア、注目ボックスの背景
- Primary Dark: {PRIMARY_DARK} - 高コントラストが必要な場面。白テキストの背景色

### アクセントカラー（データ可視化・差別化用）
- Accent Teal: {ACCENT_TEAL} - 図表の第3系列、差別化アクセント
- Accent Gold: {ACCENT_GOLD} - 図表の第4系列、ハイライト、注意表示

### セマンティックカラー（状態表示用）
- Success: {SUCCESS} - 完了、正解、推奨
- Warning: {WARNING} - 注意、要確認
- Error: {ERROR} - エラー、禁止、重要警告
- Info: {INFO} - 参考情報、ヒント、補足

### グレースケール（5段階）
- Gray-900: {GRAY_900} - 見出しテキスト
- Gray-700: {GRAY_700} - 本文テキスト
- Gray-400: {GRAY_400} - キャプション、補足テキスト
- Gray-200: {GRAY_200} - 罫線、セパレーター
- Gray-100: {GRAY_100} - カード背景、コンテナ背景
- ベース: #FFFFFF - スライド背景

### 配色ルール
- 1スライドあたりの使用色は最大3色（グレースケール除く）
- 60-30-10ルール: 60%ニュートラル、30%Primary Lightまたは背景色、10%アクセント
- セマンティックカラーは状態表示の目的でのみ使用し、装飾には使わない
- データ可視化ではPrimary → Accent Teal → Accent Goldの順で系列に割り当て

## フォント

- 見出し: {HEADING_FONT_MOOD}
- 本文: {BODY_FONT_MOOD}

## トーン

{TONE}

## 技術仕様

- アスペクト比: 16:9
- 解像度: 2752x1536px (2K)
- フッター: なし（ロゴは --logo オプションで制御。テキストフッターは入れない）
- 余白: 全辺に5%程度

## レイアウトパターン

{example-preset.md の「## レイアウトパターン」セクション（Pattern A〜M）をそのままコピー。
色指定部分（#EA5514 等）は {PRIMARY} などに置換する}

## 図解パターンガイド

{example-preset.md の「## 図解パターンガイド」セクションをそのままコピー（編集なし）}

{% if KEYWORDS or USE_RULES_CUSTOM or REFERENCE_DESIGN %}
## ブランドムード

{% if KEYWORDS %}
### キーワード
- {KEYWORDS}（カンマ区切りで列挙）
{% endif %}

### 使用ルール
- Primary 色はブランドのメインカラーとして大胆に使用
- 60-30-10 ルール: 60% ニュートラル、30% Primary Light、10% Primary
{USE_RULES_CUSTOM が空でなければここに追記}

{% if REFERENCE_DESIGN %}
### 参考デザイン
- {REFERENCE_DESIGN をリスト形式で列挙}
{% endif %}
{% endif %}

## 禁止事項

- ステレオタイプな IT 系アイコン（歯車・クラウドの汎用イメージ）
- ベタ塗り単色の背景（質感のない塗り）
- カラーコード文字列の画像内描画
{FORBIDDEN_USER が空でなければここに追記}
```

> 注: 上記の `{% if %}` 記法は Jinja ではなく、Claude が条件分岐を判断して該当セクションを出力する/しないを切り替えるためのヒント。
> 実装上はテキスト連結でセクションを組み立てる。

### 2. アクティブプリセット設定

```bash
echo "${SLUG}.md" > "${ACTIVE_PRESET_FILE}"
echo "${ACTIVE_STYLE}" > "${ACTIVE_STYLE_FILE}"
echo "アクティブプリセット: ${SLUG}.md"
echo "アクティブスタイル  : ${ACTIVE_STYLE}"
```

---

## 完了メッセージ

`FLOW_MODE` によって表示内容を変える。

### `FLOW_MODE=new`（新規プリセット作成完了時）

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  セットアップ完了！
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
生成ファイル     : {PRESETS_DIR}/{SLUG}.md
ロゴ             : {コピー済みのパス or 変更なし}
アクティブプリセット: {ACTIVE_PRESET_FILE} に記録済み（{SLUG}.md）
アクティブスタイル  : {ACTIVE_STYLE_FILE} に記録済み（{ACTIVE_STYLE}）

次回「スライドを作って」と言うと、
{BRAND_NAME} のブランド設定が自動で使われます。

設定を変更したい場合は「デザインを設定したい」と言えばいつでも再実行できます。
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### `FLOW_MODE=existing`（既存プリセット選択完了時）

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  プリセット切替完了！
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
アクティブプリセット: {ACTIVE_PRESET_FILE} → {選択したプリセットファイル名}
アクティブスタイル  : {ACTIVE_STYLE_FILE} → {ACTIVE_STYLE}

次回「スライドを作って」と言うと、
このプリセットの設定が自動で使われます。

別のプリセットに切り替えたい / 新しく作りたい場合は
「デザインを設定したい」と言えばいつでも再実行できます。
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 関連ドキュメント

インストール後（`~/.claude/skills/` に design-setup と tekion-slide-generator-v4 がサイドバイサイドで配置された前提）の絶対パス：

- `${V4_SKILL_DIR}/docs/branding.md` — このスキルが自動化している内容の元ガイド（手動でも実施可能）
- `${V4_SKILL_DIR}/references/presets/example-preset.md` — プリセットの参照テンプレート

リポジトリ内ドキュメント（インストール環境では存在しないため、ソース閲覧時のみ参照可能）：

- `project/docs/skill/design-setup-skill.md` — 本スキルの解説ドキュメント
- `project/docs/skill/footer-watermark-spec.md` — 関連: フッターウォーターマーク要件定義（未実装）
