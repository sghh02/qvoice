# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

これは「qvoice」という名前のFlutterアプリケーションで、請求書管理システムです。Android、iOS、Webプラットフォームをサポートする標準的なFlutterプロジェクト構造に従っています。
一次リリースではWebのみを対応予定

## 開発コマンド

### アプリの実行
```bash
flutter run                    # 接続されたデバイス/エミュレータで実行
flutter run -d chrome         # Webブラウザで実行
flutter run -d android        # Androidデバイス/エミュレータで実行
flutter run -d ios            # iOSデバイス/シミュレータで実行
```

### テスト
```bash
flutter test                   # 全テストを実行
flutter test test/widget_test.dart  # 特定のテストファイルを実行
```

### コード品質
```bash
flutter analyze               # 静的解析とリンティング
flutter format .              # 全Dartファイルをフォーマット
```

### ビルド
```bash
flutter build apk            # Android APKをビルド
flutter build ios            # iOSアプリをビルド
flutter build web            # Webアプリをビルド
flutter clean                # ビルド成果物をクリーン
flutter pub get              # 依存関係を取得
```

## プロジェクト構造

- `lib/main.dart` - 基本的なカウンターアプリテンプレートのエントリーポイント
- `test/` - ウィジェットテストとユニットテスト
- `android/` - Android固有の設定とネイティブコード
- `ios/` - iOS固有の設定とネイティブコード
- `web/` - Webプラットフォームのアセットと設定
- `pubspec.yaml` - 依存関係とプロジェクト設定
- `analysis_options.yaml` - flutter_lintsパッケージを使用したリンティング規則

## 依存関係

プロジェクトは最小限の依存関係を使用しています：
- `cupertino_icons` - iOSスタイルのアイコン用
- `flutter_lints` - コード品質の強化用
- 標準Flutter SDK (^3.7.0)

## アーキテクチャノート

### 状態管理
- **Riverpod**: コード生成を使用したメイン状態管理（`@riverpod`アノテーション）
- **Flutter Riverpod**: プロバイダーベースのリアクティブ状態管理
- **自動生成プロバイダー**: 生成されたプロバイダーには`part`ファイルを使用（例：`router.g.dart`）

### ナビゲーション
- **GoRouter**: プログラマティックナビゲーションを使った宣言的ルーティング
- **トランジションなし**: すべてのページ遷移は`NoTransitionPage`を使用して瞬間的に移動
- **ルート構造**: `lib/routing/router.dart`でルート定義を一元管理

### アプリケーション設計原則

#### 最重要原則（必ず守る）
- **Single Source of Truth (SSOT)**: 同一データの重複管理を避け、一箇所からの参照徹底
- **単方向データフロー**: データの流れを一方向に限定し、追跡可能な状態管理

#### 次点で重要（推奨）
- **immutableプログラミング**: Freezedによる不変データクラス
- **テスト可能性**: Provider経由でのDI、テスト時override対応
- **単一責任の原則**: 各Provider・関数・クラスの責務を明確化

### 機能構成（Feature-first構成）

#### 標準構成（シンプル機能向け）
```
lib/features/[機能名]/
├── models/          # データモデル（統一的、Entity/DTO兼用）
├── providers/       # 状態管理・ビジネスロジック（SSOT実現）
├── pages/           # ページウィジェット
└── widgets/         # 機能固有の再利用ウィジェット
```

#### 拡張構成（複雑機能向け）
```
lib/features/[機能名]/
├── data/
│   └── repositories/       # 外部データソース実装（Firebase等）
├── domain/
│   └── models/             # 統一データモデル（Freezed）
└── presentation/
    ├── pages/              # ページウィジェット
    ├── widgets/            # 機能固有ウィジェット
    └── providers/          # 状態管理・ビジネスロジック
```

#### 共通コンポーネント
```
lib/core/
├── config/          # アプリ設定
├── models/          # 全機能で共通のモデル
├── providers/       # 全機能で共通のプロバイダー
├── utils/           # ユーティリティ関数
└── validators/      # 入力バリデーション（全機能共通）
```

### 設計方針（小規模アプリ向け）
- **過度な抽象化を避ける**: 不要な中間層（datasource、usecase、repository interface）は排除
- **Provider中心設計**: StateNotifierは最小限、FutureProvider/StreamProviderを活用
- **モデル統一**: 複数レイヤー間でのデータ変換処理を排除（Entity/DTO/Model分離しない）
- **機能凝集**: 関連するProvider・Widget・Modelを近い場所に配置
- **構成選択**: 機能の複雑度に応じて標準構成と拡張構成を使い分け

### マスターデータ同期
- **バックグラウンド同期**: マスターデータ（カテゴリ、質問）がアプリ開始時に自動同期
- **バージョン管理**: 互換性確認のためメジャーバージョンチェックを使用
- **ローカルフォールバック**: リモートデータが利用できない場合はアセットがフォールバック
- **認証制限**: セカンドリリースで認証済みユーザーのみに変更予定

### マルチ環境サポート
- **フレーバー**: dart-defineファイルによるdev/prod設定
- **Firebase**: 環境ごとに別プロジェクト、自動設定選択
- **アプリID**: フレーバーごとに異なるバンドル識別子

### 国際化
- **flutter_localizations**: Flutter組み込みi18nサポート
- **ARBファイル**: `lib/l10n/`のen、ja、es、ko翻訳ファイル
- **多言語エンティティ**: ローカライズされたコンテンツ用のカスタム`MultiLanguageText`型

### UIフレームワーク
- **Material Design**: メインデザインシステム
- **カスタムテーマ**: `constants/theme.dart`でテーマ設定を一元管理
- **日本語タイポグラフィ**: NotoSansJPフォントファミリーを含む
- **グラスモーフィズム**: モダンなUI用のカスタムグラスカードコンポーネント

## 主要な依存関係

### 状態管理・アーキテクチャ
- `flutter_riverpod`: 状態管理
- `riverpod_annotation`: コード生成アノテーション
- `freezed`: 不変データクラス
- `json_annotation`: JSONシリアライゼーション

### ナビゲーション・UI
- `go_router`: ナビゲーション
- `flutter_localizations`: 国際化
- `gap`: レスポンシブスペーシング
- `auto_size_text`: テキスト自動サイズ調整

### Firebase・データ
- `firebase_core`: Firebase初期化
- `cloud_firestore`: リモートデータストレージ
- `firebase_auth`: 認証（セカンドリリース対応）
- `firebase_ai`: AI機能（セカンドリリース対応）
- `shared_preferences`: ローカル永続化

### 認証・セキュリティ（セカンドリリース）
- `firebase_auth`: Firebase認証
- `google_sign_in`: Google認証
- `sign_in_with_apple`: Apple認証（iOS）
- `flutter_secure_storage`: 認証トークン・機密情報の安全な保存

### 開発支援
- `build_runner`: コード生成
- `flutter_launcher_icons`: アプリアイコン生成
- `mockito`: テスト用モック

## テスト戦略

- `mockito`: ユニットテスト用モックオブジェクト
- テストファイルはlib/構造をtest/でミラーする
- Flutterの組み込みテストフレームワークを使用したウィジェットテスト

## 開発ガイドライン

### ウィジェット開発ルール
- **基本パターン**: 新しいウィジェットには`ConsumerWidget`を使用
- **状態管理**: StatefulWidgetよりRiverpodプロバイダーを優先
- **StatefulWidget使用**: アニメーション、ライフサイクル管理、TextEditingControllerの場合のみ
- **避けるべき**: ビジネスロジックやデータ状態でのStatefulWidget

### 推奨パターン

#### Widget実装パターン
```dart
// ✅ 推奨: ConsumerWidget（基本パターン）
class ArticleListView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final articles = ref.watch(articlesProvider);
    return ListView(
      children: articles.map((article) => Text(article.title)).toList(),
    );
  }
}

// ✅ データ管理: FutureProvider/StreamProvider（SSOTの実現）
final articlesProvider = StreamProvider<List<Article>>((ref) {
  return FirebaseFirestore.instance
    .collection('articles')
    .snapshots()
    .map((snap) => snap.docs.map((doc) => Article.fromFirestore(doc)).toList());
});

// ✅ 操作: Provider経由の関数（単方向データフロー）
final articleServiceProvider = Provider((ref) => ArticleService(ref));

class ArticleService {
  ArticleService(this._ref);
  final Ref _ref;
  
  Future<void> likeArticle(String id) async {
    await FirebaseFirestore.instance.collection('articles').doc(id).update({
      'isLiked': true,
    });
    // SSOTからの自動更新により、UI側は自動で最新状態になる
  }
}

// ❌ 避けるべき: 不必要なStateNotifier
// 編集画面など、真に必要な場合のみ使用
```

#### テスト対応パターン
```dart
// Provider経由で提供することで、テスト時のoverride対応
test('記事をいいねできる', () async {
  final container = ProviderContainer(
    overrides: [
      articleServiceProvider.overrideWith((ref) => FakeArticleService()),
    ],
  );
  
  final service = container.read(articleServiceProvider);
  await service.likeArticle('article1');
  // テスト用実装で期待通りの動作確認
});
```

### エラーハンドリング
- 重要でない操作（マスターデータ同期など）はサイレントバックグラウンド処理
- ユーザー向けエラーは適切なフィードバックを表示
- 例外が予期される場合のみtry-catchを使用

### プロジェクト固有の命名規則
- ページ: `*_page.dart`（例：`home_page.dart`）
- プロバイダー: `*_provider.dart`または`*_providers.dart`
- エンティティ: `.freezed.dart`と`.g.dart`パーツを持つFreezedクラスを使用
- 機能: アプリ機能に合致する説明的な名前を使用
- リポジトリ実装: `*_repository_impl.dart`（拡張構成の場合）
- バリデーター: `*_validator.dart`（`core/validators/`配下）

## ブランチ戦略

### メインブランチ
- **develop**: 最新の開発版（developブランチから新機能ブランチを作成）
- **main**: 本番リリース版

### 新機能開発の流れ
1. `develop`ブランチから新しいfeatureブランチを作成
```bash
git checkout develop
git pull origin develop
git checkout -b feature/your-feature-name
```

2. 機能実装・テスト完了後、`develop`ブランチへのPR作成
3. コードレビュー・マージ後、必要に応じて`main`ブランチへリリース

### ブランチ命名規則
- 機能開発: `feature/機能名`
- バグ修正: `fix/修正内容`
- リファクタリング: `refactor/対象範囲`
- ドキュメント: `docs/更新内容`