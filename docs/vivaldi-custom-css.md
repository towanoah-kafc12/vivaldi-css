# Vivaldi カスタム CSS 運用メモ

このリポジトリの `main.css` は、Vivaldi の UI に読み込ませるカスタム CSS です。Vivaldi 本体の UI は内部実装であり、アップデートで DOM 構造やセレクタが変わることがあります。この文書は、現在確認できているレイアウトと、変更時に守るべきスタック順をまとめたものです。

## 現在のレイアウト

- タブバー: 左。通常は `32px`、ホバー中は `150px` に拡大。
- アドレスバー: 上。Vivaldi の自動非表示機能を使用。
- サイドパネル: 右。
- ステータスバー: 下。Vivaldi の自動非表示機能を使用。

タブコンテナは `position: absolute` で重ねて表示します。これにより、タブバーを展開してもページ領域の幅を押し縮めません。

## タブバーの重要なルール

`main.css` の以下のセレクタが、左右のタブオーバーレイを制御します。

```css
#tabs-container.left,
#tabs-subcontainer.left,
#tabs-container.right,
#tabs-subcontainer.right
```

`main.css` の現在の基準値は次のとおりです。

```css
#tabs-container.left,
#tabs-subcontainer.left,
#tabs-container.right,
#tabs-subcontainer.right {
  top: 0;
  width: 32px !important;
  z-index: 99999999;
}

#tabs-container.left:hover,
#tabs-subcontainer.left:hover {
  width: 150px !important;
}
```

意図した値は次のとおりです。

| プロパティ | 値 | 理由 |
| --- | --- | --- |
| `position` | `absolute !important` | タブバーをページ領域の上に重ねるため |
| `top` / `bottom` | `0` | ウィンドウの上下端までタブバーを伸ばすため |
| 通常幅 | `32px` | 折りたたみ時の幅 |
| ホバー幅 | `150px` | タブ名を表示する幅 |
| `z-index` | `99999999` | 展開したタブバーを他の通常 UI より前面に出すため |

`z-index` を低くすると、CSS 上ではホバー幅が `150px` になっていても、展開部分が別の UI の背面に入り、見た目は細いままになります。タブバーのレイヤーを下げるのではなく、タブバーより前面に出す必要がある要素だけを個別に上げます。

## 重なり順のルール

### 上部の自動非表示バーと Vivaldi メニュー

Vivaldi の上部自動非表示コンテナは `.auto-hide-wrapper.top` です。この要素には、アドレスバーと左上の Vivaldi メニューボタン（`.vivaldi`）が含まれます。

```css
.auto-hide-wrapper.top {
  z-index: 100000000 !important;
}
```

これは高い `z-index` を持つタブオーバーレイよりも上部バーを前面に出します。隠れている自動非表示バーには Vivaldi 標準の `pointer-events: none` が適用されるため、表示されていない間もタブバー操作を妨げません。

> **避けること:** 左タブコンテナに `top: 40px` のようなオフセットを与えてメニューを避ける方法。上部に不自然な空白ができ、タブバーがウィンドウ上端まで届かなくなります。

### ワークスペース選択メニュー

タブバーのワークスペースボタンを押すと、Vivaldi は `.WorkspacePopup` を含む `.button-popup` を表示します。タブバーより前面にする対象は、このポップアップに限定します。

```css
#browser .button-popup:has(.WorkspacePopup) {
  z-index: 100000000 !important;
}
```

全ての `.button-popup` を上げると他のツールバーメニューまで影響を受けるため、必ず `:has(.WorkspacePopup)` を残してください。

## 右サイドパネルのホバー表示

右サイドパネルを閉じた状態（`#panels-container.right.icons:not(.switcher)`）では、`#switch` がパネルアイコンバーです。この状態だけを対象に、コンテナを右端の絶対配置オーバーレイへ変更します。

- 通常時はコンテナと `#panel_switch` を `1px` に縮め、右端にホバー検知領域だけを残す。`0px` にするとホバーできなくなる。
- `:hover` または `:focus-within` のときだけコンテナを `35px`、`#switch` を `34px` に戻す。これにより、アイコンバーは表示されるが Web ページの幅を押し縮めない。
- `.icons` 状態だけに限定する。パネルを開いた状態には適用しないため、Vivaldi 標準の固定幅／オーバーレイ設定とパネルを開く操作を維持できる。
- `#switch` の背景には不透明な `var(--colorBg)` を使う。`--colorBgAlphaBlur` は半透明・ぼかし用のため、従来と同じ不透明なバー背景には使わない。
- `.density-on` では Vivaldi 標準に合わせ、幅へ `var(--densityGap) * 2` を加える。

Vivaldi の状態クラスは内部実装であり、更新時に変わる可能性があります。機能が崩れた場合は、インストール済み `common.css` で `#panels-container`、`.icons`、`#switch`、`#panel_switch` を確認してください。

## Vivaldi 更新時の調査方法

Vivaldi のアップデート後に表示が崩れた場合、まず UI セレクタの変更を確認します。今回確認した Vivaldi 8.1.4087.55 の CSS は次の場所です。

```text
%LOCALAPPDATA%\Vivaldi\Application\<version>\resources\vivaldi\style\common.css
```

特に次の文字列を検索します。

- `#tabs-container`、`#tabs-subcontainer`、`#tabs-tabbar-container`
- `#panels-container`、`.icons`、`#switch`、`#panel_switch`
- `.auto-hide-wrapper.top`
- `.vivaldi`
- `.WorkspacePopup`
- `.button-popup`

本体の `common.css` や `bundle.js` は変更しません。リポジトリの `main.css` に、対象を絞った上書きルールだけを追加します。

## 変更後の確認手順

1. `main.css` の CSS 構文と括弧の対応を確認する。
2. Vivaldi を再起動するか、カスタム CSS を再読み込みする。
3. 次を実画面で確認する。
   - 左タブバーが上端から下端まで表示される。
   - 左タブバーをホバーすると `150px` 幅まで広がる。
   - 上部バー表示中、左上の Vivaldi アイコンが見えてクリックできる。
   - アドレスバーからマウスを離すと自動的に隠れる。
   - ワークスペース選択メニューが展開したタブバーより前面に表示され、項目を選択できる。
   - 右サイドパネルを閉じた状態では、右端ホバー時だけアイコンバーが不透明な背景で表示され、Web ページの幅が変わらない。
   - 右サイドパネルのアイコンをクリックした後は、Vivaldi の設定どおりに固定幅またはオーバーレイでパネルが開く。
4. 問題があれば、まず `z-index` の競合と、ポップアップの実際の親要素を確認する。タブコンテナの `top` を変更して回避しない。
