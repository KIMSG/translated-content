---
title: 414 URI Too Long
slug: Web/HTTP/Status/414
tags:
  - HTTP
  - Reference
  - クライアントエラー
  - ステータスコード
translation_of: Web/HTTP/Status/414
---
{{HTTPSidebar}}

HTTP の **`414 URI Too Long`** レスポンスステータスコードは、クライアントがリクエストした URL が、サーバーが解釈しようとするものよりも長いことを示します。

これが発生する条件はわずかです。

- クライアントが、 {{HTTPMethod("POST")}} リクエストを長いクエリー文字列をもつ {{HTTPMethod("GET")}} リクエストに変換した場合。
- クライアントがリダイレクトのループに陥った場合 (たとえば、リダイレクトされた URL 接頭辞が自分自身の接尾辞を指していた場合や、受け取った URL の扱いを誤った場合)。
- 潜在的なセキュリティホールを利用しようとしているクライアントがサーバーを攻撃している場合などです。

## ステータス

```
414 URI Too Long
```

## 仕様書

| 仕様書                                                       | 題名                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------- |
| {{RFC("7231", "414 URI Too Long" , "6.5.12")}} | Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content |

## 関連情報

- {{HTTPStatus(431, "431 Request Header Fields Too Large")}}
- {{Glossary("URI")}}
