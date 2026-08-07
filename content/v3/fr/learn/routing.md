# Routage

## Aperçu
Le routage dans Flight PHP fait correspondre des modèles d'URL à des fonctions de rappel ou à des méthodes de classe, permettant un traitement des requêtes rapide et simple. Il est conçu pour une surcharge minimale, une utilisation adaptée aux débutants et une extensibilité sans dépendances externes.

## Compréhension
Le routage est le mécanisme central qui relie les requêtes HTTP à la logique de votre application dans Flight. En définissant des routes, vous spécifiez comment différentes URL déclenchent un code spécifique, que ce soit via des fonctions, des méthodes de classe ou des actions de contrôleur. Le système de routage de Flight est flexible, prenant en charge les modèles de base, les paramètres nommés, les expressions régulières et des fonctionnalités avancées telles que l'injection de dépendances et le routage de ressources. Cette approche garde votre code organisé et facile à maintenir, tout en restant rapide et simple pour les débutants et extensible pour les utilisateurs avancés.

> **Remarque :** Vous voulez mieux comprendre le routage ? Consultez la page ["pourquoi un framework ?"](/learn/why-frameworks) pour une explication plus approfondie.

## Utilisation de base

### Définir une route simple
Le routage de base dans Flight s'effectue en faisant correspondre un modèle d'URL à une fonction de rappel ou à un tableau contenant une classe et une méthode.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Les routes sont mises en correspondance dans l'ordre où elles sont définies. La première route qui correspond à une requête sera invoquée.

### Utiliser des fonctions comme rappels
Le rappel peut être n'importe quel objet appelable. Vous pouvez donc utiliser une fonction classique :

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Utiliser des classes et des méthodes comme contrôleur
Vous pouvez également utiliser une méthode (statique ou non) d'une classe :

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// ou
Flight::route('/', [ GreetingController::class, 'hello' ]); // méthode préférée
// ou
Flight::route('/', [ 'GreetingController::hello' ]);
// ou 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Ou en créant d'abord un objet puis en appelant la méthode :

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **Remarque :** Par défaut, lorsqu'un contrôleur est appelé dans le framework, la classe `flight\Engine` est toujours injectée, sauf si vous spécifiez autre chose via un [conteneur d'injection de dépendances](/learn/dependency-injection-container).

### Routage spécifique à une méthode
Par défaut, les modèles de route correspondent à toutes les méthodes de requête. Vous pouvez répondre à des méthodes spécifiques en plaçant un identifiant avant l'URL.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// Vous ne pouvez pas utiliser Flight::get() pour les routes car c'est une méthode
//    pour obtenir des variables, et non pour créer une route.
Flight::post('/', function() { /* code */ });
Flight::patch('/', function() { /* code */ });
Flight::put('/', function() { /* code */ });
Flight::delete('/', function() { /* code */ });
```

Vous pouvez également mapper plusieurs méthodes à un seul rappel en utilisant un délimiteur `|` :

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### Gestion spéciale des requêtes HEAD et OPTIONS
Flight fournit une gestion intégrée pour les requêtes HTTP `HEAD` et `OPTIONS` :

#### Requêtes HEAD
- **Les requêtes HEAD** sont traitées comme les requêtes `GET`, mais Flight supprime automatiquement le corps de la réponse avant de l'envoyer au client.
- Cela signifie que vous pouvez définir une route pour `GET`, et les requêtes HEAD vers la même URL ne renverront que les en-têtes (sans contenu), comme l'exigent les normes HTTP.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// Une requête HEAD vers /info renverra les mêmes en-têtes, mais sans corps.
```

#### Requêtes OPTIONS
Les requêtes `OPTIONS` sont automatiquement gérées par Flight pour toute route définie.
- Lorsqu'une requête OPTIONS est reçue, Flight répond avec un statut `204 No Content` et un en-tête `Allow` listant toutes les méthodes HTTP prises en charge pour cette route.
- Vous n'avez pas besoin de définir une route séparée pour OPTIONS.

```php
// Pour une route définie comme :
Flight::route('GET|POST /users', function() { /* ... */ });

// Une requête OPTIONS vers /users répondra avec :
//
// Statut : 204 No Content
// Allow : GET, POST, HEAD, OPTIONS
```

### Utiliser l'objet Router
De plus, vous pouvez récupérer l'objet Router qui dispose de méthodes utilitaires à votre disposition :

```php

$router = Flight::router();

// mappe toutes les méthodes comme Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// Requête GET
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* code */});
$router->put('/users/update/@id', 	function() { /* code */});
$router->delete('/users/@id', 		function() { /* code */});
$router->patch('/users/@id', 		function() { /* code */});
```

### Expressions régulières (Regex)
Vous pouvez utiliser des expressions régulières dans vos routes :

```php
Flight::route('/user/[0-9]+', function () {
  // Cela correspondra à /user/1234
});
```

Bien que cette méthode soit disponible, il est recommandé d'utiliser des paramètres nommés, ou des paramètres nommés avec des expressions régulières, car ils sont plus lisibles et plus faciles à maintenir.

### Paramètres nommés
Vous pouvez spécifier des paramètres nommés dans vos routes qui seront transmis à votre fonction de rappel. **Cela sert surtout à la lisibilité de la route. Veuillez consulter la section ci-dessous concernant une mise en garde importante.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Vous pouvez également inclure des expressions régulières avec vos paramètres nommés en utilisant le délimiteur `:` :

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Cela correspondra à /bob/123
  // Mais ne correspondra pas à /bob/12345
});
```

> **Remarque :** La correspondance des groupes regex `()` avec des paramètres positionnels n'est pas prise en charge. Ex. : `:'\(`

#### Mise en garde importante
Bien que dans l'exemple ci-dessus, il semble que `@name` soit directement lié à la variable `$name`, ce n'est pas le cas. C'est l'ordre des paramètres dans la fonction de rappel qui détermine ce qui lui est transmis. Si vous inversiez l'ordre des paramètres dans la fonction de rappel, les variables seraient également inversées. Voici un exemple :

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

Et si vous alliez à l'URL suivante : `/bob/123`, la sortie serait `hello, 123 (bob)!`. _Soyez prudent_ lorsque vous configurez vos routes et vos fonctions de rappel !

### Paramètres optionnels
Vous pouvez spécifier des paramètres nommés facultatifs pour la correspondance en plaçant des segments entre parenthèses.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Cela correspondra aux URL suivantes :
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Tout paramètre optionnel qui ne correspond pas sera transmis comme `NULL`.

### Routage par joker
La correspondance n'est effectuée que sur des segments d'URL individuels. Si vous souhaitez correspondre à plusieurs segments, vous pouvez utiliser le joker `*`.

```php
Flight::route('/blog/*', function () {
  // Cela correspondra à /blog/2000/02/01
});
```

Pour router toutes les requêtes vers un seul rappel, vous pouvez faire :

```php
Flight::route('*', function () {
  // Faire quelque chose
});
```

### Gestionnaire 404 Introuvable
Par défaut, si une URL est introuvable, Flight envoie une réponse `HTTP 404 Not Found` très simple et basique. Si vous souhaitez une réponse 404 plus personnalisée, vous pouvez [mapper](/learn/extending) votre propre méthode `notFound` :

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Vous pourriez aussi utiliser Flight::render() avec un modèle personnalisé.
    $output = <<<HTML
		<h1>My Custom 404 Not Found</h1>
		<h3>The page you have requested {$url} could not be found.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Gestionnaire de méthode introuvable
Par défaut, si une URL est trouvée mais que la méthode n'est pas autorisée, Flight envoie une réponse `HTTP 405 Method Not Allowed` très simple et basique (Ex. : Method Not Allowed. Allowed Methods are: GET, POST). Elle inclut également un en-tête `Allow` avec les méthodes autorisées pour cette URL.

Si vous souhaitez une réponse 405 plus personnalisée, vous pouvez [mapper](/learn/extending) votre propre méthode `methodNotFound` :

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Vous pourriez aussi utiliser Flight::render() avec un modèle personnalisé.
	$output = <<<HTML
		<h1>My Custom 405 Method Not Allowed</h1>
		<h3>The method you have requested for {$url} is not allowed.</h3>
		<p>Allowed Methods are: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## Utilisation avancée

### Injection de dépendances dans les routes
Si vous souhaitez utiliser l'injection de dépendances via un conteneur (PSR-11, PHP-DI, Dice, etc.), les seuls types de routes où cela est disponible sont soit la création directe de l'objet vous-même et l'utilisation du conteneur pour créer votre objet, soit l'utilisation de chaînes pour définir la classe et la méthode à appeler. Vous pouvez consulter la page [Injection de dépendances](/learn/dependency-injection-container) pour plus d'informations.

Voici un exemple rapide :

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// faire quelque chose avec $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// Configurez le conteneur avec les paramètres dont vous avez besoin
// Voir la page Injection de dépendances pour plus d'informations sur PSR-11
$dice = new \Dice\Dice();

// N'oubliez pas de réassigner la variable avec '$dice = ' !!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Enregistrez le gestionnaire de conteneur
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Les routes comme d'habitude
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// ou
Flight::route('/hello/@id', 'Greeting->hello');
// ou
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Transmettre l'exécution à la route suivante
<span class="badge bg-warning">Obsolète</span>
Vous pouvez transmettre l'exécution à la route correspondante suivante en retournant `true` depuis votre fonction de rappel.

```php
Flight::route('/user/@name', function (string $name) {
  // Vérifier une condition
  if ($name !== "Bob") {
    // Continuer vers la route suivante
    return true;
  }
});

Flight::route('/user/*', function () {
  // Cette route sera appelée
});
```

Il est maintenant recommandé d'utiliser un [middleware](/learn/middleware) pour gérer les cas d'utilisation complexes comme celui-ci.

### Alias de route
En assignant un alias à une route, vous pouvez ensuite appeler cet alias dynamiquement dans votre application pour qu'il soit généré plus tard dans votre code (par exemple : un lien dans un modèle HTML, ou la génération d'une URL de redirection).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// ou 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// plus tard dans le code quelque part
class UserController {
	public function update() {

		// code pour enregistrer l'utilisateur...
		$id = $user['id']; // 5 par exemple

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // retournera '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

C'est particulièrement utile si votre URL vient à changer. Dans l'exemple ci-dessus, disons que les utilisateurs ont été déplacés vers `/admin/users/@id` à la place. Grâce à l'alias défini pour la route, vous n'avez plus besoin de rechercher toutes les anciennes URLs dans votre code et de les modifier, car l'alias retournera désormais `/admin/users/5` comme dans l'exemple ci-dessus.

L'alias de route fonctionne également dans les groupes :

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// ou
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Inspecter les informations de route
Si vous souhaitez inspecter les informations de la route correspondante, il y a 2 façons de procéder :

1. Vous pouvez utiliser la propriété `executedRoute` sur l'objet `Flight::router()`.
2. Vous pouvez demander que l'objet route soit transmis à votre rappel en passant `true` comme troisième paramètre dans la méthode de route. L'objet route sera toujours le dernier paramètre transmis à votre fonction de rappel.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Faire quelque chose avec $route
  // Tableau des méthodes HTTP correspondantes
  $route->methods;

  // Tableau des paramètres nommés
  $route->params;

  // Expression régulière correspondante
  $route->regex;

  // Contient le contenu de tout '*' utilisé dans le modèle d'URL
  $route->splat;

  // Affiche le chemin de l'URL....si vous en avez vraiment besoin
  $route->pattern;

  // Affiche le middleware assigné à cette route
  $route->middleware;

  // Affiche l'alias assigné à cette route
  $route->alias;
});
```

> **Remarque :** La propriété `executedRoute` n'est définie qu'après l'exécution d'une route. Si vous essayez d'y accéder avant qu'une route ne soit exécutée, elle sera `NULL`. Vous pouvez également utiliser `executedRoute` dans un [middleware](/learn/middleware) !

#### Passer `true` dans la définition de route
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Tableau des méthodes HTTP correspondantes
  $route->methods;

  // Tableau des paramètres nommés
  $route->params;

  // Expression régulière correspondante
  $route->regex;

  // Contient le contenu de tout '*' utilisé dans le modèle d'URL
  $route->splat;

  // Affiche le chemin de l'URL....si vous en avez vraiment besoin
  $route->pattern;

  // Affiche le middleware assigné à cette route
  $route->middleware;

  // Affiche l'alias assigné à cette route
  $route->alias;
}, true);// <-- Ce paramètre true est ce qui permet cela
```

### Regroupement de routes et middleware
Il peut y avoir des moments où vous souhaitez regrouper des routes connexes (comme `/api/v1`). Vous pouvez le faire en utilisant la méthode `group` :

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Correspond à /api/v1/users
  });

  Flight::route('/posts', function () {
	// Correspond à /api/v1/posts
  });
});
```

Vous pouvez même imbriquer des groupes de groupes :

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() récupère des variables, il ne définit pas une route ! Voir le contexte objet ci-dessous
	Flight::route('GET /users', function () {
	  // Correspond à GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Correspond à POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Correspond à PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() récupère des variables, il ne définit pas une route ! Voir le contexte objet ci-dessous
	Flight::route('GET /users', function () {
	  // Correspond à GET /api/v2/users
	});
  });
});
```

#### Regroupement avec le contexte objet
Vous pouvez toujours utiliser le regroupement de routes avec l'objet `Engine` de la manière suivante :

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // utiliser la variable $router
  $router->get('/users', function () {
	// Correspond à GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Correspond à POST /api/v1/posts
  });
});
```

> **Remarque :** C'est la méthode préférée pour définir les routes et les groupes avec l'objet `$router`.

#### Regroupement avec middleware
Vous pouvez également assigner un middleware à un groupe de routes :

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Correspond à /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // ou [ new MyAuthMiddleware() ] si vous souhaitez utiliser une instance
```

Voir plus de détails sur la page [middleware de groupe](/learn/middleware#grouping-middleware).

### Routage de ressources
Vous pouvez créer un ensemble de routes pour une ressource en utilisant la méthode `resource`. Cela créera un ensemble de routes pour une ressource qui suit les conventions RESTful.

Pour créer une ressource, procédez comme suit :

```php
Flight::resource('/users', UsersController::class);
```

Et ce qui se passera en arrière-plan, c'est que les routes suivantes seront créées :

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

Et votre contrôleur utilisera les méthodes suivantes :

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **Remarque :** Vous pouvez voir les routes nouvellement ajoutées avec `runway` en exécutant `php runway routes`.

#### Personnalisation des routes de ressources
Il y a quelques options pour configurer les routes de ressources.

##### Base d'alias
Vous pouvez configurer le `aliasBase`. Par défaut, l'alias est la dernière partie de l'URL spécifiée. Par exemple, `/users/` donnerait un `aliasBase` de `users`. Lorsque ces routes sont créées, les alias sont `users.index`, `users.create`, etc. Si vous souhaitez modifier l'alias, définissez le `aliasBase` à la valeur souhaitée.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only et Except
Vous pouvez également spécifier quelles routes vous souhaitez créer en utilisant les options `only` et `except`.

```php
// Autoriser uniquement ces méthodes et mettre le reste sur liste noire
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Mettre uniquement ces méthodes sur liste noire et autoriser le reste
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

Ce sont essentiellement des options de liste blanche et de liste noire pour vous permettre de spécifier les routes que vous souhaitez créer.

##### Middleware
Vous pouvez également spécifier un middleware à exécuter sur chacune des routes créées par la méthode `resource`.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Réponses en streaming
Vous pouvez maintenant diffuser des réponses vers le client en utilisant `stream()` ou `streamWithHeaders()`. Cela est utile pour envoyer de gros fichiers, des processus de longue durée, ou générer de grandes réponses. La diffusion d'une route est gérée un peu différemment d'une route classique.

> **Remarque :** Les réponses en streaming ne sont disponibles que si vous avez [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) défini sur `false`.

#### Streaming avec en-têtes manuels
Vous pouvez diffuser une réponse vers le client en utilisant la méthode `stream()` sur une route. Si vous faites cela, vous devez définir tous les en-têtes manuellement avant d'afficher quoi que ce soit au client. Cela se fait avec la fonction PHP `header()` ou la méthode `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// évidemment, vous assainiriez le chemin et tout le reste.
	$fileNameSafe = basename($filename);

	// Si vous avez des en-têtes supplémentaires à définir ici après l'exécution de la route
	// vous devez les définir avant que quoi que ce soit soit affiché.
	// Ils doivent tous être un appel brut à la fonction header() ou
	// un appel à Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// ou
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// définir manuellement la longueur du contenu si vous le souhaitez
	header('Content-Length: '.filesize($filePath));
	// ou
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Diffuser le fichier au client au fur et à mesure de sa lecture
	readfile($filePath);

// C'est la ligne magique ici
})->stream();
```

#### Streaming avec en-têtes
Vous pouvez également utiliser la méthode `streamWithHeaders()` pour définir les en-têtes avant de commencer la diffusion.

```php
Flight::route('/stream-users', function() {

	// vous pouvez ajouter ici tous les en-têtes supplémentaires que vous souhaitez
	// vous devez simplement utiliser header() ou Flight::response()->setRealHeader()

	// quelle que soit la façon dont vous récupérez vos données, à titre d'exemple...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Ceci est nécessaire pour envoyer les données au client
		ob_flush();
	}
	echo '}';

// C'est ainsi que vous définirez les en-têtes avant de commencer la diffusion.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// code de statut optionnel, par défaut 200
	'status' => 200
]);
```

## Voir aussi
- [Middleware](/learn/middleware) - Utiliser le middleware avec les routes pour l'authentification, la journalisation, etc.
- [Injection de dépendances](/learn/dependency-injection-container) - Simplifier la création et la gestion des objets dans les routes.
- [Pourquoi un framework ?](/learn/why-frameworks) - Comprendre les avantages d'utiliser un framework comme Flight.
- [Extension](/learn/extending) - Comment étendre Flight avec vos propres fonctionnalités, y compris la méthode `notFound`.
- [php.net : preg_match](https://www.php.net/manual/en/function.preg-match.php) - Fonction PHP pour la correspondance d'expressions régulières.

## Dépannage
- Les paramètres de route sont mis en correspondance par ordre, et non par nom. Assurez-vous que l'ordre des paramètres du rappel correspond à la définition de la route.
- Utiliser `Flight::get()` ne définit pas une route ; utilisez `Flight::route('GET /...')` pour le routage ou le contexte objet Router dans les groupes (par exemple `$router->get(...)`).
- La propriété `executedRoute` n'est définie qu'après l'exécution d'une route ; elle est NULL avant l'exécution.
- Le streaming nécessite que la fonctionnalité héritée de mise en mémoire tampon de sortie de Flight soit désactivée (`flight.v2.output_buffering = false`).
- Pour l'injection de dépendances, seules certaines définitions de route prennent en charge l'instanciation via un conteneur.

### Erreur 404 Introuvable ou comportement de route inattendu
Si vous voyez une erreur 404 Not Found (alors que vous jurez sur votre vie qu'elle est bien là et que ce n'est pas une faute de frappe), cela pourrait en réalité être un problème lié au fait que vous retournez une valeur dans votre point de terminaison de route au lieu de simplement l'afficher. La raison est intentionnelle, mais elle peut surprendre certains développeurs.

```php
Flight::route('/hello', function(){
	// Cela pourrait provoquer une erreur 404 Not Found
	return 'Hello World';
});

// Ce que vous voulez probablement
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

La raison en est un mécanisme spécial intégré au routeur qui traite la sortie retournée comme un signal pour « passer à la route suivante ». Vous pouvez voir ce comportement documenté dans la section [Routage](/learn/routing#passing).

## Journal des modifications
- v3 : Ajout du routage de ressources, des alias de route, de la prise en charge du streaming, des groupes de routes et du support des middlewares.
- v1 : La grande majorité des fonctionnalités de base disponibles.