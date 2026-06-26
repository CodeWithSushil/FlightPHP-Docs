# Sécurité

## Aperçu

La sécurité est un élément crucial en ce qui concerne les applications web. Vous voulez vous assurer que votre application est sécurisée et que les données de vos utilisateurs sont 
sûres. Flight fournit un certain nombre de fonctionnalités pour vous aider à sécuriser vos applications web.

## Compréhension

Il existe un certain nombre de menaces de sécurité courantes dont vous devez être conscient lors de la création d'applications web. Certaines des menaces les plus courantes
incluent :
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Les templates](/learn/templates) aident à contrer le XSS en échappant la sortie par défaut afin que vous n'ayez pas à vous en souvenir. [Les sessions](/awesome-plugins/session) peuvent aider avec le CSRF en stockant un jeton CSRF dans la session de l'utilisateur comme décrit ci-dessous. L'utilisation de requêtes préparées avec PDO peut aider à prévenir les attaques par injection SQL (ou en utilisant les méthodes pratiques de la classe [PdoWrapper](/learn/pdo-wrapper)). Le CORS peut être géré avec un simple hook avant que `Flight::start()` ne soit appelé.

Toutes ces méthodes fonctionnent ensemble pour aider à maintenir la sécurité de vos applications web. Il devrait toujours être au premier plan de vos préoccupations d'apprendre et de comprendre les meilleures pratiques en matière de sécurité.

## Utilisation de base

### En-têtes

Les en-têtes HTTP sont l'un des moyens les plus simples de sécuriser vos applications web. Vous pouvez utiliser les en-têtes pour prévenir le clickjacking, le XSS et d'autres attaques. 
Il existe plusieurs façons d'ajouter ces en-têtes à votre application.

Deux excellents sites web pour vérifier la sécurité de vos en-têtes sont [securityheaders.com](https://securityheaders.com/) et 
[observatory.mozilla.org](https://observatory.mozilla.org/). Après avoir configuré le code ci-dessous, vous pouvez facilement vérifier que vos en-têtes fonctionnent avec ces deux sites web.

#### Ajout manuel

Vous pouvez ajouter manuellement ces en-têtes en utilisant la méthode `header` sur l'objet `Flight\Response`.
```php
// Définit l'en-tête X-Frame-Options pour prévenir le clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Définit l'en-tête Content-Security-Policy pour prévenir le XSS
// Remarque : cet en-tête peut devenir très complexe, donc vous voudrez
//  consulter des exemples sur internet pour votre application
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Définit l'en-tête X-XSS-Protection pour prévenir le XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Définit l'en-tête X-Content-Type-Options pour prévenir le reniflage MIME
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Définit l'en-tête Referrer-Policy pour contrôler la quantité d'informations de référence envoyées
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Définit l'en-tête Strict-Transport-Security pour forcer HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Définit l'en-tête Permissions-Policy pour contrôler quelles fonctionnalités et APIs peuvent être utilisées
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Ceux-ci peuvent être ajoutés en haut de vos fichiers `routes.php` ou `index.php`.

#### Ajout comme filtre

Vous pouvez également les ajouter dans un filtre/hook comme suit :

```php
// Ajoute les en-têtes dans un filtre
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### Ajout comme middleware

Vous pouvez également les ajouter en tant que classe middleware qui offre la plus grande flexibilité pour les routes auxquelles appliquer cela. En général, ces en-têtes devraient être appliqués à toutes les réponses HTML et API.

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php ou partout où vous avez vos routes
// À noter, ce groupe de chaîne vide agit comme un middleware global pour
// toutes les routes. Bien sûr, vous pourriez faire la même chose et simplement ajouter
// cela uniquement à des routes spécifiques.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// plus de routes
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Le Cross Site Request Forgery (CSRF) est un type d'attaque où un site web malveillant peut faire en sorte que le navigateur d'un utilisateur envoie une requête à votre site web. 
Cela peut être utilisé pour effectuer des actions sur votre site web sans la connaissance de l'utilisateur. Flight ne fournit pas de mécanisme de protection CSRF intégré, 
mais vous pouvez facilement implémenter le vôtre en utilisant un middleware.

#### Configuration

D'abord, vous devez générer un jeton CSRF et le stocker dans la session de l'utilisateur. Vous pouvez ensuite utiliser ce jeton dans vos formulaires et le vérifier lorsque 
le formulaire est soumis. Nous utiliserons le plugin [flightphp/session](/awesome-plugins/session) pour gérer les sessions.

```php
// Génère un jeton CSRF et le stocke dans la session de l'utilisateur
// (en supposant que vous avez créé un objet session et l'avez attaché à Flight)
// voir la documentation des sessions pour plus d'informations
Flight::register('session', flight\Session::class);

// Vous n'avez besoin de générer qu'un seul jeton par session (afin qu'il fonctionne 
// sur plusieurs onglets et requêtes pour le même utilisateur)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Utilisation du template PHP Flight par défaut

```html
<!-- Utilisez le jeton CSRF dans votre formulaire -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- autres champs du formulaire -->
</form>
```

##### Utilisation de Latte

Vous pouvez également définir une fonction personnalisée pour afficher le jeton CSRF dans vos templates Latte.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// autres configurations...

	// Définit une fonction personnalisée pour afficher le jeton CSRF
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Et maintenant dans vos templates Latte, vous pouvez utiliser la fonction `csrf()` pour afficher le jeton CSRF.

```html
<form method="post">
	{csrf()}
	<!-- autres champs du formulaire -->
</form>
```

#### Vérification du jeton CSRF

Vous pouvez vérifier le jeton CSRF en utilisant plusieurs méthodes.

##### Middleware

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// index.php ou partout où vous avez vos routes
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// plus de routes
}, [ CsrfMiddleware::class ]);
```

##### Filtres d'événements

```php
// Ce middleware vérifie si la requête est une requête POST et si c'est le cas, il vérifie si le jeton CSRF est valide
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// capture le jeton csrf depuis les valeurs du formulaire
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// ou pour une réponse JSON
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Le Cross Site Scripting (XSS) est un type d'attaque où une entrée de formulaire malveillante peut injecter du code dans votre site web. La plupart de ces opportunités proviennent 
des valeurs de formulaire que vos utilisateurs finaux rempliront. Vous ne devez **jamais** faire confiance à la sortie de vos utilisateurs ! Supposez toujours que tous sont les 
meilleurs hackers du monde. Ils peuvent injecter du JavaScript ou du HTML malveillant dans votre page. Ce code peut être utilisé pour voler des informations de vos 
utilisateurs ou effectuer des actions sur votre site web. En utilisant la classe view de Flight ou un autre moteur de template comme [Latte](/awesome-plugins/latte), vous pouvez facilement échapper la sortie pour prévenir les attaques XSS.

```php
// Supposons que l'utilisateur soit malin et essaie d'utiliser ceci comme son nom
$name = '<script>alert("XSS")</script>';

// Cela va échapper la sortie
Flight::view()->set('name', $name);
// Cela affichera : &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Si vous utilisez quelque chose comme Latte enregistré comme votre classe view, cela va aussi échapper automatiquement cela.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

L'injection SQL est un type d'attaque où un utilisateur malveillant peut injecter du code SQL dans votre base de données. Cela peut être utilisé pour voler des informations 
de votre base de données ou effectuer des actions sur votre base de données. Encore une fois, vous ne devez **jamais** faire confiance à l'entrée de vos utilisateurs ! Supposez toujours qu'ils sont 
à la recherche de sang. Vous pouvez utiliser des requêtes préparées dans vos objets `PDO` qui empêcheront l'injection SQL.

```php
// En supposant que vous avez Flight::db() enregistré comme votre objet PDO
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// Si vous utilisez la classe PdoWrapper, cela peut facilement être fait en une ligne
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Vous pouvez faire la même chose avec un objet PDO avec des marqueurs ?
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### Exemple non sécurisé

Ce qui suit explique pourquoi nous utilisons des requêtes SQL préparées pour protéger contre des exemples innocents comme celui ci-dessous :

```php
// l'utilisateur final remplit un formulaire web.
// pour la valeur du formulaire, le hacker entre quelque chose comme ceci :
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Après que la requête soit construite, elle ressemble à ceci
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Cela semble étrange, mais c'est une requête valide qui fonctionnera. En fait,
// c'est une attaque par injection SQL très courante qui retournera tous les utilisateurs.

var_dump($users); // cela va vider tous les utilisateurs de la base de données, pas seulement le seul nom d'utilisateur
```

### Validation du callback JSONP

Si vous utilisez la méthode `Flight::jsonp()` de Flight, sachez que Flight valide le nom du paramètre de callback JSONP contre une regex de liste blanche stricte (`/^[A-Za-z_$][\w$.]{0,127}$/`). Tout nom de callback qui ne correspond pas à ce modèle fera que Flight lancera une exception, empêchant l'injection de JavaScript arbitraire via une valeur de callback malveillante.

Cette validation est intégrée et ne nécessite aucune configuration supplémentaire, mais il est utile d'en être conscient lors du débogage d'erreurs inattendues provenant des points de terminaison JSONP.

### CORS

Le Cross-Origin Resource Sharing (CORS) est un mécanisme qui permet à de nombreuses ressources (par exemple, polices, JavaScript, etc.) sur une page web d'être 
demandées depuis un autre domaine en dehors du domaine d'origine de la ressource. Flight n'a pas de fonctionnalité intégrée, 
mais cela peut facilement être géré avec un hook qui s'exécute avant que la méthode `Flight::start()` ne soit appelée.

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// personnalisez vos hôtes autorisés ici.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php ou partout où vous avez vos routes
$CorsUtil = new CorsUtil();

// Cela doit être exécuté avant que start ne s'exécute.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Durcissement de la configuration de Flight

Flight expose plusieurs paramètres du moteur qui ont des implications directes sur la sécurité. Configurer ces paramètres correctement est l'un des moyens les plus simples de durcir votre application.

#### `flight.allow_method_override`

Par défaut, Flight permet aux clients de remplacer la méthode HTTP d'une requête en utilisant soit l'en-tête `X-HTTP-Method-Override`, soit un champ `_method` dans un corps POST. Bien que cela soit pratique pour les formulaires HTML qui ne peuvent envoyer que `GET`/`POST`, cela peut être dangereux si vous ne vous y attendez pas — un attaquant pourrait forger des requêtes `DELETE` ou `PUT` via un formulaire normal.

Si votre application ne dépend pas de ce comportement (par exemple, vous construisez une API consommée par des clients modernes ou des frontends JavaScript qui peuvent envoyer n'importe quel verbe HTTP), vous devriez le désactiver :

```php
// Dans votre index.php ou fichier de bootstrap, avant Flight::start()
Flight::set('flight.allow_method_override', false);
```

La valeur par défaut est `true` pour la compatibilité ascendante, mais **la définir sur `false` est fortement recommandée** pour toute application qui n'a pas explicitement besoin de la fonctionnalité de remplacement.

#### `flight.debug`

Flight a un paramètre `flight.debug` qui contrôle si des informations d'erreur détaillées (message d'exception, code et trace complète de la pile) sont rendues dans le navigateur lorsqu'une exception non gérée se produit. La valeur par défaut est `false`, ce qui signifie qu'un message générique `500 Internal Server Error` est affiché — aucun détail interne n'est divulgué au client.

Ne l'activez jamais sur un serveur de production. Utilisez-le uniquement localement ou dans un environnement de staging :

```php
// Sûr uniquement pour le développement local — JAMAIS en production
Flight::set('flight.debug', true);
```

Lorsque `flight.debug` est `false` (par défaut), vous pouvez toujours capturer les erreurs en activant `flight.log_errors` :

```php
// Enregistre les erreurs côté serveur sans les exposer au client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Configuration de production recommandée

```php
// index.php ou app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Gestion des erreurs
Masquez les détails d'erreur sensibles en production pour éviter de divulguer des informations aux attaquants. En production, enregistrez les erreurs au lieu de les afficher avec `display_errors` défini sur `0`.

```php
// Dans votre bootstrap.php ou index.php

// ajoutez ceci à votre app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Désactive l'affichage des erreurs
    ini_set('log_errors', 1);     // Enregistre les erreurs à la place
    ini_set('error_log', '/path/to/error.log');
}

// Dans vos routes ou contrôleurs
// Utilisez Flight::halt() pour des réponses d'erreur contrôlées
Flight::halt(403, 'Access denied');
```

### Assainissement des entrées
Ne faites jamais confiance à l'entrée utilisateur. Assainissez-la en utilisant [filter_var](https://www.php.net/manual/en/function.filter-var.php) avant le traitement pour empêcher des données malveillantes de s'infiltrer.

```php

// Supposons une requête $_POST avec $_POST['input'] et $_POST['email']

// Assainit une entrée de chaîne
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Assainit un email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Hachage des mots de passe
Stockez les mots de passe de manière sécurisée et vérifiez-les en toute sécurité en utilisant les fonctions intégrées de PHP comme [password_hash](https://www.php.net/manual/en/function.password-hash.php) et [password_verify](https://www.php.net/manual/en/function.password-verify.php). Les mots de passe ne doivent jamais être stockés en texte clair, ni être chiffrés avec des méthodes réversibles. Le hachage garantit que même si votre base de données est compromise, les mots de passe réels restent protégés.

```php
$password = Flight::request()->data->password;
// Hache un mot de passe lors du stockage (par exemple, pendant l'enregistrement)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Vérifie un mot de passe (par exemple, pendant la connexion)
if (password_verify($password, $stored_hash)) {
    // Le mot de passe correspond
}
```

### Limitation de débit
Protégez contre les attaques par force brute ou les attaques par déni de service en limitant les taux de requête avec un cache.

```php
// En supposant que vous avez flightphp/cache installé et enregistré
// Utilisation de flightphp/cache dans un filtre
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Réinitialisation après 60 secondes
});
```

## Voir aussi
- [Sessions](/awesome-plugins/session) - Comment gérer les sessions utilisateur de manière sécurisée.
- [Templates](/learn/templates) - Utilisation des templates pour échapper automatiquement la sortie et prévenir le XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Interactions simplifiées avec la base de données avec des requêtes préparées.
- [Middleware](/learn/middleware) - Comment utiliser le middleware pour simplifier le processus d'ajout d'en-têtes de sécurité.
- [Responses](/learn/responses) - Comment personnaliser les réponses HTTP avec des en-têtes sécurisés.
- [Requests](/learn/requests) - Comment gérer et assainir l'entrée utilisateur.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Fonction PHP pour l'assainissement des entrées.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Fonction PHP pour le hachage sécurisé des mots de passe.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Fonction PHP pour vérifier les mots de passe hachés.

## Dépannage
- Reportez-vous à la section "Voir aussi" ci-dessus pour les informations de dépannage liées aux problèmes avec les composants du Framework Flight.

## Changelog
- v3.18.1 - Ajout de la section Durcissement de la configuration de Flight couvrant `flight.allow_method_override`, `flight.debug` et la validation du callback JSONP.
- v3.1.0 - Ajout de sections sur le CORS, la gestion des erreurs, l'assainissement des entrées, le hachage des mots de passe et la limitation de débit.
- v2.0 - Ajout de l'échappement pour les vues par défaut pour prévenir le XSS.