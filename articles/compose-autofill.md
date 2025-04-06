---
title: "Jetpack ComposeでAutofillManagerを使ってID・パスワードを自動入力する方法"
emoji: "👏"
type: "tech"
topics: []
published: false
---

Jetpack Composeでは、AutofillManagerを活用することで、IDやパスワードの自動入力を実装できます。この記事では、ComposeアプリでAutofill機能を組み込む方法を、具体的なコードとともに解説します。


BasicTextFieldの代わりにOutlinedTextFieldなどを使う場合も、同様の設定が必要です。

Androidの設定で「パスワードの自動入力」が無効になっていると動作しません。

🧪 デバッグのヒント
Android端末の「設定 > システム > 言語と入力 > 自動入力サービス」で有効になっていることを確認してください。

ログ出力でAutofillManagerの呼び出しを確認するには、Log.dなどを適宜追加して追跡するとよいです。

📚 参考資料
公式ドキュメント - Autofill Framework

Jetpack ComposeのAutofill対応
