# flightphp/cache

Classe leve, simples e independente de cache em PHP em arquivo, bifurcada de [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Vantagens** 
- Leve, independente e simples
- Todo o código em um único arquivo - sem drivers desnecessários.
- Seguro - todo arquivo de cache gerado possui um cabeçalho PHP com die, tornando o acesso direto impossível mesmo que alguém conheça o caminho e seu servidor não esteja configurado corretamente
- Bem documentado e testado
- Lida corretamente com concorrência via flock
- Suporta PHP 7.4+
- Gratuito sob licença MIT

Este site de documentação está usando esta biblioteca para fazer cache de cada uma das páginas!

Clique [aqui](https://github.com/flightphp/cache) para ver o código.

## Instalação

Instale via composer:

```bash
composer require flightphp/cache
```

## Uso

O uso é bastante simples. Isso salva um arquivo de cache no diretório de cache.

```php
use flight\Cache;

$app = Flight::app();

// Você passa o diretório onde o cache será armazenado no construtor
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Isso garante que o cache seja usado apenas quando estiver em modo de produção
	// ENVIRONMENT é uma constante que é definida no seu arquivo bootstrap ou em outro lugar da sua aplicação
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Obter um Valor de Cache

Você usa o método `get()` para obter um valor em cache. Se quiser um método de conveniência que atualize o cache se ele estiver expirado, pode usar `refreshIfExpired()`.

```php

// Obter instância do cache
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // retorna os dados a serem armazenados em cache
}, 10); // 10 segundos

// ou
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 segundos
}
```

### Armazenar um Valor de Cache

Você usa o método `set()` para armazenar um valor no cache.

```php
Flight::cache()->set('simple-cache-test', 'meus dados em cache', 10); // 10 segundos
```

### Apagar um Valor de Cache

Você usa o método `delete()` para apagar um valor no cache.

```php
Flight::cache()->delete('simple-cache-test');
```

### Verificar se um Valor de Cache Existe

Você usa o método `exists()` para verificar se um valor existe no cache.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// fazer algo
}
```

### Limpar o Cache
Você usa o método `flush()` para limpar todo o cache.

```php
Flight::cache()->flush();
```

### Extrair metadados com cache

Se você quiser extrair timestamps e outros metadados sobre uma entrada de cache, certifique-se de passar `true` como parâmetro correto.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Atualizando dados!" . PHP_EOL;
    return date("H:i:s"); // retorna os dados a serem armazenados em cache
}, 10, true); // true = retorna com metadados
// ou
$data = $cache->get("simple-cache-meta-test", true); // true = retorna com metadados

/*
Exemplo de item em cache recuperado com metadados:
{
    "time":1511667506, <-- timestamp unix de salvamento
    "expire":10,       <-- tempo de expiração em segundos
    "data":"04:38:26", <-- dados desserializados
    "permanent":false
}

Usando metadados, podemos, por exemplo, calcular quando o item foi salvo ou quando expira
Também podemos acessar os próprios dados com a chave "data"
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // obtém o timestamp unix quando os dados expiram e subtrai o timestamp atual dele
$cacheddate = $data["data"]; // acessamos os próprios dados com a chave "data"

echo "Último salvamento do cache: $cacheddate, expira em $expiresin segundos";
```

## Código Fonte

Visite [https://github.com/flightphp/cache](https://github.com/flightphp/cache) para ver o código.