# app-itonami-open-ossekai — Open Ossekai

おせっかい — 需給のずれ（arbitrage の機会）を公共財のカタログとして記録し、本人の同意にもとづく情緒（jocho）の評価を手元で残します。推論は含みません。

Tauri **3.0.0-alpha.6** の独立 app（`tauri-runtime-wry`、OS の WebView）と、同じファイルの static Web ビルドです。記録はすべてこの端末の中（localStorage、key `itonami.open-ossekai.v1`）に保存し、外部へは通信しません。外部操作の宣言もありません（`dist/descriptor.json` の `operations: []`）。保存に失敗したときは画面に表示します。

## できること

| 操作 | 内容 |
| --- | --- |
| 機会を登録 (registerArbitrage) | — |
| 機会を見る (getArbitrage) | — |
| 機会の一覧 (listArbitrage) | — |
| 情緒を記録 (recordJocho) | — |
| 情緒の一覧 (listJocho) | — |
| 情緒を見る (getJocho) | — |
| 記録の数 (coverage) | — |

記録（collection）: `cloud.itonami.apps.openOssekai.arbitrageOpportunity`、`sealed:cloud.itonami.apps.openOssekai.jochoAssessment`

## 補足

- 情緒の評価は read-cap（書いた本人と、明示した受け手の DID）つきの sealed record として保存し、read-cap に入っていない DID には返しません。**ただし暗号化はしていません。** 旧 SDK の E2E 暗号化（鍵の共有つき）は移していないため、評価はこの端末の localStorage に平文の JSON で残ります。端末をほかの人と共有する場合は注意してください。E2E を戻すには、鍵の保管と受け渡しを含めた別設計が必要です。
- 旧 Worker が担っていた L1 の推論（intel brief・採点）と sales lead の送出は含みません。
- `worker/cljs/`（旧 UI）と `worker/kotodama.jsonld` は残しています。この app のビルド・テストでは使いません。

## 構成

| パス | 役割 |
| --- | --- |
| `src/domain/` | domain logic（TypeScript）。ビルド時に Node の type stripping で JS にします（bundler・依存なし） |
| `src/local-store.js` | この端末内の record store。domain が使う `read` / `write` を localStorage で実装 |
| `src/app-def.js` | 画面の操作と domain 関数・collection の対応 |
| `src/ui/` | 依存なしの UI。DOM はテキストノードだけで組み、HTML 文字列を使いません（CSP `style-src 'self'`） |
| `vendor/` | kotoba-lang/kotoba の `kotoba-client-runtime.js` / consent CSS と `PINS.json`。`scripts/build.mjs` が digest のずれで失敗します |
| `src-tauri/` | Tauri v3 host。read-only command は `awai_host_info` の 1 つだけ。filesystem・network・shell・installer 権限なし |


## コマンド

```sh
npm ci
npm test              # build してから domain と local-only のテスト（local-only.test.mjs, open-ossekai.test.ts）
npm run build         # dist/ と SHA-256 の release.json
npm run build:desktop # native desktop 実行ファイル（bundle・署名なし）
```

## 状態と制限（AWAI-3079, 2026-11-17）

- 旧構成（Cloudflare Worker の XRPC dispatcher / adapter、`wrangler.jsonc`、`@etzhayyim/sdk`）は削除しました。履歴は git にあります。
- 検証済み: `npm test`、`npm run build`、`cargo check`（Tauri 3.0.0-alpha.6）。
- 未検証: native window の起動、`tauri build`、Android、署名、配布。アイコンは app-soshiki の仮アイコンです。
