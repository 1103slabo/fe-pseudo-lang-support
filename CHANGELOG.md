# Changelog

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
