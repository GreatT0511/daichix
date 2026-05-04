# 変更履歴

GAS版で「カルーセルの4・8・9番目のパックがクリックできない」「キラキラ加工が右下に剥がれて見える」問題を、GitHub Pagesにデプロイしながら検証・修正した記録。

## 1. GitHub Pagesにデプロイ
- `index.html` としてHTMLを保存し、`main` ブランチにプッシュ。
- リポジトリを Public に変更し、GitHub Pages を有効化。
- 公開URL: https://greatt0511.github.io/daichix/
- GitHub Pages でも同じ症状が出たので、原因は GAS ではなくコード側と確定。

## 2. カルーセルのクリック問題

### 試したこと（経緯）

**(a) `backface-visibility: hidden` の削除**
- `.carousel__cell` から `backface-visibility: hidden` / `-webkit-backface-visibility: hidden` を削除。
- 360°に並んだ10枚のうち、裏面を向いているカードが描画もクリックもされない問題を解消する狙い。
- → これだけでは直らなかった。

**(b) 中央カードのみクリック可能にする方式**
- `.carousel__cell` に `pointer-events: none` を設定。
- `.carousel__cell.is-selected` に `pointer-events: auto` を設定。
- 中央（選択中）のカードだけクリックを受け付ける設計に変更。
- → 環境によっては中央でもパック4・8・9だけ押せない症状が残った。

### 最終的な解決策
**(c) 専用の「このパックを開ける！」ボタンを追加**
- 3Dカルーセル上のクリック判定に頼らず、明示的な開封ボタンに分離。
- `.open-btn` クラスを新規追加（黄色いCTAボタンのスタイル）。
- HTML: 矢印ボタン群の下に `<button id="btn-open">このパックを開ける！</button>` を追加。
- JS: `btn-open` のクリックで `goToCutScreen()` を呼ぶハンドラを追加。
- カードセル自体は `pointer-events: none` のまま（演出専用）、`cursor: default` に変更。
- 操作フロー: ← →（または矢印キー）で目的のパックを中央に → 「このパックを開ける！」ボタンで開封。

## 3. キラキラ（ホロ）加工の修正

### 元の問題
- `.holo-effect::after` が `top: -100% / left: -100% / width: 300% / height: 300%` で巨大に作られ、`translate(50%, 50%)` で右下に動いていた。
- ループのリセット時にスナップして、加工がカードから「剥がれた」ように見える。

### 修正1（中間版）
- `::after` を `inset: 0` でカード内に収める。
- `background-size: 250% 250%` で大きめのグラデを敷き、`background-position` を `150% 150% → -50% -50%` でアニメ。
- 両端を完全透明にしたものの、中央の光の帯の端が見える瞬間があった。

### 修正2（最終版）
- グラデーション全体を**広く**して両端も薄い白に（完全透明ゾーンをなくす）→ 帯の境界線が見えない。
- `background-size: 200% 200%`、位置を `10% 10% ↔ 90% 90%` で**往復**（端まで行かない）→ ループのスナップ自体が起きない。
- `animation: holo-shine 3.2s infinite ease-in-out` で自然な加減速。
- `mix-blend-mode: overlay` を追加し、カードの色と馴染ませてキラキラ感を強化。

## コミット履歴

| コミット | 内容 |
|---|---|
| `b837529` | Add pack opening game HTML for GitHub Pages testing |
| `8562063` | Remove backface-visibility hidden so all carousel cells stay clickable |
| `45c1565` | Disable pointer-events on non-selected carousel cells |
| `21fe039` | Add explicit open button to bypass 3D click capture |
| `9701cb2` | Fix holo shine: confine to card and avoid loop snap |
| `b40a4a6` | Widen holo gradient and oscillate position so edges never enter view |

## GAS版に同じ修正を反映する手順
このリポジトリの `index.html` の差分（特に以下の3箇所）を、GAS側の HTML ファイルにそのまま反映すれば同じ挙動になる。

1. `.carousel__cell` のCSS（`pointer-events: none`、`backface-visibility` 削除）。
2. `.open-btn` のCSS、`#btn-open` のHTML、`btn-open` クリックハンドラの追加。
3. `.holo-effect::after` と `@keyframes holo-shine` の差し替え。
