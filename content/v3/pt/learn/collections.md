# Coleções

## Visão Geral

A classe `Collection` no Flight é um utilitário útil para gerenciar conjuntos de dados. Ela permite acessar e manipular dados usando tanto a notação de array quanto a de objeto, tornando seu código mais limpo e flexível.

## Entendendo

Uma `Collection` é basicamente um invólucro em torno de um array, mas com alguns poderes extras. Você pode usá-la como um array, percorrê-la, contar seus itens e até acessar itens como se fossem propriedades de objeto. Isso é especialmente útil quando você deseja passar dados estruturados em seu aplicativo ou quando quer tornar seu código um pouco mais legível.

As coleções implementam várias interfaces do PHP:
- `ArrayAccess` (então você pode usar sintaxe de array)
- `Iterator` (então você pode percorrer com `foreach`)
- `Countable` (então você pode usar `count()`)
- `JsonSerializable` (então você pode facilmente converter para JSON)

## Uso Básico

### Criando uma Coleção

Você pode criar uma coleção simplesmente passando um array para seu construtor:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Acessando Itens

Você pode acessar itens usando a notação de array ou de objeto:

```php
// Notação de array
echo $collection['name']; // Saída: FlightPHP

// Notação de objeto
echo $collection->version; // Saída: 3
```

Se você tentar acessar uma chave que não existe, obterá `null` em vez de um erro.

### Definindo Itens

Você também pode definir itens usando qualquer uma das notações:

```php
// Notação de array
$collection['author'] = 'Mike Cao';

// Notação de objeto
$collection->license = 'MIT';
```

### Verificando e Removendo Itens

Verifique se um item existe:

```php
if (isset($collection['name'])) {
  // Faça algo
}

if (isset($collection->version)) {
  // Faça algo
}
```

Remova um item:

```php
unset($collection['author']);
unset($collection->license);
```

### Percorrendo uma Coleção

As coleções são iteráveis, então você pode usá-las em um loop `foreach`:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Contando Itens

Você pode contar o número de itens em uma coleção:

```php
echo count($collection); // Saída: 4
```

### Obtendo Todas as Chaves ou Dados

Obtenha todas as chaves:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Obtenha todos os dados como um array:

```php
$data = $collection->getData();
```

### Limpando a Coleção

Remova todos os itens:

```php
$collection->clear();
```

### Serialização JSON

As coleções podem ser facilmente convertidas para JSON:

```php
echo json_encode($collection);
// Saída: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Uso Avançado

Você pode substituir o array de dados interno completamente, se necessário:

```php
$collection->setData(['foo' => 'bar']);
```

As coleções são especialmente úteis quando você deseja passar dados estruturados entre componentes, ou quando deseja fornecer uma interface mais orientada a objetos para dados de array.

## Veja Também

- [Requisições](/learn/requests) - Aprenda como lidar com requisições HTTP e como as coleções podem ser usadas para gerenciar dados de requisição.
- [SimplePdo](/learn/simple-pdo) - Auxiliar de banco de dados que retorna linhas de consulta como Coleções.

## Solução de Problemas

- Se você tentar acessar uma chave que não existe, obterá `null` em vez de um erro.
- Lembre-se de que as coleções não são recursivas: arrays aninhados não são convertidos automaticamente em coleções.
- Se você precisar redefinir a coleção, use `$collection->clear()` ou `$collection->setData([])`.

## Registro de Alterações

- v3.0 - Melhorias nas dicas de tipo e suporte ao PHP 8+.
- v1.0 - Lançamento inicial da classe Collection.