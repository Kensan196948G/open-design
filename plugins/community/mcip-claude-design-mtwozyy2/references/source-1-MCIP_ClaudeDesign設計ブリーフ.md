# MCIP Claude Design 設計ブリーフ

> このファイルは **Claude Design（claude.ai/design）へそのまま貼り付けて使う**ための指示書である。
> 原典は `doc/MCIP_WebUI設計書.md`（1510 行 / 48 章）。本書はそれを設計エージェントが行動できる形へ圧縮したもので、
> 判断に迷う箇所が出たら原典の該当章を参照する（章番号を各節に付けてある）。
>
> 文書版：v1.0　作成日：2026-09-11

---

## 0. このプロダクトは何か（§1）

**MCIP / みらい建設情報基盤** — 建設会社の情報基盤。設計図、構造計算、協議記録、施工写真、BIM/CIM、GIS、
点群などを「案件・構造物を入口に」たどれるようにする Web システム。

UI 設計で最も大切な一文は原典の次の言葉である。

> **「社員にフォルダ構成を覚えさせない」**

従来の `現場案件 → 設計 → 橋梁 → 下部工 → P1 → P1_橋脚_最終_修正2.dwg` という階層移動をやめ、
**`P1橋脚` を開けば一般図・3D モデル・構造計算・数量・設計条件・協議・施工記録・写真・維持管理が並ぶ**
「対象物中心」の UI にする。フォルダツリーを主画面にしてはならない。

---

## 1. 揺らしてはいけない 8 原則（§2）

設計判断が割れたら、この順で優先する。

1. **Search First** — まず検索。保存場所を知らなくても使える。
2. **Object First** — ファイルではなく Project / Facility / Structure / Component を入口にする。
3. **Relationship First** — 図面、計算書、協議、変更理由などの関係をつなぐ。
4. **AI Assisted** — 検索、分類、比較、要約、不足検知を AI が支援する。
5. **Human Controlled** — 正式化、重要変更、外部提出は人が承認する。
6. **Role Aware** — 所属、役割、案件、情報分類に応じて画面を出し分ける。
7. **Spatial First** — 3D / GIS を主要な入口として扱う。
8. **Mobile Friendly** — PC、タブレット、スマートフォンで利用しやすくする。

---

## 2. デザイントークン（§32 / §33 / §35）

原典は色名を「Navy / Blue」のように方針で示しており、具体値は定めていない。
下表の値のうち **Primary / Text / Border / Background は MCIP の既存文書テンプレート
（`tools/doc-template.css`）で実際に使われている色**で、そのまま採ればハウススタイルが揃う。
**Secondary / Success / Warning / Danger は本ブリーフの新規提案**であり、変更してよい。
色相の意味づけ（Primary=Navy 系、Status の対応）は原典どおりに保つこと。

### カラー

白地に対するコントラスト比を実測して併記した（本文テキストは AA = 4.5:1 以上が必要）。

| 役割 | 方針（原典） | 値 | 白地コントラスト | 出所 |
| --- | --- | --- | --- | --- |
| Primary（濃） | Navy / Blue | `#12324a` | 13.28:1 | 既存テンプレート |
| Primary | Navy / Blue | `#154e75` | 8.83:1 | 既存テンプレート |
| Secondary | Cyan | `#0e7490` | 5.36:1 | 新規提案 |
| Success | Green | `#15803d` | 5.02:1 | 新規提案 |
| Warning | Amber | `#b45309` | 5.02:1 | 新規提案 |
| Danger | Red | `#b91c1c` | 6.47:1 | 新規提案 |
| Text | — | `#182d40` | 14.11:1 | 既存テンプレート |
| Text（muted） | — | `#465d70` | 6.86:1 | 既存テンプレート |
| Border | — | `#cddae3` | （非テキスト） | 既存テンプレート |
| Background | Light Gray / White | `#edf2f5` / `#ffffff` | （非テキスト） | 既存テンプレート |

テキストに使う 8 色はすべて AA を満たす。色を差し替える場合も 4.5:1 を下回らせないこと（§34）。

### Status（7 種・原典どおり）

| Status | 色 |
| --- | --- |
| Draft | Gray |
| Shared | Blue |
| Approved | Green |
| Warning | Amber |
| Rejected | Red |
| Archived | Dark Gray |
| AI Running | Cyan |

> **色だけで判断させない。Status は必ずテキストを併記する**（§32 / §34 / §46）。
> バッジは「● Approved」のように色 + ラベルで構成し、アイコンのみ・色のみの表現を作らない。

### Typography（§33）

| 用途 | フォント |
| --- | --- |
| 日本語 | Noto Sans JP → Yu Gothic UI → Meiryo |
| 英数字 | Inter → Segoe UI |
| ID・Code | Monospace |

Information ID（`INF-000123`）、Structure ID（`STR-P1`）、Project ID（`PRJ-2026-001`）、
ChangeSet ID（`CS-000123`）は**必ず等幅**で表示する。

### ブレークポイント（§35）

```text
Desktop >= 1200px
Tablet   768 - 1199px
Mobile   < 768px
```

---

## 3. グローバルレイアウト（§4 / §5 / §6）

```text
┌──────────────────────────────────────────────────────────────┐
│ MCIP Logo │ Global Search │ AI Assistant │ Notification │ User │
├───────────────┬──────────────────────────────────────────────┤
│ Side Menu     │ Main Content                                 │
└───────────────┴──────────────────────────────────────────────┘
```

サイドナビ（権限で出し分ける）:

```text
01 Dashboard        02 My Projects      03 Projects
04 3D / GIS         05 Information Catalog
06 AI Assistant     07 Approval         08 Submission
09 Data Quality     10 Agent / Skills   11 Administration
```

- **Global Search** は Project / Facility / Structure / Component / Information / Document / BIM / GIS / 写真 / 点群 / 協議 / ChangeSet を横断する。入力例：`P1橋脚` `○○港` `2026年度 出来形` `最新承認図` `INF-00123` `STR-P1`
- **AI Assistant** はどの画面からでも呼べる。現在の Project / Structure / ユーザー役割を暗黙の文脈として渡す（§41）ので、「最新図面を見せて」だけで対象が決まる
- **Command Palette** は `Ctrl / Cmd + K`（§42）
- **Notification** の対象：承認依頼 / レビュー依頼 / AI 処理完了 / データ不足 / Version 更新 / 外部提出期限 / MCP・API エラー / Security Alert

---

## 4. 画面インベントリ（§44）

**Phase 1（まずこの 14 画面）**

```text
01 Login              02 Dashboard          03 Project List
04 Project Home       05 Structure Detail   06 Information Catalog
07 Information Detail 08 Upload             09 Search
10 Relationship       11 Approval           12 AI Assistant
13 Admin User/Role    14 Audit
```

Phase 2: Data Quality / Version Compare / ChangeSet / Submission Center / API Integration / CDE Federation
Phase 3: 3D Viewer / GIS Viewer ほか

**デザインの着手順の推奨**：02 Dashboard → 04 Project Home → 05 Structure Detail → 07 Information Detail → 11 Approval。
この 5 画面が MCIP の体験の中心で、残りはここで決めた語彙の再利用になる。

---

## 5. 主要画面の仕様

### 5.1 Dashboard（§7）

ログイン直後に「**今日、自分が何をすればよいか**」が分かること。

```text
おはようございます、○○さん
─────────────────────────────────
My Projects 5 │ Approval 3 │ Alert 2 │ AI Tasks 4
─────────────────────────────────
最近見た情報
承認待ち
AI からのおすすめ
  「P1橋脚に未紐付けの図面があります」
```

Widget 候補：My Projects / Approval Queue / Data Quality Alerts / Recent Information /
Recent AI Activity / Submission Deadline / Project KPI。

利用者ごとに優先情報が違う（§3）。役割でウィジェットの並びを変える設計にする。

| 利用者 | 優先するもの |
| --- | --- |
| 経営企画 | Portfolio / KPI / AI Value |
| DX 推進・Platform 管理 | Admin / Governance / Monitoring |
| 技術本部 | Structure / Design / Review |
| 施工本部・作業所 | My Project / 3D / Quick Upload |
| 安全品質環境 | Review Queue / Evidence |
| 営業・発注者対応 | Client Requirements / Submission |
| 維持管理 | Asset Map / Inspection |

### 5.2 Project Home（§9）— MCIP の中心画面

ヘッダに `PRJ-2026-001 / ○○港湾整備工事`、Status・発注者・支店・作業所・Data Owner。

Tab: `Overview | Structures | 3D/GIS | Information | Design | Construction | Quality | Meetings | Changes | Submission | AI`

### 5.3 Structure Detail（§10）

```text
STR-P1 | P1橋脚        Status / Owner / Latest Update
─────────────────────────────────────────────
[3D Preview]
─────────────────────────────────────────────
Overview | Drawing | BIM | Calculation | Quantity
Condition | Meeting | Construction | Inspection
─────────────────────────────────────────────
Related Information
```

表示：Structure ID / 名称 / 工種 / 位置 / Project / Facility / 施工 Status / 設計 Status /
Data Quality / 最新 Version / 担当者。

### 5.4 Information Detail（§11 / §12 / §13）

```text
INF-000123   P1橋脚一般図
Type Drawing │ Version 3 │ Status Approved │ Format DWG
Owner 技術本部 │ Created 2026-08-20 │ Approved 2026-08-25
```

Action: Preview / Download / Open Original / Compare Version / Related Information /
Ask AI / Request Review / Create ChangeSet / Add Relationship

**Version UI** は Timeline 形式（v1 Draft → v2 Shared → v3 Approved）。
差分表示の対象は Metadata / File / Geometry / Attribute / Approval / Relationship。

**Relationship UI** は Graph View と List View の両方を持つ。関係ラベル（`hasDrawing` `calculatedBy`
`basedOn` `changedBy`）を辺に明示する。

### 5.5 Approval Center（§22）

一覧の列：`Request / Type / Project / Requested By / Reviewer / Status / Deadline`

Approval Type: Information Approval / ChangeSet / AI Write / Submission / Skill Publication / MCP Permission

> 承認は MCIP の中核統制である。誰が要求し誰が承認したかが常に見え、**同一人物が要求と承認を兼ねられない**
> ことが UI 上も分かる形にする（自己承認は DB 側でも拒否される）。

### 5.6 ChangeSet（§23）

```text
CS-000123
Object   STR-P1
Change   PierHeight
Before   8,000mm
After    8,500mm
Reason   ○○協議
Affected Drawing 3 / Calculation 2 / Quantity 1 / BIM 1

AI 分析
  Impact Level: Medium
  再計算推奨 / 数量再算出必要 / 一般図更新必要
```

Before / After は必ず並置し、影響範囲を件数で示す。AI の影響分析は**提案**であり、適用は承認を経る。

### 5.7 Data Quality（§21）

```text
Data Quality Score
  Missing Metadata     12
  Broken Relationship   3
  Unknown Version       4
  Duplicate Information 1
  Coordinate Error      2
  Approval Missing      5
```

AI 提案には必ず `[適用] [確認] [無視]` の 3 択を付ける。AI が黙って直す UI にしない。

### 5.8 Upload / データ登録（§20 / §27）

Drag & Drop またはファイル選択 → **AI Classification を確信度付きで提示** →
ユーザーが承認して登録。

```text
Project           ○○港工事   98%
Structure         P1橋脚      93%
Information Type  Drawing     96%
Version           v3          82%

[承認して登録]  [修正]
```

確信度は数値で出し、低いものが目に付くようにする。モバイルの写真登録も同じ構造（§27）。

### 5.9 AI Assistant（§15 / §40）

Chat 型。**回答には必ず根拠を添える。**

```text
User: P1橋脚の設計変更理由を教えて

AI:  変更理由は○○です。
     根拠:
       1. INF-006 河川協議記録
       2. CS-00012 ChangeSet
       3. INF-005 設計条件
     [原典を開く] [3Dで表示] [関連情報]
```

必ず表示する項目：**Answer / Sources / Information ID / Version / Access Scope**。
重要業務では `AI Generated` と `Human Review Required` を明記する。

### 5.10 エラー UI（§30）

`Error 403` のような表示を作らない。理由と次の行動を出す。

```text
この情報を表示する権限がありません。

理由：この情報は「原価限定」に分類されています。
必要な場合：Project Manager へアクセス申請できます。

[アクセス申請]
```

### 5.11 Empty State（§31）

`No Data` を作らない。

```text
P1橋脚にはまだ「施工記録」が登録されていません。

[施工記録を登録]  [AIに既存データを探させる]
```

### 5.12 モバイル（§26）

下部ナビ: `Home / Project / Camera / AI / More`
Quick Action: 写真登録 / QR・ID 読取 / Structure 選択 / 音声メモ / AI 質問 / 最新図面表示

---

## 6. AI・Engine・人間の役割分担

MCIP の設計上の中核であり、UI にも現れなければならない。

| 担当 | 役割 | UI での現れ方 |
| --- | --- | --- |
| **AI** | 条件整理・検索・説明・実行調整 | 提案・要約・分類・不足検知。必ず根拠付き。確定操作は持たない |
| **Engine** | 数値計算（検証済みのものだけ） | 計算結果には Engine 名と版、入力 Snapshot を併記 |
| **人間** | 最終判断 | 承認・却下。正式化・重要変更・外部提出は必ず人が押す |

AI に「削除」「上書き」「承認」のボタンを直接与えない（§46）。
AI ができるのは **ChangeSet の作成まで**で、その先は人の承認を通る。

---

## 7. 必ずやること / やってはいけないこと（§46 / §47）

### 必ず表示する

- Project / Structure / Information の **ID**
- **Status**（色 + テキスト）
- **Version**
- **Owner**
- **Source（原典）**
- **Related Information**
- AI 回答の**根拠**

### やってはいけない

- フォルダツリーをメイン UI にする
- ファイル名だけで正式版を判断させる
- AI 回答だけ表示して原典を隠す
- 3D 画面だけで全情報を完結させる
- Admin 画面を一般ユーザーに見せる
- AI Agent に直接ストレージ操作ボタンを与える
- 承認済みデータを無警告で上書きする
- 重要情報を**色だけ**で識別させる

---

## 8. アクセシビリティ（§34）

WCAG を意識し、次を満たす。

- Keyboard 操作 / Focus 表示
- **色だけに依存しない**（Status は Icon + Text）
- Contrast 確保
- Screen Reader 対応
- Form Error の説明（何が悪く、どうすれば直るか）
- Responsive

---

## 9. 代表ユーザーフロー（§45）

**最新図面にたどり着く — 目標 5 クリック以内**

```text
Login → Global Search →「P1橋脚」→ Structure → Drawing → Approved → Latest → Preview
```

**AI 検索**

```text
P1橋脚画面 → AI Assistant →「高さ変更理由は？」→ ChangeSet → 協議記録 → 設計条件 → 回答
```

**データ登録**

```text
Upload → AI Classification → User Confirm → Validation → Register → Relationship Proposal → Complete
```

**AI 起点の変更**

```text
User → AI → Change Proposal → Impact Analysis → ChangeSet → Reviewer → Approver → Update
```

**成果品提出**

```text
Project → Submission → MLIT Rule → Collect → Adapter → Validation → Preview → Approval → Export
```

---

## 10. UI と Information Core の対応（§38）

デザイン上の名前は、この対応を崩さないこと。

| UI | Core |
| --- | --- |
| Project 画面 | Project |
| 3D Object | Structure / Component |
| Information Detail | Information |
| Version Timeline | Version |
| Related Info | Relationship |
| Change 画面 | ChangeSet |
| Access | Authorization |
| Source | Provenance |

---

## 11. UX KPI（§43）

デザインの良し悪しはこれで測る。

情報検索時間 / 最新情報到達時間 / 誤版利用件数 / データ重複件数 / 未紐付け件数 /
Metadata 不足件数 / 承認リードタイム / 成果品作成時間 / AI 回答から原典到達率

---

## 更新履歴

| 日付 | ファイル名 | バージョン | 変更内容 |
| --- | --- | --- | --- |
| 2026-09-11 | MCIP_ClaudeDesign設計ブリーフ.md | v1.0 | `doc/MCIP_WebUI設計書.md` v1.0（48 章）から Claude Design 向けに圧縮して作成 |
