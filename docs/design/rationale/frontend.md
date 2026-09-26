# フロントエンド: 外した案・経緯

[`../frontend.md`](../frontend.md)の決定について、検討して外した案・経緯・長い論証を、本体と同じ見出しで置く。決定の文言だけでは実装が自明に決まらないときに読む。

## 技術

- **Next.js**: ルーティングのためだけに使うには、抱え込むものが多すぎる
  - Next.jsの価値の中心は、サーバーでの描画・React Server Components・Server Actions等のサーバー側の機能にある。このプロジェクトはサーバーの役割をすべてFastAPIに置くので、その価値を使わない
  - 使うと、Node.jsの2つ目のサーバーがFastAPIとクライアントの間に入り、「真実の源はバックエンド」の「バックエンド」が2つになる。サーバーを持たない静的なサイトとして書き出すモード（static export）もあるが、`generateStaticParams()`なしの動的なルートに対応しないので、`/games/{id}`のようにIDが実行時に決まるURLと相性が悪い
  - 使わない機能も、攻撃される面としては抱え込む。実例として、2025年12月にReact Server Componentsの脆弱性（CVE-2025-55182、通称React2Shell。CVSS 10.0。認証なしでサーバー上の任意のコードを実行できる）が公表され、公表から数時間で実際の攻撃に使われた。Next.jsも影響を受けた（CVE-2025-66478として追跡）
- **React Routerのフレームワークのモード**: サーバーでの描画を含むので、Next.jsと同じ理由で使わない
- **TanStack Router**: URLの引数まで型で扱えるのが特長の、React Routerの代わりになるルーティングのライブラリ。検討の対象には入るが、今回はより標準的なReact Routerを採った
- **Svelte**: 今は選ばない。ポートフォリオとして今のフェーズで作りこみたい箇所はバックエンド・テストエンジニアリングに大きく寄っている。作り替えたくなるのはフロントエンドの性能を作り込むとき。作り替えを安くするための構造は「フレームワークに依存させない部分」で先に取っておく
  - Svelteに作り替える場合も、SvelteKitは同じくサーバーでの描画を含むフレームワークなので、Next.jsを外したのと同じ問いがもう一度立つ（SvelteKitはサーバーでの描画を切ってSPAとして動かすモードを持つ）

## フレームワークに依存させない部分

作り替えのときに、何を持っていけるか。

| 資産 | Svelteに作り替える場合 | Unityクライアントを作る場合 |
|---|---|---|
| APIの契約（サーバー側） | そのまま | そのまま。Unityクライアントを足せることが、「真実の源はバックエンド」「薄いクライアント」を選んだ理由 |
| Pixi.jsの描画のコード | Reactに依存させていなければ、そのまま持っていける | 持っていけない（言語がC#） |
| APIの呼び出し・Zodのスキーマ | Reactに依存させていなければ、そのまま持っていける | 持っていけない。ただしFastAPIが生成するOpenAPIの定義から、C#の型を生成する道がある |
| ルーティングのコード | 書き直し（SvelteKitはファイルの置き場所でルートを決める方式で、書き方がまったく違う） | 書き直し。そもそもUnityにはURLのルーティングがなく、シーンの切り替えになる |
| ルーティングの設計（どのURLで何を表すか） | そのまま（`/games/{id}`という形は、ブックマーク等で利用者が依存する約束で、ライブラリとは独立している） | 対応する概念（「対局IDで対局の画面を開く」）として持っていける |
| Reactのコンポーネント・状態管理 | 書き直し | 書き直し |

- ルーティングは、コードは移行できず、設計（URLの形）だけが移行できる。なので、ルーティングの定義に処理を載せず、URLから対局IDを取り出して、フレームワークに依存しない部分へ渡すだけにしておくと、書き直す量が最小になる
- Unityクライアントで持っていけるのは、サーバー側とAPIの契約だけ。フロントエンドのコードの構造は、Webの中でのフレームワークの作り替えにしか効かない

出典:

- [Datadog Security Labs: CVE-2025-55182 (React2Shell)](https://securitylabs.datadoghq.com/articles/cve-2025-55182-react2shell-remote-code-execution-react-server-components/)
- [Next.js: Static Exports](https://nextjs.org/docs/app/guides/static-exports)
- [React Router: Picking a Mode](https://reactrouter.com/start/modes)
- [SvelteKit: Single-page apps](https://svelte.dev/docs/kit/single-page-apps)
