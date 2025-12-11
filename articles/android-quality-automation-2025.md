---
title: "Androidアプリの品質を「忘れない」ための自動化戦略"
emoji: "🤖"
type: "tech"
topics: ["android", "ci", "githubactions", "ai"]
published: true
---

## はじめに

私は今年リリースしたライブ配信アプリ「Avvy」のAndroid開発を担当しています。爆速で機能開発を進める中で、ある問題に直面しました。

**「配信画面のUIが、少しズレている」**

新機能を追加したら、既存のボタン位置が微妙に変わっていた。コンポーネントを修正したら、別の画面でレイアウトが崩れていた。コードレビューでは気づかなかった変化が発覚する。

これらの問題に共通するのは、**「人間は忘れる・見落とす」** ということです。

爆速開発の中で、品質チェックを毎回手動でやるのは現実的ではありません。だからこそ、**「忘れない」仕組みを自動化で作る**ことが重要です。

この記事では、2025年に私が取り組んだ（そして現在進行形で取り組んでいる）4つの品質自動化施策を紹介します。

## 4つの自動化施策

| 施策 | 解決する課題 |
|------|-------------|
| Screenshot Testing | UIの視覚的変化を見逃す |
| Library Update管理 | 技術的負債の蓄積 |
| Performance Monitoring | パフォーマンス劣化に気づかない |
| Auto Crash Fix | クラッシュ対応が後回しになる |

## 1. Screenshot Testing - UIの変更を見逃さない

### 課題

UIの変更は、人間の目では見逃しやすいものです。

- パディングが1dp変わった
- ボタンの位置が微妙にズレた
- 意図しない要素が重なっている

特に、Jetpack Composeで開発していると、コンポーネントの変更が思わぬところに影響を与えることがあります。実際に、コメントセクションがギフトボタンを隠してしまうバグが発生したことがありました。

### 解決策

Android公式のScreenshot Testingを導入し、Visual Regression Testを自動化しました。

```kotlin
@Test
fun streamingScreen_displaysCorrectly() {
    composeTestRule.setContent {
        StreamingScreen(/* ... */)
    }
    composeTestRule.onRoot().captureRobolectricImage()
}
```

### テスト対象の状態を絞る

配信画面では、UIが動的に表示・非表示されるケースが多くあります。すべての状態をテストすると保守コストが爆発するため、**全てのUIが表示されている状態**のPreviewでゴールデン画像を作成する方針を取りました。

![Screenshot Testの状態マシン](/images/screenshot-test-state-machine.png)

これにより、テストの頻繁な更新を避けつつ、重要なUIの変更を検出できます。

### ゴールデン画像はCI上で生成する

Screenshot Testで重要なのは、テストの安定性です。ローカル環境でゴールデン画像を生成すると、環境差異（フォントレンダリング、OSバージョン等）によってCIで差分が出てしまうことがあります。

そこで、**ゴールデン画像の生成もGitHub Actions上で行う**ようにしています。これにより、CI環境とゴールデン画像の環境が一致し、安定したテストが実現できます。

### ポイント

- **重要な画面に絞る**: 全画面ではなく、Streaming・Viewerなどの重要な画面に限定
- **全UI表示状態をテスト**: 動的に変わるUIは、全て表示された状態でゴールデン画像を作成
- **CI上でゴールデン画像を生成**: 環境差異による差分を防ぎ、テストの安定性を確保

## 2. Library Update管理 - 技術的負債を溜めない

### 課題

ライブラリの更新は、「今は動いているから後で」と後回しにされがちです。定期的に棚卸しをしないと、いつの間にか古くなってしまいます。

### 現状の棚卸し

まず、主要ライブラリの現状を可視化しました。

| Library | Current | Latest | Priority |
|---------|---------|--------|----------|
| Kotlin | 2.1.0 | 2.2.x | 🔴 High |
| AGP | 8.7.3 | 8.13.x | 🔴 High |
| Compose BOM | 2025.05.01 | 2025.09.01+ | 🟡 Medium |
| Dagger Hilt | 2.54 | 2.57.2 | 🟡 Medium |
| Navigation Compose | 2.9.0 | 2.9.6 | 🟢 Low |
| Lifecycle | 2.8.7 | 2.10.0 | 🟢 Low |
| Coil | 3.0.0-rc01 | 3.x stable | 🟡 Medium |
| Core KTX | 1.15.0 | 1.17.0 | 🟢 Low |

### リスクベースの段階的アップデート

優先度は「更新量の大きさ」と「破壊的変更の可能性」で判断しています。

**Phase 1 - 低リスク（更新量が少ない）**
- Navigation Compose 2.9.0 → 2.9.6
- Core KTX 1.15.0 → 1.17.0
- Lifecycle 2.8.7 → 2.10.0
- Coil rc → stable

**Phase 2 - 中リスク**
- Dagger Hilt 2.54 → 2.57.2
- Compose BOM 2025.05.01 → 2025.09.01

**Phase 3 - 破壊的変更の可能性あり**
- Kotlin 2.1.0 → 2.2.x
- AGP 8.7.3 → 8.13.x

AvvyではKMM（Kotlin Multiplatform Mobile）を採用し、Android/iOS間でUseCase・Repository層を共通化しています。そのため、KotlinやAGPのバージョンを上げる際はKMM側の対応状況も考慮する必要があり、簡単には最新化できません。これがPhase 3を最後に回す理由です。

### 今後の自動化

古いライブラリを片付けたら、以下の自動化を予定しています。

#### Renovate導入

依存関係の自動更新PR作成にRenovateを導入します。

```json
{
  "extends": ["config:recommended", ":dependencyDashboard"],
  "timezone": "Asia/Tokyo",
  "schedule": ["before 9am on Monday"],
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true
    },
    {
      "groupName": "Kotlin",
      "matchPackagePatterns": ["^org.jetbrains.kotlin"]
    },
    {
      "groupName": "Compose",
      "matchPackagePatterns": ["androidx.compose"]
    },
    {
      "groupName": "Firebase",
      "matchPackagePatterns": ["^com.google.firebase"]
    },
    {
      "groupName": "Hilt",
      "matchPackagePatterns": ["^com.google.dagger", "^androidx.hilt"]
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true
  }
}
```

ポイントは以下の通りです。

- **毎週月曜の朝にチェック**: 週1回に絞ることでPRの洪水を防ぐ
- **パッチバージョンは自動マージ**: 低リスクな更新は自動化
- **関連ライブラリをグルーピング**: Compose、Firebase、Hilt等は1つのPRにまとめる
- **脆弱性アラート有効化**: セキュリティ問題は即座に検知

#### AIによる変更履歴分析

Renovateが作成するPRに対して、AIが変更履歴と影響箇所を自動で出力する仕組みも構築予定です。

### ポイント

- **一気にやらない**: 破壊的変更があるライブラリは最後に回す
- **更新量で優先度を判断**: マイナーバージョンの差が小さいものから着手
- **自動化で継続性を担保**: 一度きれいにしても、また古くなっては意味がない

## 3. Performance Monitoring - パフォーマンス劣化を検知する

### 課題

パフォーマンスの問題は、普段の開発では気づきにくいものです。

そして最悪なのは、**ユーザーからの報告で初めて気づく**というパターンです。

- App Storeのレビューで「最近アプリが重い」と書かれる
- Xで「Avvy重くない？」とポストされる
- Play Console Vitalsでジャンク率上昇を発見するも、いつから・何が原因かわからない

ユーザーに指摘されてから対応するのでは遅すぎます。**開発者が自分たちで気づける体制**を整える必要があります。

### 解決策

包括的なパフォーマンスモニタリング基盤を構築しています。

**Phase 1: Foundation**
- Baseline Profiles（AOT最適化）
- Macrobenchmarkモジュール（CI計測用）
- Looker Studioダッシュボードの検証

**Phase 2: Expanded Observability**
- Firebase Performance traces（重要なユーザーフロー）
- JankStats（Composeのフレームドロップ検出）

**Phase 3: CI Integration**
- GitHub ActionsにMacrobenchmarkを統合
- エミュレータでリグレッション検出

**Phase 4: Dashboard & Scoring**
- Looker StudioダッシュボードをNotionに埋め込み
- パフォーマンススコアリングシステム

### 現在の状況

Firebase Performanceの導入が完了し、以降のPhaseを順次進めています。

### ポイント

- **計測できないものは改善できない**: まずは可視化から
- **CIに組み込む**: リリース前にリグレッションを検出
- **スコアリング**: 数値化することでチーム全体の意識が変わる

## 4. Auto Crash Fix - AIがクラッシュを自動修正

### 課題

少数精鋭のチームでは、機能開発に集中することが多い。Firebase Crashlyticsにクラッシュレポートが届いても、発生頻度が低ければ優先度を上げにくいのが現実です。

しかし、クラッシュの修正は実際にはNullチェックの追加や例外ハンドリングなど、**軽微な修正で済むことが多い**です。

そこで、こういった定型的な修正作業をAIに任せることで、開発者は機能開発に集中しながらも、クラッシュ対応を進められる仕組みを作りました。

### 解決策

Claude Codeを使って、クラッシュの分析から修正PRの作成までを自動化しています。

**仕組み:**

```
毎週月曜 09:00 JST (GitHub Actions)
    ↓
Firebase Crashlyticsからトップクラッシュを取得
    ↓
Claude Codeが分析
    ↓
修正PRを自動生成
    ↓
LinearにIssue作成して追跡
```

**ワークフローの概要:**

```yaml
name: Weekly Crashlytics Auto Fix
on:
  schedule:
    - cron: '0 0 * * 1'  # 毎週月曜 00:00 UTC
  workflow_dispatch:
    inputs:
      dry_run:
        description: 'Dry run mode'
        type: boolean
        default: true
```

### プロンプト設計の工夫

AIに渡すプロンプトでは、以下の点を工夫しています。

**1. 既存Issueの重複チェック**

同じクラッシュに対してIssueが重複作成されないよう、CrashlyticsのIssue IDでLinearを検索してから作成します。

**2. 最小限の修正に絞る**

プロンプトで「Minimal changes only - Fix the crash, don't refactor surrounding code」と明示し、AIが過剰な修正をしないよう制御しています。

**3. ビルド・テスト通過を必須に**

PRを作成する前に`./gradlew assembleDebug`と`./gradlew testDebugUnitTest`の通過を必須としています。ビルドが通らない場合はPR作成をスキップします。

**4. 不確実な修正は人間に委ねる**

「If unsure about the fix, create the PR with a clear note asking for guidance」として、AIが確信を持てない修正は人間にレビューを求めるようにしています。

### ポイント

- **Dry runモード**: いきなり本番適用しない
- **人間レビュー必須**: AIが作ったPRは必ずレビューする
- **Linear連携**: 追跡可能性を担保
- **ビルド・テスト必須**: CIが通らないPRは作成しない

## 共通する考え方

これら4つの施策に共通するのは、以下の考え方です。

### 1. 「人間は忘れる・見落とす」を前提に設計する

どんなに優秀なエンジニアでも、忙しいときは品質チェックを忘れます。だからこそ、**忘れても大丈夫な仕組み**を作ることが重要です。

### 2. CI/自動化に組み込んで継続性を担保する

一度きれいにしても、仕組みがなければまた同じ状態に戻ります。GitHub Actionsに組み込むことで、**継続的に品質を監視**できます。

### 3. 開発速度を落とさない

品質管理のために開発が止まっては本末転倒です。

- 全画面ではなく重要な画面だけテスト
- リスクベースで段階的にアップデート
- AIに単純作業を任せる

### 4. AIを活用して人間の負担を減らす

AIは完璧ではありませんが、単純作業や分析の初手を任せることで、人間はより重要な判断に集中できます。

## まとめ

大事なのは、**「人間が忘れても品質が保たれる仕組み」**を作ることです。

今回紹介した4つの施策は、すべて「自動化」がキーワードです。

1. Screenshot Testing → UIの変化を自動検出
2. Library Update管理 → 更新を自動化
3. Performance Monitoring → 劣化を自動検知
4. Auto Crash Fix → 修正を自動提案

### 品質チェックのタイミング

これらの施策により、Avvyでは以下のタイミングで品質チェックが自動実行されます。

| タイミング | 実行内容 |
|-----------|---------|
| PR作成時 | 単体テスト、UIテスト（Compose + Robolectric）、Screenshot Test |
| 毎週月曜 | Renovate（ライブラリ更新PR）、Auto Crash Fix（クラッシュ修正PR） |
| 常時 | Firebase Performance（パフォーマンス計測） |

AvvyはフルCompose構成のため、UIテストもRobolectricで高速に実行できています。エミュレータ不要でCIが軽量に回るのは大きなメリットです。

開発者が意識しなくても、品質チェックが継続的に回り続ける仕組みです。

### GitHub Actions + Claude Codeの組み合わせが強力

今回の取り組みを通じて感じたのは、**GitHub ActionsとClaude Codeの組み合わせが非常に強力**だということです。

- **GitHub Actions**: スケジュール実行、トリガー管理、シークレット管理、PR作成
- **Claude Code**: コード分析、修正提案、Issue作成、コンテキストを理解した判断

GitHub Actionsが「いつ・何を実行するか」を担い、Claude Codeが「どう分析・修正するか」を担う。この役割分担により、従来は人間がやるしかなかった知的作業の一部を自動化できるようになりました。

まだ進行中の施策もありますが、少しずつ自動化を進めていくことで、品質の高いアプリを継続的に提供できると信じています。
