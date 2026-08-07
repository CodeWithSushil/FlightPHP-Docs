# FlightPHP/Permissions

Este é um módulo de permissões que pode ser usado em seus projetos se você tiver múltiplas funções em sua aplicação e cada função tiver uma funcionalidade um pouco diferente. Este módulo permite que você defina permissões para cada função e então verifique se o usuário atual tem a permissão para acessar uma determinada página ou realizar uma determinada ação.

Clique [aqui](https://github.com/flightphp/permissions) para o repositório no GitHub.

Instalação
-------
Execute `composer require flightphp/permissions` e você está pronto!

Uso
-------
Primeiro você precisa configurar suas permissões, depois você informa ao seu aplicativo o que as permissões significam. Em última análise, você verificará suas permissões com `$Permissions->has()`, `->can()`, ou `is()`. `has()` e `can()` têm a mesma funcionalidade, mas são nomeados de forma diferente para tornar seu código mais legível.

## Exemplo Básico

Vamos supor que você tenha um recurso em sua aplicação que verifica se um usuário está logado. Você pode criar um objeto de permissões assim:

```php
// index.php
require 'vendor/autoload.php';

// algum código

// então você provavelmente tem algo que lhe diz qual é a função atual da pessoa
// provavelmente você tem algo onde puxa a função atual
// de uma variável de sessão que define isso
// depois que alguém faz login, caso contrário terão uma função 'guest' ou 'public'.
$current_role = 'admin';

// configura permissões
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// Você provavelmente vai querer persistir este objeto no Flight em algum lugar
Flight::set('permission', $permission);
```

Então em algum controlador, você pode ter algo assim.

```php
<?php

// algum controlador
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// faz algo
		} else {
			// faz outra coisa
		}
	}
}
```

Você também pode usar isso para rastrear se eles têm permissão para fazer algo em sua aplicação.
Por exemplo, se você tem uma forma dos usuários interagirem com postagens em seu software, você pode
verificar se eles têm permissão para realizar determinadas ações.

```php
$current_role = 'admin';

// configura permissões
$permission = new \flight\Permission($current_role);
$permission->defineRule('post', function($current_role) {
	if($current_role === 'admin') {
		$permissions = ['create', 'read', 'update', 'delete'];
	} else if($current_role === 'editor') {
		$permissions = ['create', 'read', 'update'];
	} else if($current_role === 'author') {
		$permissions = ['create', 'read'];
	} else if($current_role === 'contributor') {
		$permissions = ['create'];
	} else {
		$permissions = [];
	}
	return $permissions;
});
Flight::set('permission', $permission);
```

Então em algum controlador...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// faz algo
		} else {
			// faz outra coisa
		}
	}
}
```

## Injetando dependências
Você pode injetar dependências no closure que define as permissões. Isso é útil se você tiver algum tipo de alternância, id, ou qualquer outro ponto de dados que você queira verificar. O mesmo funciona para chamadas do tipo Class->Method, exceto que você define os argumentos no método.

### Closures

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... código
});

// no seu arquivo de controlador
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// faz algo
	} else {
		// faz outra coisa
	}
}
```

### Classes

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... código
	}
}
```

## Atalho para definir permissões com classes
Você também pode usar classes para definir suas permissões. Isso é útil se você tiver muitas permissões e quiser manter seu código limpo. Você pode fazer algo assim:
```php
<?php

// código bootstrap
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// Assumindo que você configurou isso previamente
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // todos podem visualizar um pedido
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // gerentes podem criar pedidos
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // se o usuário tiver um alternador especial, eles podem atualizar pedidos
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // administradores podem excluir pedidos
		}
		return $allowed_permissions;
	}
}
```
A parte legal é que também existe um atalho que você pode usar (que também pode ser armazenado em cache!!!) onde você simplesmente informa à classe de permissões para mapear todos os métodos em uma classe para permissões. Então se você tem um método chamado `order()` e um método chamado `company()`, estes serão automaticamente mapeados para que você possa simplesmente executar `$Permissions->has('order.read')` ou `$Permissions->has('company.read')` e funcionará. Definir isso é muito difícil, então fique comigo aqui. Você só precisa fazer isso:

Crie a classe de permissões que você quer agrupar.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// código para determinar permissões
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// código para determinar permissões
		return $permissions_array;
	}
}
```

Então torne as permissões descobríveis usando esta biblioteca.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Finalmente, chame a permissão em sua base de código para verificar se o usuário está autorizado a realizar uma determinada permissão.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('Você não pode criar um pedido. Desculpe!');
		}
	}
}
```

### Cache

Para habilitar o cache, veja a simples biblioteca [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Um exemplo de como habilitar isso está abaixo.
```php

// este $app pode fazer parte do seu código, ou
// você pode simplesmente passar null e ele
// puxará de Flight::app() no construtor
$app = Flight::app();

// Por enquanto aceita isso como um cache de arquivo. Outros podem ser facilmente
// adicionados no futuro.
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 é quantos segundos para armazenar em cache. Deixe isso de fora para não usar cache
```

E pronto!