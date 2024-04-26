---
title: "KMPの始め方１"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [KMP]
published: true
---

# 背景

KMPのアプリ開発に取り組みたいと考えたのですが、実際に作ってみないと理解が進まないので、簡単なアプリを作成してみます。

# 作るもの

- AppBar

検索窓、ベルアイコンを持つ一般的なAppBar。

- Bottom Navigation

・アイコンとラベルを持つメニュー。
・タップしたらハプティックフィードバック。
・選んだメニューをハイライト。

- Bottom Sheet

・一般的なBottomSheet。

- WebView

・Pull To Refreshを制御できる？
・UserAgentはカスタマイズできる？
・JavaScriptInterfaceは受け取れる？

- AppLinks

特定のURLパターンで開いて、アプリ起動する。

- API通信

適当なAPI引いてアプリ表示まで。

- DataBase

起動日、氏名を表示のみ保存するようなDB。

# プロジェクト作成

- [Kotlin Multiplatform Wizard](https://kmp.jetbrains.com/?_ga=2.219420249.1973960092.1714167749-1154006750.1666066378&_gl=1*j6te2z*_ga*MTE1NDAwNjc1MC4xNjY2MDY2Mzc4*_ga_9J976DJZ68*MTcxNDE3MjIxMi4yNS4xLjE3MTQxNzIzNjguNTguMC4w)を使用してプロジェクトを作成

- kdoctorのインストール

```
brew install kdoctor
```

実行したらオールグリーン。Flutterみたいに環境構築が分かりやすい。

![](/images/kmp-tutorial1/2024-04-27-08-17-04.png)

# パッケージ構成

![](/images/kmp-tutorial1/2024-04-27-08-29-17.png)

重要なパッケージ構成。どこに何のファイルを配置するべきなのか。

- composeApp
AndroidアプリにビルドされるKotlinモジュールを持つフォルダ。sharedモジュールをAndroidライブラリとして利用する。

・androidMain：Androidコードを書く。主にJetpack Compose想定。
・composeMain：Compose Multiplatformを使う場合に使用される。

- iosApp
iOSアプリにビルドされるXCodeプロジェクトを持つフォルダ。sharedモジュールをCocoaPods dependencyとして利用する。

・iosApp：iOSコードを書く。主にSwift UI想定。

- shared
Android, iOSの両方で共通なKotlinモジュールを持つフォルダ。Gradleによって共通化のシステムのビルドが行われる。

・composeMain：Android / iOS共通のコードを書く。
・androidMain：Android特有のコードを書く。
・iosApp：iOS特有のコードを書く。

composeMainにInterfaceを定義し、各OSで実装コードを書く。
composeMainに共通のロジックコードを書く。

![](/images/kmp-tutorial1/2024-04-27-08-41-26.png)

[JetBrains KMP doc "Examine the project structure"](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-create-first-app.html#examine-the-project-structure)より

# まとめ

プロジェクトを作成して、アプリ起動まで行いました。

![](/images/kmp-tutorial1/2024-04-27-08-53-31.png)

アプリの修正も、次回以降の記事で提供します！
