# Scratchバックパック 外部サイトからの接続検証

実験ページ: https://riki-kusa.github.io/-kennsyou-/

## 目的
第三者サイト（GitHub Pages）から、Scratchのバックパックサーバーへの通信・認証・書き込みが実現可能か、段階的に切り分ける。

## 実測（2026-11-12、利用者のブラウザ）

- A: GET https://backpack.scratch.mit.edu/ → HTTP 200、response.type = cors。ブラウザが応答を取得できた。
- B: 不正なテスト用認証・不正形式のPOST → HTTP 403、response.type = cors。少なくともその条件でサーバーから拒否応答を取得できた。
- これらは「外部サイトから正常にバックパックへ保存できる」の証明にはならない。

## 公式コードの確認事項
- Scratch GUI はバックパックの list / save / delete のインターフェースを持つ。
- セッション情報の型に username / token が含まれる。
- これはScratch内部での機能で、任意の第三者Webサイトへの認証委任を意味しない。

公式コード: https://github.com/scratchfoundation/scratch-editor/blob/develop/packages/scratch-gui/src/gui-config.ts

## 第三者認証について
Scratch Authはコメントやクラウド変数によってScratchアカウントの所有者を確認する独立サービスであり、Scratch本体のバックパックへの書き込み許可を与えるサービスではない。サービス側のドキュメントには「identity ... but nothing more」と明記されている。

出典: https://auth.itinerary.eu.org/documentation/using-scratch-auth

現時点で公式に利用可能な「第三者サイトへバックパック書き込み権限を委任する」OAuth等の仕組みは確認できない。このことは不可能性の証明ではない。

## Cの追加実験（2026-11-12）
- Cookieを含める設定（credentials: 'include'）を付けた、バックパックサーバーのルート宛てのGETを行う。
- 成功なら、その応答について認証付きCORSのブラウザ条件を満たすことを示すが、Cookieが送信されたか、バックパックの認証を通ったかは示さない。
- 失敗なら、認証付きCORS・ネットワーク・ブラウザのCookie制限等を区別するためNetworkタブの情報が必要。
- 認証情報、レスポンス本文、バックパック項目は表示・保存しない。

## 安全上の方針
- パスワード、Cookie、セッションID、X-Tokenを検証サイトに入力させない。
- Scratchの認証情報をGitHubにコミット・公開しない。
- 正式な書き込み権限のない状態で本物の素材をPOSTしない。
- 本人確認（認証）と操作権限（認可）を区別する。

## 設計上の暫定結論
1. HTTP・CORSのテスト結果だけでは、ワンクリック保存の可否は確定しない。
2. Scratch Authだけではバックパックに保存する権限を得られない。
3. 正規の認可経路が確認できない場合は、手動のSVG/MP3インポートまたは利用者が明示的に許可したブラウザ拡張機能経由の操作を別案として検討する。


## 最終判断（2026-11-12）

**結論：一般の外部Webサイトから、Scratchにログイン済みの任意の利用者のバックパックへ、そのサイトのボタンだけで直接SVG・MP3を保存する方法は、現時点で確認できない。通常のWeb機能として実装可能と見なさない。**

確定した事実と留保：

1. Scratch公式エディターのバックパック保存インターフェースには `save()` がある。セッションは `username` と `token` を含む。
   - https://github.com/scratchfoundation/scratch-editor/blob/develop/packages/scratch-gui/src/gui-config.ts
2. 過去のScratch公式フォーラムの利用例でも、バックパックAPIの書き込みには `X-Token` などが使われる。2019年の投稿を現行仕様の確証として扱わない。
   - https://scratch.mit.edu/discuss/post/3479674
3. Scratch Authはアカウントの本人確認をする独立サービスで、Scratchバックパックへの操作権限を委任しない。公式ドキュメントに「identity ... but nothing more」とある。
   - https://auth.itinerary.eu.org/documentation/using-scratch-auth
4. 外部サイトではScratch自身のログインセッション／トークンを自由に読めない。パスワードやトークンを利用者に貼り付けさせる実装は採用しない。
5. ユーザー実測では認証情報なしのGETは200、無効な認証のPOSTは403で、ともにCORS応答を読めた。一方、`credentials: include` を指定したGETは `TypeError: Failed to fetch` で失敗した。CORS、Cookieポリシー、ネットワーク等のどれが原因かは、この結果だけでは確定しない。
6. したがって「絶対に技術的に不可能」とは断定しないが、**通常の外部Webサイト向けの正規の安全な保存権限委任手段は確認できず、ワンクリックの直接保存機能は現段階では採用しない。**

実用的な代替案：SVG/MP3のダウンロードとScratchのアップロードUIを使う。将来的にScratch公式が認証連携APIを提供した場合は再検討する。既存のブラウザ拡張機能を検討する場合は、外部サイト自体のワンクリック機能とは別物として、利用者の明示的な許可と安全性を考慮する。

追加したGitHub Actionsのヘッダー検証については、コミットの通常のstatus APIから実行結果を取得できなかった。実行成否について憶測しない。この結果にかかわらず、現在の最重要の制約は認証・権限委任が確認できない点にある。
