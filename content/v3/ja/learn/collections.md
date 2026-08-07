# コレクション

## 概要

Flightの`Collection`クラスは、データセットを管理するための便利なユーティリティです。配列表記とオブジェクト表記の両方でデータにアクセス・操作できるため、コードがよりクリーンで柔軟になります。

## 理解

`Collection`は基本的に配列のラッパーですが、いくつかの追加機能があります。配列のように使用したり、ループしたり、アイテム数を数えたり、アイテムをオブジェクトプロパティのようにアクセスしたりできます。これは、アプリ内で構造化データを渡したい場合や、コードを少し読みやすくしたい場合に特に便利です。

コレクションはいくつかのPHPインターフェースを実装しています：
- `ArrayAccess`（配列構文を使用できます）
- `Iterator`（`foreach`でループできます）
- `Countable`（`count()`を使用できます）
- `JsonSerializable`（簡単にJSONに変換できます）

## 基本的な使い方

### コレクションの作成

コンストラクタに配列を渡すだけで、コレクションを作成できます：

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### アイテムへのアクセス

配列表記またはオブジェクト表記のどちらでもアイテムにアクセスできます：

```php
// 配列表記
echo $collection['name']; // 出力: FlightPHP

// オブジェクト表記
echo $collection->version; // 出力: 3
```

存在しないキーにアクセスしようとすると、エラーではなく`null`が返ります。

### アイテムの設定

アイテムの設定も、どちらの表記でも行えます：

```php
// 配列表記
$collection['author'] = 'Mike Cao';

// オブジェクト表記
$collection->license = 'MIT';
```

### アイテムの確認と削除

アイテムが存在するか確認する：

```php
if (isset($collection['name'])) {
  // 何かを行う
}

if (isset($collection->version)) {
  // 何かを行う
}
```

アイテムを削除する：

```php
unset($collection['author']);
unset($collection->license);
```

### コレクションの反復処理

コレクションは反復可能なので、`foreach`ループで使用できます：

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### アイテム数のカウント

コレクション内のアイテム数を数えることができます：

```php
echo count($collection); // 出力: 4
```

### すべてのキーまたはデータを取得

すべてのキーを取得：

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

すべてのデータを配列として取得：

```php
$data = $collection->getData();
```

### コレクションのクリア

すべてのアイテムを削除：

```php
$collection->clear();
```

### JSONシリアライズ

コレクションは簡単にJSONに変換できます：

```php
echo json_encode($collection);
// 出力: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## 高度な使い方

必要に応じて、内部のデータ配列を完全に置き換えることができます：

```php
$collection->setData(['foo' => 'bar']);
```

コレクションは、コンポーネント間で構造化データを渡したい場合や、配列データに対してよりオブジェクト指向のインターフェースを提供したい場合に特に便利です。

## 関連項目

- [Requests](/learn/requests) - HTTPリクエストの処理方法と、リクエストデータを管理するためにコレクションをどのように使用できるかについて学びます。
- [SimplePdo](/learn/simple-pdo) - クエリ結果の行をコレクションとして返すデータベースヘルパーです。

## トラブルシューティング

- 存在しないキーにアクセスしようとすると、エラーではなく`null`が返ります。
- コレクションは再帰的ではないことに注意してください。ネストされた配列は自動的にコレクションには変換されません。
- コレクションをリセットする必要がある場合は、`$collection->clear()`または`$collection->setData([])`を使用します。

## 変更履歴

- v3.0 - 型ヒントの改善とPHP 8+サポート。
- v1.0 - Collectionクラスの初期リリース。