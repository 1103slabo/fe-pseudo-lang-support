# Changelog

## [1.9.4] - 2026-06-24

### Fixed

- F5 デバッグ開始時にブレークポイントが無視されて最後まで実行されてしまう問題を修正
  - 原因：DAP 起動シーケンスで `launch` が `configurationDone` より後に処理されるケースがあり、`configurationDoneRequest` 時点で `this.sourceFile` が空文字のまま `continue` が実行されていた
  - `configurationDoneRequest` から `continue` 呼び出しを削除
  - `launchRequest` 末尾（`this.sourceFile` 確定後）でブレークポイントを参照して実行開始するよう変更
    - ブレークポイントあり → `runtime.continue(bps)` でブレークポイントまで実行
    - ブレークポイントなし → `runtime.step()` で先頭行に停止

- F5 起動直後の `continue()` 呼び出しで先頭ブレークポイントがスキップされる問題を修正
  - 原因：`runtime.continue()` の `isFirst = true`（現在行での再停止防止フラグ）が F5 起動時にも適用されており、先頭ブレークポイントの行が即座にスキップされていた
  - `runtime.continue()` に `isInitialLaunch` パラメータを追加
    - F5 起動時（初回実行）は `isInitialLaunch: true` を渡し `isFirst = false`（スキップなし）
    - Continue ボタン押下時はデフォルト（`isInitialLaunch: false`）のまま `isFirst = true` で現在行の再停止を防止

## [1.9.3] - 2026-06-24

### Changed

- `pseudoDebugSession.ts`：`continueRequest()` に追加されていた一時デバッグログ（`[BP DEBUG]`）を削除

## [1.9.2] - 2026-06-24

### Fixed
- 戻り値あり関数呼び出しを含む変数宣言行（`整数型: a ← func()`）でF11ステップインが機能しない問題を修正

## [1.9.1] - 2026-06-23

### Added
- `が 未定義` / `が 未定義でない` 条件式を実装
  - 変数が未初期化（null）かどうかを判定する条件式
  - `tokenizer.ts`：`未定義でない` / `未定義`（`が` を含まない形）を複合キーワードとして登録（長い方を先に）。`が` は既存の `GA` トークンとして別途トークン化されるため、トークン列は `GA` + `IS_UNDEFINED`/`IS_NOT_UNDEFINED` となる
  - `ast.ts`：`UndefinedCheckExpressionNode` を追加
  - `parser.ts`：`parseComparisonExpression()` の GA 分岐に未定義チェック処理を追加
  - `evaluator.ts`：`UndefinedCheckExpression` の評価を追加
  - `flowchartGenerator.ts`：`exprToString()` に対応を追加
  - 連結リスト系疑似言語コード（`while (current が 未定義でない)`）が実行可能になった

## [1.9.0] - 2026-06-23

### Added

- フローチャートパネルにズームイン／アウト機能を追加
  - ツールバー左側に「−」「スライダー（25〜200%）」「パーセント表示」「＋」「リセット（⟳）」を追加
  - CSS `transform: scale()` でズーム処理（Webview側のみで完結、`flowchartPanel.ts` 変更なし）
  - キーボードショートカット：Ctrl+= でズームイン、Ctrl+- でズームアウト、Ctrl+0 でリセット
  - PNG保存時はズームをリセットして原寸×2倍で出力（従来通り）

## [1.8.0] - 2026-06-22

### Changed

- 大域変数スコープの厳格化とメイン処理の `〇main()` 必須化（言語仕様変更）
  - `大域:` なしのトップレベル変数・配列宣言をエラーにする
    - エラー例：「トップレベルの変数宣言には "大域:" が必要です（"x"）」
  - `大域:` 付き変数・配列をグローバルスコープに正しく登録し、関数内からの参照・代入を有効化
  - `〇main()` がなくトップレベルに処理文がある場合をエラーにする
    - エラー例：「メイン処理は "〇main()" の中に記述してください」
  - `〇main()` なし・大域変数宣言＋関数定義のみのファイルは正常終了を維持

### Fixed

- 大域変数の初期値（`大域: 整数型: counter ← 0` の `← 0` 部分）が `main()` 呼び出し前に実行されず `null` のまま参照されてエラーになるバグを修正
  - `run()` の先頭でトップレベルの `Assignment` 文を事前実行するよう変更

## [1.7.5] - 2026-06-22

### Fixed

- フローチャート（分岐表現モード）のループ戻り矢印の終点を修正
  - while・for 文：loopBack エッジの終点をひし形左頂点（`cx - DIAMOND_W, diamondMidY`）からひし形上頂点の 16px 上（`cx, y - 16`）に変更
  - do-while 文：同様に終点を `cx, y - 16` に変更し、ひし形とボディをつなぐ縦線の上に矢印が重なるよう修正
  - 不要になった `loopBackEndX` / `loopBackEndY` 変数および `getFirstVisualNode()` メソッドを削除

## [1.7.4] - 2026-06-22

### Changed

- フローチャートのループ端（六角形）にループ連番名を表示するよう変更
  - ループごとに「ループ1」「ループ2」と連番で名前を付与
  - for / while ループ：上端に「ループN＋条件」（2行）、下端に「ループN」を表示
  - do-while ループ：上端に「ループN」、下端に「ループN＋条件」（2行）を表示

### Fixed

- フローチャートのループ端左横の後戻り矢印を削除

## [1.7.3] - 2026-06-21

### Changed

- フローチャートのfor文ループ端ラベルをIPA公式表記に変更
  - 変更前：`i: 1～8`（増分が表示されない独自表記）
  - 変更後：`i: 1,1,8`（変数名:初期値,増分,終値）
  - 減少方向（`ずつ減らす`）の場合は増分に`-`を付与（例：`i: 10,-2,1`）

- PNG保存・CSV出力のデフォルトファイル名を現在開いている `.pseudo` ファイル名に統一
  - フローチャートパネルの「PNG保存」ダイアログの初期ファイル名が `flowchart.png`（固定）から `<ファイル名>.png` に変わる
  - トレース表の「CSV出力」ダイアログの初期ファイル名が `trace.csv`（固定）から `<ファイル名>.csv` に変わる
  - 保存ダイアログの初期フォルダも `.pseudo` ファイルと同じフォルダを指定（毎回フォルダを辿る手間を削減）
  - 未保存ファイル（`Untitled-1.pseudo` など）の場合はフォルダ指定なし・ファイル名のみ反映
  - 変更ファイル：
    - `src/views/flowchartPanel.ts`：`_document` フィールドを追加し `_update()` で保持、`_getDefaultSaveUri()` ヘルパーを追加、`_savePng()` の `defaultUri` を差し替え
    - `src/views/traceViewProvider.ts`：`_currentFilePath` フィールド・`setCurrentFile()` メソッドを追加、`_getDefaultCsvUri()` ヘルパーを追加、`_saveCsv()` の `defaultUri` を差し替え
    - `src/debug/pseudoDebugSession.ts`：`launchRequest()` でデバッグ開始時に `traceView?.setCurrentFile(filePath)` を呼び出すように変更

## [1.7.2] - 2026-06-21

### Added

- フローチャートパネルにPNG保存ボタンを追加
  - パネル右上の「PNG保存」ボタンをクリックすると、フローチャートをPNG画像としてローカルに保存できる
  - 保存先はダイアログで選択可能
  - 新規ファイル：`src/views/flowchartPanelView.js`（WebviewクライアントスクリプトでSVGをCanvasに変換し、base64でホストへ送信）

## [1.7.1] - 2026-06-20

### Changed

- README更新

## [1.7.0] - 2026-06-18

### Changed

- フローチャート可視化をMermaid（CDN）から自前SVGレンダラーに完全置き換え
  - JIS X 0121準拠の図形記号に対応
    - 端子記号（角丸矩形）：開始・終了（開始ノードには関数名、関数の終了ノードには「呼び出し元へ戻る」を表示）
    - 処理記号（矩形）：代入・変数宣言
    - データ記号（平行四辺形）：出力文
    - 判断記号（ひし形）：if条件分岐（Yes/No分岐後は折れ線で合流し、1本の線で下に続く）
    - ループ端記号（角を斜めに切り落とした六角形のペア）：while・do-while・for
    - 定義済み処理記号（二重線矩形）：関数呼び出し
  - 関数定義を、メインのフローチャートと横に並ぶ独立したフローチャート（開始→body→終了）として表示
  - 矢印表記をJIS X 0121準拠に修正
    - 通常の流れ（上→下、左→右）は矢印なしの線
    - 逆方向（下→上）の戻り線のみ矢印を表示
  - CDN依存を完全排除（オフライン環境で動作可能）
  - CSPから外部スクリプト許可を削除
  - 新規ファイル：`src/flowchart/svgRenderer.ts`（FlowIR → SVG変換クラス）
  - 削除ファイル：`src/flowchart/mermaidRenderer.ts`

### Changed (FlowIR)

- `flowIR.ts`：`output`ノードを`io`ノード（平行四辺形）に変更
- `flowIR.ts`：`loop`ノードに`loopType: 'while' | 'doWhile' | 'for'`を追加
- `flowIR.ts`：`start`・`end`ノードに`label`フィールドを追加（関数名・「呼び出し元へ戻る」表示用）
- `flowIR.ts`：`call`ノード（関数呼び出し用、定義済み処理記号）を追加
- `flowIR.ts`：複数フローチャートを束ねる`FlowchartUnit`・`FlowchartDocument`型を追加
- `flowchartGenerator.ts`：`PrintStatement` → `io`ノードに変更
- `flowchartGenerator.ts`：各ループに`loopType`を付与
- `flowchartGenerator.ts`：関数定義を抽出し、メインとは別の`FlowchartUnit`として生成するよう変更
- `flowchartGenerator.ts`：`FunctionCall` → `call`ノードに変更

## [1.6.0] - 2026-06-17

### Added

- フローチャート可視化機能を追加
  - `.pseudo` ファイルの制御フローをMermaid形式のフローチャートとして可視化
  - エディタ右上のアイコンボタン（`$(type-hierarchy)`）をクリックするとエディタ右側にプレビューパネルが開く（Markdown Previewと同様の左右分割表示）
  - 対応構文：代入・変数宣言・配列宣言・if文・if-else文・while文・do-while文・for文・関数定義・関数呼び出し・出力文・return文・swap文・append文
  - 同じパネルを再利用する設計（再実行時に新規パネルは開かない）
  - Mermaidレンダリングはcdn.jsdelivr.net経由
  - パースエラー時はエラーメッセージをパネル内に表示
  - 新規ファイル追加：
    - `src/flowchart/flowIR.ts`：AST→Mermaid変換の中間表現（IR）型定義
    - `src/flowchart/flowchartGenerator.ts`：AST → IR変換クラス
    - `src/flowchart/mermaidRenderer.ts`：IR → Mermaid文字列変換クラス
    - `src/views/flowchartPanel.ts`：WebviewPanel管理クラス
    - `src/views/flowchartPanel.html`：Webview HTMLテンプレート
    - `src/views/flowchartPanel.css`：Webview CSS
  - 変更ファイル：
    - `src/extension.ts`：`fe-pseudo-lang.showFlowchart` コマンドを登録
    - `package.json`：コマンド・`editor/title` メニューを追加

## [1.5.0] - 2026-06-17

### Added

- for文のデクリメント構文（`ずつ減らす`）に対応
  - `for (i を 10 から 1 まで 1 ずつ減らす)` のような降順ループが実行可能に
  - `tokenizer.ts`：`ずつ減らす` を `STEP_DEC` トークンとして追加（既存の `STEP` は `STEP_INC` にリネーム）
  - `ast.ts`：`ForStatementNode` に `direction: 'inc' | 'dec'` フィールドを追加
  - `parser.ts`：`parseForStatement()` で `STEP_INC` / `STEP_DEC` の両方を受け付けるよう変更
  - `evaluator.ts`：`genForStatementBody()` のループ継続条件・ステップ加減算を `direction` に応じて切り替え

## [1.4.0] - 2026-06-17

### Added
- トレース表パネルを追加
- トレース表にCSV出力ボタンを追加
  - トレース表パネルの監視式バーの右端に「CSV出力」ボタンを配置
  - テーブルの全行（ヘッダ含む）をCSVファイルとして保存可能
  - UTF-8 BOM付き（Excelでの文字化け防止）
  - 配列値など、カンマを含むセルは自動的にダブルクォートでエスケープ
  - VSCodeの保存ダイアログで保存先を指定（デフォルトファイル名：`trace.csv`）
  - テーブルが空の間はボタンを非活性（`disabled`）表示
  - 変更ファイル：
    - `traceView.html`：CSV出力ボタンを追加
    - `traceView.js`：`exportCsv()` / `escapeCsvCell()` 関数を追加、ボタンの活性/非活性制御
    - `traceView.css`：CSV出力ボタンのスタイル追加
    - `traceViewProvider.ts`：`exportCsv` メッセージのハンドリング、`_saveCsv()` メソッド追加

### Fixed

- トレース表の for文行でループ変数が `-`（未定義）と表示されるバグを修正
  - for文の初回停止時に、ループ変数の初期値がセットされる前にトレース行が記録されていた
  - `genStatement()` で ForStatement を早期処理し、初期値セット後に yield するよう変更
  - `genForStatement()` を `genForStatementBody()` に分割・整理し `isFirst` フラグを除去
  - 変更ファイル：`evaluator.ts`

## [1.3.0] - 2026-06-16

### Added

- 配列インデックスの 0 始まり / 1 始まり切り替え機能を追加
  - IPA 問題文の「配列の要素番号は 0 から始まる」記述に対応
  - `.pseudo` ファイル内に `要素番号は 0 から始まる`（全角数字「０」も可）と記述するだけで自動判別
  - ディレクティブがない場合は従来どおり 1 始まり（既存動作に変更なし）
  - 検出パターン：`/要素番号は\s*[0０]\s*から始まる/`
  - 変更ファイル：
    - `evaluator.ts`：`arrayBase` プロパティ・`setArrayBase()` / `getArrayBase()` を追加。`evaluateArrayAccess()` / `executeArrayAssignment()` / `assignToExpression()` の全インデックス計算を `this.arrayBase` 基準に変更。範囲外エラーメッセージに有効範囲（例：`有効範囲: 0 ～ 2`）を追加。`detectArrayBase()` をモジュール関数としてエクスポート
    - `runtime.ts`：`detectArrayBase` を import し、`load()` で `setArrayBase()` を呼ぶように変更。`getArrayBase()` を公開
    - `runCurrentFile.ts`：`detectArrayBase` を import し、`Evaluator` 生成直後に `setArrayBase()` を呼ぶように変更
    - `pseudoDebugSession.ts`：変数ウィンドウの配列インデックスラベル（1次元・2次元）を `arrayBase` 基準に変更。`buildMemoryVars()` で `arrayBase` を `MemoryVariable` に付与
    - `memoryViewProvider.ts`：`MemoryVariable` に `arrayBase?: 0 | 1` フィールドを追加
    - `memoryView.js`：実体セルのインデックスラベル生成を `v.arrayBase ?? 1` 基準に変更
- スニペットにディレクティブ入力ショートカットを追加
  - `/* ここで，配列の要素番号は 0 から始まる */`（prefix: `array0` / `0はじまり` / `zerohajimari`）
  - `/* ここで，配列の要素番号は 1 から始まる */`（prefix: `array1` / `1はじまり`）

## [1.2.4] - 2026-06-16

### Fixed

- 文字列リテラルから始まる連結出力構文の対応

## [1.2.3] - 2026-06-16

### Added

- `÷ の商` 演算子を追加（整数除算・切り捨て）
  - IPA公式問題（R5 問3・R6 問5）の分析にもとづき、`÷` 単体を常に実数除算に変更したうえで、整数除算専用の `÷ の商` 構文を新設
  - 例：`7 ÷ 2 の商` → `3`（切り捨て）、`(first ＋ last) ÷ 2 の商` が正しく動作

- `÷ の余り` 演算子を追加（剰余演算・`mod` の別記法）
  - IPA疑似言語の自然言語表記 `A ÷ B の余り` に対応（サンプル問題 問16：`cp ÷ 256 の余り`）
  - `mod` キーワードと同等の演算を行う。`mod` は引き続き使用可能

### Changed

- `÷` の演算結果を常に実数除算に変更（**破壊的変更**）
  - 変更前：両辺が整数型の場合は `Math.trunc()` で切り捨て
  - 変更後：型に関係なく常に実数除算（例：`7 ÷ 2` → `3.5`）
  - 根拠：R6 問5 で整数型同士の `÷` に `/* 実数として計算する */` コメントがあることを確認

### Fixed

- 全角演算子（`＋` `－` `＞` `＜` `＝`）を含む式が「未対応の演算子」エラーになる問題を修正
  - v1.2.2 で全角演算子を追加した際、`addToken()` の value に全角文字のままを渡していたため、`evaluator.ts` の switch 分岐が半角ケースに到達できなかった
  - tokenizer 側で全角→半角に正規化（`＋` → `+` 等）。type・シンタックスハイライトへの影響なし

- シンタックスハイライト：変数名・関数名直後に空白を省略した場合でも色付けされない問題を修正
  - `xの値を出力する`（空白なし）でキーワードが色付けされなかった問題を修正
    - `identifiers` の match パターンからひらがなを除外し（`[\p{L}\p{N}_]+` → ASCII英数字・CJK漢字・カタカナのみ）、ひらがなキーワード部分が `keywords` パターンに正しく到達するように修正
  - `〇add(...)`（空白なし）で関数名が色付けされなかった問題を修正
    - `function-definition` の後読みアサーションを `(?<=〇\s)` → `(?<=〇\s?)` に変更し、空白省略に対応

## [1.2.2] - 2026-06-16

### Added

- 全角演算子記号（`＋` `－` `＞` `＜` `＝`）をtokenizerに追加
  - IPA公式PDFのコードをそのままコピペして実行できるよう、半角記号（`+` `-` `>` `<` `=`）に加えて全角バリエーションも受け付けるように対応
  - 既存の半角記号・全角記号（`×` `÷` `≧` `≦` `≠`）の動作に変更なし
  - 全角 `－`（U+FF0D）は単項マイナス（負の数リテラル）としても正しく動作
- スニペットに `＋`（たす / tasu）・`－`（ひく / hiku）を追加

## [1.2.1] - 2026-06-15

### Fixed

- フォーマッタ（`formatter.ts`）のインデント崩れを修正
  - `if`〜`endif` の後に通常の `while (...)` ループが続く場合、`while` 行と `endwhile` 以降の行（`return` 文など）がインデント0（行頭）になる不具合を修正
    - do-whileの終端 `while(...)` 行のみインデントを下げるよう、ブロック種別をスタックで管理する方式に変更
  - ブロックコメント `/* ... */` が、ファイル先頭以外（関数の `return` 文の直後など）に置かれた場合に、前の関数のインデントを引き継いで行頭にインデントが入ってしまう不具合を修正
    - ブロックコメント（開始行・継続行とも）を常にインデント0（行頭）で出力するように変更

## [1.2.0] - 2026-06-15

### Added

- ブロックコメント `/* ... */` に対応
  - `tokenizer.ts`：行コメント（`//`）判定の前にブロックコメント判定を追加。複数行に跨る場合も `line`/`column` を正しく更新し、閉じタグ `*/` がない場合は構文エラーとする
  - `formatter.ts`：複数行ブロックコメント中であることを示す `inBlockComment` フラグを追加し、ブロックコメント中・開始行のインデントを正しく維持
  - `language-configuration.json`：`comments.blockComment` および `autoClosingPairs` に `/* `→`*/` を追加し、エディタのコメント切替・自動補完に対応

## [1.1.3] - 2026-06-15

### Fixed

- 「新しいFE疑似言語ファイル」のテンプレートの〇を正しい記号に修正

## [1.1.2] - 2026-06-14

### Changed

- `.vscodeignore` に `.claude/**` を追加し、Claude Code のローカル設定ファイルが VSIX に同梱されないように修正
- 疑似メモリパネルのビューコンテナID `ipa-pseudo-memory-panel` を `fe-pseudo-memory-panel` に変更
  - `package.json` の `viewsContainers.panel[].id` と `views` のキーを更新
- README.md のライセンス節のリンクを修正
  - `[LICENSE]` → `[LICENSE](./LICENSE)`

## [1.1.1] - 2026-06-14

### Changed

- `package.json` に `categories`（Education / Debuggers / Programming Languages）と `keywords` を追加し、Marketplaceでの検索性を向上
- README.md に新規ファイル作成機能・疑似メモリクリア機能の説明を追加
- `scripts/release-support.js` を修正し、LICENSEファイルも `fe-pseudo-lang-support` にコピー・commitするように変更
  - これにより、support リポジトリ上のREADMEからも `LICENSE` ファイルへの参照が機能するようになる

## [1.1.0] - 2026-06-14

### Added

- 「新しいFE疑似言語ファイル」コマンドを追加
  - コマンドパレットから新規 `.pseudo` ファイルを作成可能
  - エクスプローラーでフォルダを右クリックして新規 `.pseudo` ファイルを作成可能
  - エクスプローラー上部に新規 `.pseudo` ファイル作成アイコンを追加
  - ファイル名を自動採番し、`Untitled-1.pseudo`、`Untitled-2.pseudo`、… の形式で生成
  - 既存ファイルと重複しない名前を自動的に選択
  - 作成したファイルを自動でエディタに表示
  - 新規ファイル生成時に初期テンプレートを挿入

- 独自ライセンス「FE疑似言語 教育利用ライセンス 1.0」を追加
  - `LICENSE` ファイルを追加
  - 個人利用および教育目的での利用条件を明文化
  - 無償での複製および再配布を許可
  - 商用利用、改変、改変版の配布、他ソフトウェアへの組み込みを禁止する利用条件を追加

### Changed

- エクスプローラー上部のボタン追加方式を `explorer/title` から `view/title` に変更
  - `workbench.explorer.fileView` を対象とすることで、Explorer 上部へのアイコン表示を安定化

- ライセンス管理方式を見直し
  - `package.json` のライセンス設定を更新
  - README のライセンス説明を追加・整理

## [1.0.17] - 2026-06-14

### Fixed

- `if`〜`endif` を含む関数で `endif` 以降のインデントが1段浅くなるバグを修正
  - `format()` メソッド内で `endif` に対するインデント減算が2箇所（行頭判定・行末判定）に存在し、合計2回 `indentLevel` が減算されていた
  - 行末側の重複ブロック（`if (raw === 'endif') { indentLevel-- }`）を削除し、行頭側の `startsWithAny` による減算のみに統一

## [1.0.16] - 2026-06-13

### Added

- 疑似メモリビューのタイトルバーに「クリア」アイコンボタン（`$(clear-all)`）を追加
  - デバッグコンソールの「コンソールのクリア」と同様のUI・操作感
  - 押すとデバッグセッション開始前の初期表示状態（「デバッグを開始するとメモリが表示されます」）に戻る
  - デバッグ中・非デバッグ中を問わず常時表示
  - `package.json`：`fe-pseudo-lang.clearMemoryView` コマンドを追加、`view/title` メニューに登録
  - `extension.ts`：`clearMemoryCommand` を登録し、既存の `MemoryViewProvider.clear()` を呼び出す

## [1.0.15] - 2026-06-13

### Changed
- README.md を修正

## [1.0.14] - 2026-06-13

### Changed

- ユーザー向けサポート用リポジトリ `fe-pseudo-lang-support`(public)を新設
  - 本体のソースコードリポジトリ(`fe-pseudo-lang`)はprivateのまま維持
  - README.md / CHANGELOG.md のコピーをsupportリポジトリに配置
  - 学生からの質問・不具合報告はsupportリポジトリのIssueで受け付ける
- `package.json` の `repository` / `bugs` / `homepage` を新設したsupportリポジトリのURLに変更
  - Marketplaceページの「Repository」「Issues」リンクの参照先をsupportリポジトリに変更

## [1.0.13] - 2026-06-13

### Fixed

- ステップ実行時、ループ・分岐の宣言部分・条件判定部分で止まらない問題を修正
  - `for`：2周目以降のループ先頭（`for`行）で再停止するように `genForStatement()` を修正
  - `while`：2回目以降の条件判定前（`while`行）で再停止するように `genWhileStatement()` を修正
  - `do-while`：`while`条件判定行で新たに停止するよう追加
    - `ast.ts`：`DoWhileStatementNode` に `whileLine`/`whileColumn` を追加
    - `parser.ts`：`parseDoWhileStatement()` で `while` トークンの行情報を保持
    - `evaluator.ts`：`genDoWhileStatement()` を2段階停止（do本体開始時／while条件判定時）に変更
  - `if`：`elseif`/`else`節の行で、節に入る直前に停止するよう追加
    - `ast.ts`：`IfStatementNode` の `elseifClauses` に `line`/`column`、`elseBody` 用に `elseLine`/`elseColumn` を追加
    - `parser.ts`：`parseIfStatement()` で `elseif`/`else` トークンの行情報を保持
    - `evaluator.ts`：`genIfStatement()` を修正。従来 `clause.condition` の評価に誤って `stmt.line`/`stmt.column`（if文自身の行）を使っていた箇所も `clause.line`/`clause.column` に修正
  - ステップ実行時の推奨停止仕様：

    | 構文     | 停止タイミング                |
    | -------- | ----------------------------- |
    | if       | 条件判定前                    |
    | elseif   | 条件判定前                    |
    | else     | else節に入る直前              |
    | for      | 各ループ開始時                |
    | while    | 各条件判定時                  |
    | do-while | do本体開始時とwhile条件判定時 |

## [1.0.12] - 2026-06-12

### Changed
 
- README.md の比較演算子の部分を修正


## [1.0.11] - 2026-06-12

### Added

- ウォッチ式で任意の式を評価できるように機能拡張
  - 変数名のみ対応だった `evaluateRequest()` を、トークナイザー → パーサー → エバリュエーターの既存パイプラインに通す方式に変更
  - 対応した式の種類：
    - 四則演算（`i1 + i2`、`r1 ÷ r2` など。整数型同士は切り捨て、実数型が絡む場合は浮動小数点）
    - 比較式（`a > b`、`a ≦ b`、`a = b`、`a ≠ b` など）
    - 論理演算（`t and f`、`t or f`、`not t`）
    - 配列アクセス（`arr[1]`、`mat[i, j]`）
    - 関数呼び出し（`add(x, y)`）
    - 上記の組み合わせ（`add(x, y) > 5`、`arr[1] + arr[2]` など）
  - エラー時はウォッチ式欄に `エラー: <メッセージ>` を表示
  - 変更ファイル：
    - `evaluator.ts`：`evaluateExpressionPublic()` を追加
    - `runtime.ts`：`evaluateExpressionInContext()` を追加
    - `parser.ts`：`parseExpressionPublic()` を `parseConditionExpression()` ベースに変更、前置 `not` に対応
    - `pseudoDebugSession.ts`：`evaluateRequest()` を式評価パイプライン方式に全面書き換え

## [1.0.10] - 2026-06-12

### Fixed

- 整数型同士の `÷` 演算が整数除算（切り捨て）にならなかった問題を修正
  - 例：`整数型: x ← 5` のとき `x ÷ 2 × 2` が `5` になっていたが、正しくは `4`
  - `evaluator.ts` の `evaluateBinaryExpression()` で `÷` の結果を JavaScript の浮動小数点除算のまま返していたため
  - `getExprDataType()` メソッドを新設し、左辺・右辺の宣言型を再帰的に判定
  - 両辺がともに `実数型` でない場合は `Math.trunc()` で切り捨てるよう修正
  - 実数型が絡む場合（例：`実数型: y ← 5.0` → `y ÷ 2 × 2 = 5.0`）は従来どおり浮動小数点除算

## [1.0.9] - 2026-06-10

### Removed

- 出力パネルへの「▶ 実行開始」「▶ デバッグ開始」メッセージを削除
  - 実行・デバッグの動作には影響しない装飾的なメッセージだったため
  - `runCurrentFile.ts` の `printToChannel('▶ 実行開始\n')` を削除
  - `pseudoDebugSession.ts` の `printToChannel('▶ デバッグ開始\n')` を削除

## [1.0.8] - 2026-06-10
 
### Changed
 
- README.md を更新

## [1.0.7] - 2026-06-10

### Added

- F5・Ctrl+F5 キーバインドを追加
  - `F5`：`.pseudo` ファイルを開いている状態でデバッグを開始（非デバッグ中のみ）
  - `Ctrl+F5`：`.pseudo` ファイルを開いている状態で実行（デバッグなし）
- VSCode メニューバーの「実行」メニューに「疑似言語を実行」「デバッグ」を追加
  - `.pseudo` ファイルを開いている場合のみ表示
- デバッグツールバーに「疑似言語を実行」ボタンを追加
  - デバッグセッション中（`debugType == pseudo`）に表示

## [1.0.6] - 2026-06-10

### Fixed

- 変数ウィンドウで二次元配列の子要素が正しく展開されなかった問題を修正
  - 行の値が `3,5` のようにフラットな文字列で表示されていた
  - `variablesRequest()` の配列子要素展開で、各要素が配列（行）かどうかをチェックせず
    `String(v)` で文字列化していたため
  - 行要素が配列の場合は `row:arrayName:rowIndex` キーで新たな `variablesReference` を割り当て、
    2段階ネストで展開するよう修正
  - 疑似メモリビューへの影響なし（データソースが `buildMemoryVars()` → `updateMemory()` の
    独立したパスのため）

## [1.0.5] - 2026-06-10

### Fixed

- 疑似メモリビューで二次元配列の実体セルが表示されなかった問題を修正
  - `buildMemoryVars()` の `nextAddr` 加算が `info.value.length`（行数）しか進まず、列方向の要素がカウントされていなかった
  - `fixedArrayBodyMap` への初回登録タイミングが宣言行（空配列 `[]`）だったため、要素数0で採番が固定されていた

### Changed

- 疑似メモリビューのアドレス採番を**変数エリア／実体エリア分離方式**に全面変更
  - 変数エリア：`0x0001` から連番（スカラー・配列ポインタ問わず宣言順）
  - 実体エリア：`0x0020` から連番（配列実体のみ）
  - 変更理由：IPA疑似言語の配列は参照渡し（参照モデル）であり、ポインタと実体が分離しているモデルの方が意味論的に正確。また、空配列問題・採番ずれ問題が構造的に解消される
  - `pseudoDebugSession.ts`：`nextBodyAddr = 0x0020` フィールドを追加、`buildMemoryVars()` を分離方式に全面書き換え
  - `memoryView.js`：`buildLayout()` を分離方式に全面書き換え（変数エリアは宣言順連番、実体エリアは `arrayAddr` をそのまま使用）

- 疑似メモリビューのグリッドを**2ブロック描画**に変更
  - 変数エリアと実体エリアの間に余白行（`block-spacer`）を挟んで別ブロックとして描画
  - 空白セルが並ぶ問題を解消
  - `memoryView.js`：`renderGrid()` に `renderBlock()` ヘルパーを追加、`varCells` / `bodyCells` に分割して描画
  - `memoryView.css`：`tr.block-spacer td` スタイルを追加

- 二次元配列の実体セルを正しくフラット化して表示
  - `buildLayout()` で二次元配列を `[行,列]` 形式のラベルでフラット展開
  - `buildMemoryVars()` で `reduce` を使い行×列のフラット要素数を正確にカウント

- 配列中間セル間の境界線が消える問題を修正
  - `td.arr-mid` の左右ボーダーを `none` から `1px solid var(--border-color)` に変更

- 疑似メモリビューの矢印（ポインタ → 実体）を縦方向に変更
  - 変更前：ポインタセル右端 → 実体セル左端（横方向、複数配列で交差して見づらかった）
  - 変更後：ポインタセル下端左寄り → 実体セル上端左寄り（縦方向の折れ線）
  - `renderArrows()` の座標計算を `x = left + 10px`、`y = bottom/top` に変更
  - 2ブロック分離により上下に配置されたグリッド構造と自然に対応

## [1.0.4] - 2026-06-10

### Added
- 二次元配列の要素アクセスにIPA仕様の `[i, j]` 記法を追加
  - `parseArrayIndices()` を `[i, j]` カンマ区切り形式に対応
  - `[i][j]` 二重ブラケット記法は廃止し、`[i, j]` に統一
  - evaluator.ts・ast.ts は無変更（既存の `indices[0]` / `indices[1]` ロジックをそのまま利用）
  - 令和7年度公開問題 問5・サンプル問題 問4 などのIPA公式問題をそのまま実行可能に
## [1.0.3] - 2026-06-09

### Fixed

- 疑似メモリビューに箱が表示されなくなった問題を修正
  - `src/views/memoryView.js` の `renderGrid` 関数の引数が `(variables)` のみで、
    関数内で参照していた `maxAddrOverride` が未宣言のままになっていた
  - `function renderGrid(variables, maxAddrOverride)` に修正

## [1.0.2] - 2026-06-09
 
### Fixed
 
- 変数ウィンドウの配列アドレス表示を疑似メモリビューと一致させた
  - `buildMemoryVars()` のアドレス採番ロジックを疑似メモリビューの `buildLayout()` と完全に統一
  - 旧実装では `nextArrayAddr = vars.size + 1` を使っていたため、変数が増えるたびに配列の実体アドレスがずれていた
  - `addr` 1本で「ポインタセル消費 → 配列実体消費」を宣言順に処理する1ループ方式に変更し、`vars.size` への依存を廃止
- 変数ウィンドウのスカラー変数（配列以外）からメモリアドレス表示を除去
  - 配列変数のみ `Array(N) @ 0x????` 形式でアドレスを表示するように変更

### Changed
 
- `pseudoDebugSession.ts` の `buildMemoryVars()`
  - `ptrCount` / `nextArrayAddr = ptrCount + 1` を廃止
  - `arrayToAddr: Map<object, number>` と単一カウンター `addr` で採番を統一
  - 非配列変数は `addrMap` に登録しない（変数ウィンドウへのアドレス表示なし）
- `pseudoDebugSession.ts` の `variablesRequest()`
  - 配列変数のみ `addrMap` を参照してアドレスを表示
  - 非配列変数はアドレスなしでシンプルに値のみ表示

## [1.0.1] - 2026-06-09
 
### Added
- スニペットに記号入力ショートカットを追加
  - `〇`（まる / maru）
  - `×`（かける / kakeru）
  - `÷`（わる / waru）
  - `←`（だいにゅう / dainyuu）
  - `≧`（いじょう / ijou）
  - `≦`（いか / ika）
  - `≠`（ひとしくない / hitoshikunai）
  - 各記号はひらがな・ローマ字どちらのprefixでも入力可能

## [1.0.0] - 2026-06-09

### Added
- 初回リリース
- 疑似言語ファイル（.pseudo）の実行
- ステップ実行（F10 / F11）
- ブレークポイント
- 変数ウィンドウ・Watch式・コールスタック
- シンタックスハイライト
- コードフォーマッター
