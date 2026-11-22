少し前に、Xで RFC 9457 "Problem Details for HTTP APIs" が話題になっていて、気になるトピックだったので簡単にまとめておく。

https://www.rfc-editor.org/rfc/rfc9457.html

## 概要

HTTP APIのエラーレスポンスで問題の詳細を伝えるための標準的なフォーマットを定義する文書。

> This document defines a "problem detail" to carry machine-readable details of errors in HTTP response content to avoid the need to define new error response formats for HTTP APIs.

ステータスは **Proposed Standard** 。仕様が安定して現場で実装や検証が始まり、Internet Standard に昇格するために必要な成熟を目指す段階だといえる。

### 例

RFC 9457の中で紹介されている最も基本的な例がこちら。
メディアタイプは、 ` application/problem+json` を指定する。
各メンバーについては、この後簡単に見ていく。

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json
Content-Language: en

{
 "type": "https://example.com/probs/out-of-credit",
 "title": "You do not have enough credit.",
 "detail": "Your current balance is 30, but that costs 50.",
 "instance": "/account/12345/msgs/abc",
 "balance": 30,
 "accounts": ["/account/12345",
              "/account/67890"]
}
```

### type

typeメンバーは問題種別の識別子を指定する。URI形式が求められ、HTTP(S)で人間が読むことができるドキュメントを参照することが推奨される。
tagというURIスキームを使用して、単に問題種別を一意に識別する値を指定することも可能。

### title

人間が理解できる形式の問題の概要を提供する。

### detail

クライアントが問題を修正するのに役立つ問題の説明を提供する。

### instance

発生した問題自体を識別するURIを指定する。
参照可能なURIの場合、問題の詳細オブジェクトへのリンクとして機能し、参照不可なURIの場合は単に問題の識別子として機能する。

### extension

標準の基本メンバーだけでは伝えきれない詳細情報を提供する。
ただし、問題タイプに応じて固有のメンバーである必要がある。問題タイプに応じた固有のメンバーに関する例は下記の通り。

- 「残高不足」の問題タイプに残高を示す balance メンバーを追加する
  * 記事の冒頭で、RFC 9457から引用した例がこれに該当する
- 「バリデーションエラー」の問題タイプに個別のエラーリストを示す errors メンバーを追加する
  * こちらもRFC 9457の中で紹介されている例を下記に記載

```
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json
Content-Language: en

{
 "type": "https://example.net/validation-error",
 "title": "Your request is not valid.",
 "errors": [
             {
               "detail": "must be a positive integer",
               "pointer": "#/age"
             },
             {
               "detail": "must be 'green', 'red' or 'blue'",
               "pointer": "#/profile/color"
             }
          ]
}
```

## 何が嬉しいのか

このRFC 9457が普及することは、Web APIの開発者にとって何が嬉しいのか。大きく下記の2点。

1点目は悩みがちなエラーレスポンスの形式にもう悩まなくて済むという点。すでに運用が開始されているAPIであれば既存の規約に従うことが多いと思うが、いざ新規サービスを立ち上げるとなると、悩ましいポイントのひとつだったりする。

2点目は、開発者間の共通理解が形成されて不要なコミュニケーションを減らせる点。これはすでに運用されているサービスでも、新規サービスでも同様にメリットがある。それぞれの開発者の経験に基づく独自のフォーマットがぶつかることがなくなり、確認や調整のコストを削減できる。つまり、世の中の衝突が減るということ。(平和)


既に運用が開始されているAPIに対していきなり導入していくことは難しいが、少なくとも今後新規サービスの開発をする際は積極的に採用していきたい所存。
