# Gカード・ゲーム版の独立運用セットアップ

**作業先は新しい `g-card-game` だけです。元の `g-card` リポジトリ、元のGカード用スプレッドシート・Apps Script・Firebaseの設定は変更しません。**

## 1. 新しい学校用データとApps Script

1. 作成済みの「Gカード・ゲーム版」専用の空シートを使います。URLは学校側で管理し、公開リポジトリには書きません。共有は制限付きのままにします。元のGカードのシートをコピーすると個人情報と運用記録まで複製されるため、空のシートを使います。
2. **その新しいシート**の「拡張機能 → Apps Script」を開き、このリポジトリの `gas/Code.gs`、`gas/Learning.gs`、`gas/Bridge.html` を同名ファイルとして登録します。元のApps Script編集画面には貼り付けません。
3. まず `configureGameInstance()` を実行します。新しい空シートに紐づくプロジェクトだけで成功し、保存先をスクリプトプロパティに記録します。続けて `setup()` を実行し、各表を作ります。`Settings` の `schoolDomain` に学校メールの `@` 以降、`adminEmails` に先生の学校メールを設定します。Apps Scriptのスクリプトプロパティ `GOOGLE_CLIENT_ID` に、ゲーム版用のウェブアプリクライアントIDを登録します。`SESSION_SECRET` は `setup()` が作成します。
4. Apps Scriptを「ウェブアプリ」として**新規デプロイ**します。実行ユーザーは自分、アクセスは全員。新しく発行された `/exec` URLを控えます。元のデプロイのバージョン更新はしません。
5. 新しい `Users` シートに名簿を登録します。ログインにはメールアドレスと氏名が必要です。管理者画面からCSVで取り込むこともできます。

## 2. Googleログイン

Google Cloudでゲーム版用のウェブアプリクライアントIDを作成します。学校の設定で選べる場合は対象ユーザーを内部にします。GitHub Pagesで使う場合の承認済みJavaScript生成元は `https://ken-ui-web.github.io` です。クライアントIDを新しいApps Scriptの `GOOGLE_CLIENT_ID` と、このアプリの `VITE_GOOGLE_CLIENT_ID` に設定します。クライアントシークレットは使いません。

## 3. オンライン対戦

ゲーム版用のFirebaseプロジェクトを新しく作成します。Authenticationで匿名認証を有効にし、Realtime Databaseをロックモードで作成します。ルールにはこのリポジトリの [database.rules.json](../firebase/database.rules.json) を登録します。ウェブアプリの `firebaseConfig` に `databaseURL` を含め、`VITE_FIREBASE_CONFIG` にJSONとして設定します。元のFirebaseプロジェクトは使用しません。

## 4. 新しいGitHub Pages

新しい `g-card-game` リポジトリを作成し、このフォルダーの内容を送ります。Pagesの公開元はGitHub Actionsにします。新しいリポジトリのActions変数だけに以下を登録します。

| 変数 | 値 |
| --- | --- |
| `VITE_GAS_URL` | 手順1で発行したゲーム版専用 `/exec` URL |
| `VITE_GOOGLE_CLIENT_ID` | 手順2のゲーム版用クライアントID |
| `VITE_FIREBASE_CONFIG` | 手順3のゲーム版用Firebase設定JSON |

想定URLは `https://ken-ui-web.github.io/g-card-game/#/home` です。元のリポジトリのActions変数やPages設定を変更しないでください。

## 5. 公開前確認

ゲーム版の管理者でログインし、元のGカードと異なる初期残高・所持カードになっていることを確認します。テストの作成・受験、カード購入、筋トレ、CPU戦、ルームコード対戦を確認します。元のGカードの `Users`・`PointLog`・`OwnedCards` に新しい行が増えていないことを確認します。

ゲーム版は元のGASへの接続をクライアント側で拒否し、専用GASを識別してから通信します。専用GASも元のスプレッドシートIDを拒否します。
