---
title: "Jetpack ComposeでScreenを作成する方法（単純な画面）"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android","jetpackcompose"]
published: true
---

# 背景

Jetpack Composeを使い始めると、新しくモジュールであったりレイアウトを作成するときには、必ずJetpack Composeを使いたいですよね。私も最近はほぼXMLは触らずにJetpack Composeでレイアウト作成をしています。

しかし、ActivityやFragmentの一部に対してComposeViewを使ってRecyclerViewのViewHolderのレイアウトをCompose化することから始めていて、まだScreen全体をCompose化したことが無い状態です。いま新しくリニューアル開発作業をしていることがあり、画面を１から作る業務が発生したので、ここでScreenからCompose化する方法を進めたい。そのときに考えたことなどを書いていきます。

# 対象読者

- Jetpack Compose導入始めたばかりの人
- Jetpack Composeで画面の要素はComposeに移行しはじめたが画面も書き換える大きな１歩を踏み出す人

# モジュールについて

いまは画面や機能単位でModule分解しているのを見かける（主にNow In Android、DroidKaigiのアプリなど）ので、今回作る画面についても、featureモジュールを作成し、featureモジュールの中に今回作る画面のモジュールを作成しました。

```
root
  -feature
    - screen A
    - screen B
```

リニューアルで作成する画面は、上記のようにfeatureモジュールに画面ごとにモジュール分解して作成しようと思っています。いま心配なのは、今はscreen Aしかなくて、screen Aに依存関係としてComposeの依存関係を追加しているが、同じような依存関係の追加が画面ごとに追記が必要そうなので、どこかで共通管理する必要がありそう。build-logicモジュールの作成も検討が必要なんでしょう。

```
dependencies {
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.foundation)
    implementation(libs.androidx.compose.ui.tooling)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    testImplementation(libs.junit)
}
```

# 今回作る画面

今回作る画面は非常にシンプルです。

```
　　画像
　　テキスト（タイトル）
　　テキスト（説明）
　　ボタン（次へ）
```
このような要素だけを持つ画面です。ボタンタップで次画面に遷移する形です。

# 画面遷移について

作業するアプリは、Compose化は進んでおらず、基本的にはActivityが複数あり, Fragmentもある状態です。今回新規で作成する画面も、Activity内で表示をする必要があり、次へボタンをタップしたら、ComposeViewの表示を非表示にして、先の画面を見せる実装が要件とフィットしそうなので、遷移はしない方式で進めます。なのでNavHostControllerなどは使用しない。

# Scaffoldを使うべきなのか？

画面を作るとき、Scaffoldを使うケースが多いのかと思います。Scaffoldについて、調べると以下のような特徴のようです。

> Scaffold コンポーザブルは、マテリアル デザイン ガイドラインに沿ってアプリの構造をすばやく構築するために使用できる簡単な API を提供します。Scaffold は、複数のコンポーザブルをパラメータとして受け取ります。次に例を示します。
> topBar: 画面上部のアプリバー。
> bottomBar: 画面下部にあるアプリバー。
> floatingActionButton: 画面の右下に表示されるボタン。重要なアクションを表示できます。

[Android Developers Scaffoldより](https://developer.android.com/develop/ui/compose/components/scaffold?hl=ja)

色々考えましたが、今回はAppBarもBottomBarもないので、Boxで画面を作っても全然良さそうと思いました。なので面白みがない結論ですが、Boxでいきます。

# Themeは使うか？

デザイナさん的に、画面のデザインにマテリアルテーマを使って進めるかが大事に思います。今回、新規アプリでもないため、既存のアプリのリニューアルになりますが、既存の状態としても、マテリアルデザインで色や文字サイズなどを管理している形にはなっておらず、画面ごとに色などをリソースファイルを指定して管理しているような形になっています。そのため、今回も現状はThemeなどは作らず進める形になりそう。ただ、デザイナさんとコミュニケーションを持ち、今後の改修では統一したテーマがありそうかは聞いておこうと思います。

# まとめ

今回はシンプルな要件の画面について、新規で画面を作る際に検討したことを書きました。まとめとしては以上です！

- AppBarなどが無いすごく単純な画面のため、Scaffoldの使用せず、Boxで画面を作る。
- MaterialThemeや自社独自のテーマなどは現状なさそうなので、Themeも使用しない。ただし、デザイナさんにテーマ作るように提案してみる。
- マルチモジュール構成としてリニューアル画面の作成は進める。ただし、共通の依存関係の追加にはbuild-logicなどのモジュールを作るなど工夫が必要そう。

最後まで読んでくださりありがとうございました！
