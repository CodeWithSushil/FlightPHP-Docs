# flightphp/cache

軽量でシンプル、スタンドアロンのPHPファイル内キャッシュクラス。[Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)からフォーク

**利点**
- 軽量でスタンドアロン、シンプル
- すべてのコードが1つのファイル - 無意味なドライバーはありません。
- セキュア - 生成されたすべてのキャッシュファイルにはPHPヘッダーが付いており、dieを使用するため、パスを知っていても直接アクセスは不可能で、サーバーが適切に設定されていない場合でも安全
- ドキュメントが充実しており、テスト済み
- flockを通じて同時実行を正しく処理
- PHP 7.4+をサポート
- MITライセンスで無料

このドキュメントサイトは、各ページをキャッシュするためにこのライブラリを使用しています！

コードを表示するには[こちら](https://github.com/flightphp/cache)をクリックしてください。

## インストール

composer経由でインストール:

```bash
composer require flightphp/cache
```

## 使用方法

使用方法は非常に簡単です。これにより、キャッシュディレクトリにキャッシュファイルが保存されます。

```php
use flight\Cache;

$app = Flight::app();

// キャッシュが保存されるディレクトリをコンストラクタに渡します
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// キャッシュが本番モードでのみ使用されることを保証します
	// ENVIRONMENTはbootstrapファイルまたはアプリ内の他の場所で設定される定数です
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### キャッシュ値の取得

`get()`メソッドを使用してキャッシュされた値を取得します。期限切れの場合にキャッシュを更新する便利なメソッドが必要な場合は、`refreshIfExpired()`を使用できます。

```php

// キャッシュインスタンスを取得
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // キャッシュするデータを返します
}, 10); // 10秒

// または
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10秒
}
```

### キャッシュ値の保存

`set()`メソッドを使用してキャッシュに値を保存します。

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10秒
```

### キャッシュ値の削除

`delete()`メソッドを使用してキャッシュ内の値を削除します。

```php
Flight::cache()->delete('simple-cache-test');
```

### キャッシュ値の存在確認

`exists()`メソッドを使用してキャッシュ内に値が存在するかを確認します。

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// 何かを実行
}
```

### キャッシュのクリア
`flush()`メソッドを使用してキャッシュ全体をクリアします。

```php
Flight::cache()->flush();
```

### キャッシュからメタデータを取得

キャッシュエントリのタイムスタンプやその他のメタデータを取得したい場合は、正しいパラメータとして`true`を渡してください。

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // キャッシュするデータを返します
}, 10, true); // true = メタデータ付きで返す
// または
$data = $cache->get("simple-cache-meta-test", true); // true = メタデータ付きで返す

/*
メタデータ付きで取得したキャッシュアイテムの例:
{
    "time":1511667506, <-- 保存されたunixタイムスタンプ
    "expire":10,       <-- 秒単位の有効期限
    "data":"04:38:26", <-- デシリアライズされたデータ
    "permanent":false
}

メタデータを使用することで、例えばアイテムが保存された時間や有効期限を計算できます
"data"キーでデータ自体にアクセスすることもできます
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // データの有効期限を示すunixタイムスタンプを取得し、そこから現在のタイムスタンプを引きます
$cacheddate = $data["data"]; // "data"キーでデータ自体にアクセスします

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## ソースコード

コードを表示するには[https://github.com/flightphp/cache](https://github.com/flightphp/cache)をご覧ください。