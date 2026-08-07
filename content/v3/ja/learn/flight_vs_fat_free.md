# FlightとFat-Freeの比較

## Fat-Freeとは？
[Fat-Free](https://fatfreeframework.com)（通称 **F3**）は、動的で堅牢なWebアプリケーションを迅速に構築するために設計された、強力でありながら使いやすいPHPマイクロフレームワークです。

Flightは、多くの点でFat-Freeと比較でき、おそらく機能とシンプルさの面で最も近い親戚です。Fat-FreeにはFlightにはない機能が多くありますが、Flightにある機能も多く持っています。Fat-Freeは時代を感じさせ始めており、かつてほど人気はありません。

アップデートの頻度は減り、コミュニティも以前ほど活発ではありません。コード自体はシンプルですが、構文の規律に欠けるため、読み解くのが難しいこともあります。PHP 8.3では動作しますが、コード自体はまだPHP 5.3時代のものに見えます。

## Flightと比較した場合の利点

- Fat-FreeはGitHubでFlightよりも少し多くのスターを獲得しています。
- Fat-Freeにはそれなりのドキュメントがありますが、明確さに欠ける部分がいくつかあります。
- Fat-Freeには、フレームワークを学ぶために利用できるYouTubeチュートリアルやオンライン記事などの資料がいくつかあります。
- Fat-Freeには、役立つことがある[プラグイン](https://fatfreeframework.com/3.8/api-reference)が組み込まれています。
- Fat-Freeには、データベースと対話するために使用できるMapperと呼ばれるORMが組み込まれています。Flightには[active-record](/awesome-plugins/active-record)があります。
- Fat-Freeには、セッション、キャッシュ、ローカライゼーションが組み込まれています。Flightではサードパーティライブラリが必要ですが、[ドキュメント](/awesome-plugins)でカバーされています。
- Fat-Freeには、フレームワークを拡張するために使用できる[コミュニティ作成プラグイン](https://fatfreeframework.com/3.8/development#Community)の小さなグループがあります。Flightには[ドキュメント](/awesome-plugins)と[例](/examples)のページでカバーされているものがあります。
- Fat-FreeはFlightと同様に依存関係がありません。
- Fat-FreeはFlightと同様に、開発者にアプリケーションの制御を委ね、シンプルな開発者体験を提供することに重点を置いています。
- Fat-FreeはFlightと同様に後方互換性を維持しています（部分的にはアップデートが[頻度が減っている](https://github.com/bcosca/fatfree/releases)ためです）。
- Fat-FreeはFlightと同様に、初めてフレームワークの世界に足を踏み入れる開発者を対象としています。
- Fat-Freeには、Flightのテンプレートエンジンよりも堅牢な組み込みテンプレートエンジンがあります。Flightではこれを実現するために[Latte](/awesome-plugins/latte)を推奨しています。
- Fat-Freeには独自のCLIタイプの「ルート」コマンドがあり、Fat-Free自体の中でCLIアプリケーションを構築し、それを`GET`リクエストのように扱うことができます。Flightはこれを[runway](/awesome-plugins/runway)で実現しています。

## Flightと比較した場合の欠点

- Fat-Freeにはいくつかの実装テストがあり、非常に基本的な独自の[テスト](https://fatfreeframework.com/3.8/test)クラスもあります。しかし、Flightのように100%ユニットテストされているわけではありません。
- ドキュメントサイトを実際に検索するには、Googleなどの検索エンジンを使う必要があります。
- Flightのドキュメントサイトにはダークモードがあります。（マイクドロップ）
- Fat-Freeには、残念ながらメンテナンスされていないモジュールがいくつかあります。
- Flightにはデータベースアクセスのための[SimplePdo](/learn/simple-pdo)があり、Fat-Freeの組み込み`DB\SQL`クラスよりも少しシンプルです（そして非推奨のPdoWrapperよりも推奨されます）。
- Flightには、アプリケーションを保護するために使用できる[権限プラグイン](/awesome-plugins/permissions)があります。Fat-Freeではサードパーティライブラリを使用する必要があります。
- Flightには[active-record](/awesome-plugins/active-record)と呼ばれるORMがあり、Fat-FreeのMapperよりもORMらしく感じられます。
  `active-record`の追加の利点は、レコード間のリレーションシップを定義して自動的に結合できることです。一方、Fat-FreeのMapperでは
  [SQLビュー](https://fatfreeframework.com/3.8/databases#ProsandCons)を作成する必要があります。
- 驚くべきことに、Fat-Freeにはルート名前空間がありません。Flightは独自のコードと衝突しないように、完全に名前空間が設定されています。
  ここで最大の問題は`Cache`クラスです。
- Fat-Freeにはミドルウェアがありません。代わりに、コントローラーでリクエストとレスポンスをフィルタリングするために使用できる`beforeroute`と`afterroute`フックがあります。
- Fat-Freeはルートをグループ化できません。
- Fat-Freeには依存性注入コンテナのハンドラーがありますが、その使用方法に関するドキュメントは非常に乏しいです。
- 基本的にすべてが[`HIVE`](https://fatfreeframework.com/3.8/quick-reference)と呼ばれるものに格納されるため、デバッグが少し難しくなることがあります。