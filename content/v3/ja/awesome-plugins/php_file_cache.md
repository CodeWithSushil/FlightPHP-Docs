# flightphp/cache

軽量でシンプルなスタンドアロン PHP ファイル内キャッシュクラス [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache) からフォーク

**利点** 
- 軽量、スタンドアロン、シンプル
- すべてのコードが1つのファイルに - 無意味なドライバーはありません。
- セキュア - 生成されたすべてのキャッシュファイルには die 付きの php ヘッダーがあり、誰かがパスを知っていても、サーバーが適切に設定されていない場合でも直接アクセスは不可能
- 十分にドキュメント化され、テスト済み
- flock を通じて同時実行を正しく処理
- PHP 7.4+ をサポート
- MIT ライセンスで無料

このドキュメントサイトはこのライブラリを使用して各ページをキャッシュしています！

コードを表示するには[こちら](https://github.com/flightphp/cache)をクリックしてください。

## インストール

composer 経由でインストール：

```bash
composer require flightphp/cache
```

## 使用方法

使用方法は非常に簡単です。これによりキャッシュディレクトリにキャッシュファイルが保存されます。

```php
use flight\Cache;

$app = Flight::app();

// コンストラクタにキャッシュを保存するディレクトリを渡します
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// これにより、キャッシュが本番モードの場合にのみ使用されることが保証されます
	// ENVIRONMENT はブートストラップファイルまたはアプリ内の他の場所で設定される定数です
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### キャッシュ値の取得

`get()` メソッドを使用してキャッシュされた値を取得します。期限切れの場合にキャッシュを更新する便利なメソッドが必要な場合は、`refreshIfExpired()` を使用できます。

```php

// キャッシュインスタンスを取得
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // キャッシュするデータを返す
}, 10); // 10秒

// または
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10秒
}
```

### キャッシュ値の保存

`set()` メソッドを使用してキャッシュに値を保存します。

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10秒
```

### キャッシュ値の削除

`delete()` メソッドを使用してキャッシュ内の値を削除します。

```php
Flight::cache()->delete('simple-cache-test');
```

### キャッシュ値の存在確認

`exists()` メソッドを使用してキャッシュに値が存在するかどうかを確認します。

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// 何かを実行
}
```

### キャッシュのクリア
`flush()` メソッドを使用してキャッシュ全体をクリアします。

```php
Flight::cache()->flush();
```

### キャッシュからメタデータを取得

キャッシュエントリに関するタイムスタンプやその他のメタデータを取得したい場合は、正しいパラメータとして `true` を渡すようにしてください。

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // キャッシュするデータを返す
}, 10, true); // true = メタデータ付きで返す
// または
$data = $cache->get("simple-cache-meta-test", true); // true = メタデータ付きで返す

/*
メタデータ付きで取得したキャッシュアイテムの例：
{
    "time":1511667506, <-- 保存された unix タイムスタンプ
    "expire":10,       <-- 秒単位の有効期限
    "data":"04:38:26", <-- 非シリアライズデータ
    "permanent":false
}

メタデータを使用することで、例えばアイテムが保存された時刻や有効期限を計算することができます
"data" キーでデータ自体にアクセスすることもできます
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // データが期限切れになる unix タイムスタンプを取得し、そこから現在のタイムスタンプを減算
$cacheddate = $data["data"]; // "data" キーでデータ自体にアクセス

echo "最新のキャッシュ保存: $cacheddate, $expiresin 秒後に期限切れ";
```

## ソースコード

コードを表示するには [https://github.com/flightphp/cache](https://github.com/flightphp/cache) をご覧ください。