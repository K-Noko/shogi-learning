# 設計書: フロントエンド

Walking Skeleton（Phase1）の時点で決まっていることだけを書く。設計書の読み方は[`overview.md`](overview.md)を参照。外した案・経緯・長い論証は[`rationale/frontend.md`](rationale/frontend.md)に、同じ見出しで置く。

フロントエンドは、サーバーが配信する状態を描画し、ユーザーの意図を送るだけの薄いクライアントに徹する（[`overview.md`](overview.md)「全体のアーキテクチャ」）。

## 技術

| 用途 | 技術 |
|---|---|
| UIの土台・ビルド | React＋Vite |
| ルーティング | React Router（フレームワークのモードは使わない） |
| 盤の描画 | Pixi.js |
| 境界の検証 | Zod（Phase3まではレスポンスのスキーマを`z.strictObject`にする。[`api.md`](api.md)「レスポンスへのフィールド追加と後方互換性」） |

- 盤以外のUI（ボタン等）はReactで作り、Reactの画面の中にPixi.jsの描画面を置く
- React Routerは、ルーティングのライブラリとして使うモード（Declarative・Data）に限る。サーバーでの描画を含むフレームワークのモードは、FastAPIとは別の2つ目のサーバーを生むので使わない。DeclarativeとDataのどちらにするかはWalking SkeletonのIssueで決める

外した案: [`rationale/frontend.md`](rationale/frontend.md)「技術」

## 画面のURL

- 対局の画面のURLに対局IDを入れる（例: `/games/{id}`）。再読み込みしたら、そのIDで`GET /games/{id}`を呼んで同じ対局に戻る（[`api.md`](api.md)「Phase1のエンドポイント」）
- 対局の状態は対局IDに紐づけ、IDが変わったら作り直す（Reactなら`key={gameId}`等。[`api.md`](api.md)「新規対局とリセット」）

## フレームワークに依存させない部分

- **Pixi.jsの描画のコードをReactに依存させない**。Reactは描画面を載せ、状態を渡すだけにする。依存を内側（フレームワークに依存しない部分）へ向ける、という規則をフロントエンドに当てはめたもの。後でUIの土台を別のフレームワーク（Svelte等）に作り替えるとき、盤の描画をそのまま持っていけるようにするため
- 候補（Walking SkeletonのIssueで決める）
  - APIの呼び出しとZodのスキーマも、Reactに依存しないモジュールにする
  - ルーティングの定義は薄く保ち、URLから対局IDを取り出して渡すことだけをさせる

詳しい理由（作り替えのときに何が持っていけるか）: [`rationale/frontend.md`](rationale/frontend.md)「フレームワークに依存させない部分」

## 未決の事項

Walking SkeletonのIssueで決める。

- 状態管理の方法（Reactの標準の仕組みで足りるか）
- React RouterのDeclarativeとDataのどちらのモードにするか
- フロントエンドからバックエンドへの接続を、Viteのプロキシにするか、CORSにするか（[`overview.md`](overview.md)「ポート」）
