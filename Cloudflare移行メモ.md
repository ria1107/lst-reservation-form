# GitHub Pages → Cloudflare Pages 移行メモ

作成日: 2026-09-08
対象: LST撮影会予約フォーム(表側の静的サイトのみ。裏側のGAS処理は変更なし)

## 背景

このフォームは撮影会の予約受付＋Square事前決済(33,000円)への誘導を行う商用フォームのため、
GitHub Pagesの利用規約(商取引を主目的としたサイトでの利用は非推奨)に触れるリスクがあった。
無料のまま商用利用でき転送量が無制限のCloudflare Pagesへ移行した(印鑑注文フォームで先行実施した手法を踏襲)。

## 1. 現状構成の確認結果

- フォルダー直下に`index.html`と`img/`のみ(サブフォルダー構成なし)
- `grep -rn '\.\./'`で親ディレクトリ参照が無いことを確認済み → 単独でそのままCloudflare Pagesに載せ替え可能
- `netlify.toml`・`_redirects`・`_headers`等のホスティング固有設定ファイルは無し

## 2. Cloudflare Pages側の設定値

| プロジェクト名 | Production branch | Build output directory |
|---|---|---|
| `f3-lst-reservation` | `main` | `/`(リポジトリ直下そのまま) |

CLIでの直接デプロイ方式(GitHub連携は今回未実施)。コマンド:

```
npx wrangler pages project create f3-lst-reservation --production-branch=main
npx wrangler pages deploy . --project-name=f3-lst-reservation --branch=main --commit-dirty=true
```

**ダッシュボードでのGitHub連携は未実施のため、今後`index.html`を更新しても自動では反映されない。**
更新のたびに上記の`wrangler pages deploy`を再実行するか、後日ダッシュボードで「Connect to Git」を設定すること。

## 3. 実機確認結果(2026-09-08)

| 項目 | 結果 |
|---|---|
| 新URL | https://f3-lst-reservation.pages.dev |
| HTTPステータス | 200 |
| タイトル | 撮影会予約 \| Leading Style Tokyo |

## 4. 旧ホスティング(GitHub Pages)の状況

| 項目 | 結果 |
|---|---|
| URL | https://ria1107.github.io/lst-reservation-form/ |
| 状態 | 稼働中(HTTPステータス200、今回は停止していない) |

→ 今回は新旧並行稼働。GitHub Pages側の停止・削除は社長の最終確認後に別途対応する。

## 5. 残作業

1. 案内URLをどのタイミングで`f3-lst-reservation.pages.dev`(または独自ドメイン)に切り替えるか社長と相談
2. Notion「DB_アプリURL台帳」の更新
3. 問題なければGitHub Pages側の停止(今回は未実施・指示があるまで着手しない)

## 参考

同じ手法での移行実績: `会社基盤/products/印鑑販売_行政書士様向け/Cloudflare移行メモ.md`
