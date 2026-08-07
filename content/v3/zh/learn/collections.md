# 集合

## 概述

`Flight` 中的 `Collection` 类是一个用于管理数据集的便捷工具。它允许你使用数组和对象两种表示法来访问和操作数据，从而使你的代码更简洁、更灵活。

## 理解

`Collection` 本质上是对数组的封装，但拥有一些额外的能力。你可以像使用数组一样使用它，遍历它，统计它的项目数量，甚至可以像访问对象属性一样访问其中的项目。当你想要在应用中传递结构化数据，或者想让代码更具可读性时，这尤其有用。

集合实现了多个 PHP 接口：
- `ArrayAccess`（因此你可以使用数组语法）
- `Iterator`（因此你可以使用 `foreach` 循环）
- `Countable`（因此你可以使用 `count()`）
- `JsonSerializable`（因此你可以轻松转换为 JSON）

## 基本用法

### 创建集合

你可以通过简单地将数组传递给构造函数来创建一个集合：

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### 访问项目

你可以使用数组或对象表示法来访问项目：

```php
// 数组表示法
echo $collection['name']; // 输出：FlightPHP

// 对象表示法
echo $collection->version; // 输出：3
```

如果你尝试访问一个不存在的键，你将得到 `null` 而不是错误。

### 设置项目

你也可以使用任意一种表示法来设置项目：

```php
// 数组表示法
$collection['author'] = 'Mike Cao';

// 对象表示法
$collection->license = 'MIT';
```

### 检查与移除项目

检查项目是否存在：

```php
if (isset($collection['name'])) {
  // 执行某些操作
}

if (isset($collection->version)) {
  // 执行某些操作
}
```

移除项目：

```php
unset($collection['author']);
unset($collection->license);
```

### 遍历集合

集合是可迭代的，因此你可以在 `foreach` 循环中使用它们：

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### 统计项目数量

你可以统计集合中项目的数量：

```php
echo count($collection); // 输出：4
```

### 获取所有键或数据

获取所有键：

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

以数组形式获取所有数据：

```php
$data = $collection->getData();
```

### 清空集合

移除所有项目：

```php
$collection->clear();
```

### JSON 序列化

集合可以轻松转换为 JSON：

```php
echo json_encode($collection);
// 输出：{"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## 高级用法

如果需要，你可以完全替换内部数据数组：

```php
$collection->setData(['foo' => 'bar']);
```

当你想要在组件之间传递结构化数据，或者想要为数组数据提供更面向对象的接口时，集合尤其有用。

## 另请参阅

- [Requests](/learn/requests) - 了解如何处理 HTTP 请求，以及如何使用集合来管理请求数据。
- [SimplePdo](/learn/simple-pdo) - 返回查询行作为集合的数据库助手。

## 故障排除

- 如果你尝试访问一个不存在的键，你将得到 `null` 而不是错误。
- 请记住，集合不是递归的：嵌套数组不会自动转换为集合。
- 如果需要重置集合，请使用 `$collection->clear()` 或 `$collection->setData([])`。

## 更新日志

- v3.0 - 改进了类型提示并支持 PHP 8+。
- v1.0 - 初次发布 Collection 类。