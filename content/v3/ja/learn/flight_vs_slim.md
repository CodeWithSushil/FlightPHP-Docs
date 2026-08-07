# Flight vs Slim

## Slimとは？
[Slim](https://slimframework.com) は、シンプルでありながら強力なWebアプリケーションやAPIをすばやく作成するのに役立つPHPマイクロフレームワークです。

Flight の v3 機能の多くは、実際には Slim から着想を得ています。ルートのグループ化と、特定の順序でミドルウェアを実行することは、Slim に触発された2つの機能です。Slim v3 はシンプルさを重視して登場しましたが、v4 については[賛否両論](https://github.com/slimphp/Slim/issues/2770)があります。

## Flight と比べた長所

- Slim は開発者のコミュニティが大きいため、車輪の再発明を避けるのに役立つ便利なモジュールが多数あります。
- Slim は PHP コミュニティで一般的な多くのインターフェースと標準に準拠しており、相互運用性が高まります。
- Slim には、フレームワークを学習するために使用できる十分なドキュメントとチュートリアルがあります（ただし、Laravel や Symfony には及びません）。
- Slim には、フレームワークを学習するために使用できる YouTube チュートリアルやオンライン記事などのさまざまなリソースがあります。
- Slim は PSR-7 準拠であるため、コアルーティング機能を処理するために任意のコンポーネントを使用できます。

## Flight と比べた短所

- 驚くべきことに、Slim はマイクロフレームワークとして想像するほど高速ではありません。詳細については、[TechEmpower ベンチマーク](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) を参照してください。
- Flight は、軽量で高速かつ使いやすい Web アプリケーションを構築したい開発者を対象としています。
- Flight には依存関係がありませんが、[Slim にはインストールしなければならない依存関係がいくつかあります](https://github.com/slimphp/Slim/blob/4.x/composer.json)。
- Flight はシンプルさと使いやすさを重視しています。
- Flight のコア機能の 1 つは、後方互換性を維持するために最善を尽くすことです。Slim v3 から v4 への移行は破壊的な変更でした。
- Flight は、初めてフレームワークの世界に足を踏み入れる開発者を対象としています。
- Flight はエンタープライズレベルのアプリケーションにも対応できますが、Slim ほど多くの例やチュートリアルはありません。また、物事を整理整頓された構造に保つために、開発者側により多くの規律が求められます。
- Flight は開発者にアプリケーションのより多くの制御を提供しますが、Slim は舞台裏で魔法をこっそりと忍び込ませることができます。
- Flight にはデータベースアクセスのための [SimplePdo](/learn/simple-pdo) があります（非推奨の PdoWrapper よりも推奨されます）。Slim ではサードパーティのライブラリを使用する必要があります。
- Flight には、アプリケーションを保護するために使用できる [permissions プラグイン](/awesome-plugins/permissions) があります。Slim ではサードパーティのライブラリを使用する必要があります。
- Flight には、データベースと対話するために使用できる [active-record](/awesome-plugins/active-record) という ORM があります。Slim ではサードパーティのライブラリを使用する必要があります。
- Flight には、コマンドラインからアプリケーションを実行するために使用できる [runway](/awesome-plugins/runway) という CLI アプリケーションがあります。Slim にはありません。