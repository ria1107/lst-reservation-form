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

## 6. 【2026-09-18追記】独自ドメイン`lst.f-3.jp`を追加(D-024関連、開発課依頼)

**このフォームはSquare本番決済が実際に動いている生きたシステムのため、今回は静的ファイルの配信設定(Custom domains)のみを変更し、注文APIへのテスト送信・送信ボタンの操作は一切行っていない。**

Cloudflare Pages Custom Domains APIで`lst.f-3.jp`を追加(状態: `pending`、CNAME未設定待ち)。
お名前.com側で`lst` → CNAME → `f3-lst-reservation.pages.dev` の追加が必要
(手順は`会社基盤/products/印鑑販売_行政書士様向け/Cloudflare移行メモ.md`の【2026-09-18追記】3.を参照、同じ手順で6件まとめて設定する)。

品質チェックは`curl -sI https://f3-lst-reservation.pages.dev/`でHTTP 200を確認するに留めた(表示確認のみ)。

## 参考

同じ手法での移行実績: `会社基盤/products/印鑑販売_行政書士様向け/Cloudflare移行メモ.md`(2026-09-18追記に今回のAPI操作の詳細・6フォーム分のCNAME一覧あり)

## 7. 【2026-10-22追記】本番の正体とデプロイ経路(D-106 LIFF対応の本番化で調査)

| URL | 中身 | 備考 |
|---|---|---|
| https://lst-reservation.f-3.jp/ | Pagesプロジェクト `f3-lst-reservation` の本番 | **お客様に案内する本番URL**。中身はpages.devと同一(Cloudflareのメール難読化でメールアドレス部分だけ表示が違う) |
| https://f3-lst-reservation.pages.dev/ | 同上 | LINEのLIFF(本番 `2012156192-mIEOjHYR`)の開き先 |
| https://ria1107.github.io/lst-reservation-form/ | GitHubの `main` をそのまま配信する旧版 | 停止するかは社長判断(下記) |
| `lst.f-3.jp` | 名前解決できない(使われていない) | 9/18に追加申請したがCNAME未設定のまま |

- `f3-lst-reservation` は **GitHub(ria1107/lst-reservation-form)の`main`と連携済み**。出力フォルダーはリポジトリ直下(空欄=直下)で、過去のGit自動ビルドも中身入りで成功していた(個別URLを1つずつ確認済み)。
- 10/03のD-064までは `wrangler pages deploy` の直接アップロードで本番に出しており、GitHubの`main`は古いまま(9/27の6e3e77b)だった → **GitHubの`main`にpushすると、その中身が本番に出る**。今後は「`main`にpush=本番反映」の1経路に統一する。
- 手元フォルダーの `.git` は `refs/heads/main.lock`・`index.lock` の残骸で `main` が空に見える状態(10/04から)。中身は D-068(振込名義カナ・未公開)のコミット 53ad422。10/22の本番化は手元を触らず、一時クローンから push した。手元の修復(main.lockの退避)は社長確認後に行う。
- GitHub Pages版は `main` を配信しているので、`main` へのpushで同じ新版になる(古い版が残る心配はなくなった)。商用利用の規約リスクがあるため、停止(Settings → Pages → Unpublish)を推奨。停止は社長判断。
