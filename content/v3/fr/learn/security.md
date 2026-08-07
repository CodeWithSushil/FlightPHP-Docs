# Sécurité

## Vue d'ensemble

La sécurité est un enjeu majeur en ce qui concerne les applications web. Vous voulez vous assurer que votre application est sécurisée et que les données de vos utilisateurs sont en sécurité. Flight fournit un certain nombre de fonctionnalités pour vous aider à sécuriser vos applications web.

Le [squelette](https://github.com/flightphp/skeleton) officiel inclut également un **`SECURITY.md`** dédié ainsi qu'un middleware d'en-têtes de sécurité, afin que les [outils d'IA](/learn/ai) (et les humains) disposent d'un endroit réfléchi pour les secrets, les en-têtes et les règles XSS/SQL — distinct du style de codage général dans `AGENTS.md`.

## Comprendre

Il existe un certain nombre de menaces de sécurité courantes dont vous devez être conscient lors de la création d'applications web. Parmi les menaces les plus courantes, on trouve :
- Cross Site Request Forgery (CSRF) / (Falsification de requête intersite)
- Cross Site Scripting (XSS) / (Script intersite)
- SQL Injection / (Injection SQL)
- Cross Origin Resource Sharing (CORS) / (Partage de ressources entre origines)

Les [Templates](/learn/templates) aident contre XSS en échappant les sorties par défaut (Twig et Latte le font ; profitez de cet avantage). Les [Sessions](/awesome-plugins/session) peuvent aider contre CSRF en stockant un jeton CSRF dans la session de l'utilisateur, comme décrit ci-dessous. L'utilisation de requêtes préparées avec PDO — ou d'assistants sur [SimplePdo](/learn/simple-pdo) — aide à prévenir l'injection SQL. Le CORS peut être géré avec un simple hook avant l'appel de `Flight::start()`.

Toutes ces méthodes fonctionnent ensemble pour aider à garder vos applications web sécurisées. Il doit toujours être à l'avant-plan de votre esprit d'apprendre et de comprendre les bonnes pratiques de sécurité. Ne demandez pas à un assistant IA de « désactiver CSP » ou d'affaiblir les en-têtes simplement pour qu'une page se charge sans comprendre le compromis.

## Utilisation de base

### En-têtes

Les en-têtes HTTP sont l'un des moyens les plus simples de sécuriser vos applications web. Vous pouvez utiliser les en-têtes pour empêcher le clickjacking, les XSS et d'autres attaques. Il existe plusieurs façons d'ajouter ces en-têtes à votre application.

Deux excellents sites pour vérifier la sécurité de vos en-têtes sont [securityheaders.com](https://securityheaders.com/) et [observatory.mozilla.org](https://observatory.mozilla.org/). Une fois le code ci-dessous configuré, vous pouvez facilement vérifier que vos en-têtes fonctionnent avec ces deux sites.

Le squelette inclut `App\Middleware\SecurityHeadersMiddleware` (CSP avec un nonce par requête, options de frame, HSTS, etc.). Préférez étendre cela délibérément plutôt que de désactiver les en-têtes.

#### Ajout manuel

Vous pouvez ajouter manuellement ces en-têtes en utilisant la méthode `header` sur l'objet `Flight\Response`.
```php
// Définir l'en-tête X-Frame-Options pour empêcher le clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Définir l'en-tête Content-Security-Policy pour empêcher XSS
// Remarque : cet en-tête peut devenir très complexe, vous voudrez donc
//  consulter des exemples sur Internet pour votre application
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Définir l'en-tête X-XSS-Protection pour empêcher XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Définir l'en-tête X-Content-Type-Options pour empêcher le reniflage MIME
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Définir l'en-tête Referrer-Policy pour contrôler la quantité d'informations de référent envoyée
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Définir l'en-tête Strict-Transport-Security pour forcer HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Définir l'en-tête Permissions-Policy pour contrôler les fonctionnalités et API pouvant être utilisées
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Ces en-têtes peuvent être ajoutés en haut de vos fichiers `routes.php` ou `index.php`.

#### Ajout en tant que filtre

Vous pouvez également les ajouter dans un filtre/hook comme suit :

```php
// Ajouter les en-têtes dans un filtre
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

#### Ajout en tant que middleware

Vous pouvez également les ajouter en tant que classe middleware, ce qui offre la plus grande flexibilité quant aux routes auxquelles appliquer ces en-têtes. En général, ces en-têtes doivent être appliqués à toutes les réponses HTML et API.

Chemin et espace de noms de style squelette (**la casse du dossier correspond à `App\Middleware`**) :

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

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
		// Préférer un nonce CSP provenant du bootstrap lorsque vous avez des scripts inline
		// (le squelette définit csp_nonce)
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — groupe de chaîne vide = middleware global pour toutes les routes
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// plus de routes
}, [SecurityHeadersMiddleware::class]);
```

Les projets plus anciens peuvent encore utiliser `app/middlewares` et `app\middlewares` ; cela fonctionne si les dossiers correspondent. Les nouvelles applications de squelette utilisent **`app/Middleware/`** et **`App\Middleware`**. Voir [Autoloading](/learn/autoloading).

### Cross Site Request Forgery (CSRF)

La falsification de requête intersite (CSRF) est un type d'attaque où un site web malveillant peut amener le navigateur d'un utilisateur à envoyer une requête à votre site web. Cela peut être utilisé pour effectuer des actions sur votre site web sans que l'utilisateur en ait connaissance. Flight ne fournit pas de mécanisme de protection CSRF intégré, mais vous pouvez facilement implémenter le vôtre en utilisant un middleware.

#### Configuration

Tout d'abord, vous devez générer un jeton CSRF et le stocker dans la session de l'utilisateur. Vous pouvez ensuite utiliser ce jeton dans vos formulaires et le vérifier lors de la soumission du formulaire. Nous utiliserons le plugin [flightphp/session](/awesome-plugins/session) pour gérer les sessions.

```php
// Générer un jeton CSRF et le stocker dans la session de l'utilisateur
// (en supposant que vous avez créé un objet session et l'avez attaché à Flight)
// voir la documentation des sessions pour plus d'informations
Flight::register('session', flight\Session::class);

// Vous n'avez besoin de générer qu'un seul jeton par session (il fonctionne donc
// sur plusieurs onglets et requêtes pour le même utilisateur)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Avec le template PHP par défaut de Flight

```html
<!-- Utiliser le jeton CSRF dans votre formulaire -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- autres champs du formulaire -->
</form>
```

##### Avec Twig (défaut du squelette)

Enregistrez une fonction Twig ou passez le jeton à chaque vue de formulaire. Exemple minimal avec un global et un champ de formulaire :

```php
// Lors de la configuration de Twig (ex. services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# autres champs #}
</form>
```

##### Avec Latte

Vous pouvez également définir une fonction personnalisée pour afficher le jeton CSRF dans vos templates Latte.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// autres configurations...

	// Définir une fonction personnalisée pour afficher le jeton CSRF
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Et maintenant, dans vos templates Latte, vous pouvez utiliser la fonction `csrf()` pour afficher le jeton CSRF.

```html
<form method="post">
	{csrf()}
	<!-- autres champs du formulaire -->
</form>
```

#### Vérifier le jeton CSRF

Vous pouvez vérifier le jeton CSRF en utilisant plusieurs méthodes.

##### Middleware

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

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

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// plus de routes
}, [CsrfMiddleware::class]);
```

##### Filtres d'événements

```php
// Ce middleware vérifie si la requête est une requête POST et si c'est le cas,
// il vérifie si le jeton CSRF est valide
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// capturer le jeton csrf depuis les valeurs du formulaire
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

Le Cross Site Scripting (XSS) est un type d'attaque où une entrée de formulaire malveillante peut injecter du code dans votre site web. La plupart de ces opportunités proviennent de valeurs de formulaire que vos utilisateurs finaux rempliront. Vous ne devez **jamais** faire confiance aux sorties de vos utilisateurs ! Supposez toujours qu'ils sont les meilleurs hackers du monde. Ils peuvent injecter du JavaScript ou du HTML malveillant dans votre page. Ce code peut être utilisé pour voler des informations à vos utilisateurs ou effectuer des actions sur votre site web. En utilisant la classe de vue de Flight ou un moteur de templates comme [Twig](/awesome-plugins/twig) ou [Latte](/awesome-plugins/latte), vous pouvez facilement échapper les sorties pour prévenir les attaques XSS.

```php
// Supposons que l'utilisateur est malin et essaie d'utiliser ceci comme nom
$name = '<script>alert("XSS")</script>';

// Cela échappera la sortie
Flight::view()->set('name', $name);
// Cela affichera : &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (défaut du squelette) et Latte échappent automatiquement par défaut — préférez-les à un echo PHP brut
Flight::render('template', ['name' => $name]);
// Twig : {{ name }}  → échappé
// Évitez |raw / les sorties non échappées sauf si le contenu est entièrement fiable
```

### Injection SQL

L'injection SQL est un type d'attaque où un utilisateur malveillant peut injecter du code SQL dans votre base de données. Cela peut être utilisé pour voler des informations depuis votre base de données ou effectuer des actions sur celle-ci. Encore une fois, vous ne devez **jamais** faire confiance aux entrées de vos utilisateurs ! Supposez toujours qu'ils sont assoiffés de sang. Utilisez des requêtes préparées — les assistants de [SimplePdo](/learn/simple-pdo) font de cette approche le chemin par défaut.

```php
// En supposant que vous avez enregistré Flight::db() comme SimplePdo (ou que vous injectez SimplePdo dans le contrôleur)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (recommandé) — une ligne avec des paramètres liés
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Même idée avec des espaces réservés ?
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

Dans les contrôleurs de style squelette, préférez l'injection par constructeur de `SimplePdo` plutôt que `Flight::db()` afin que les tests et le code généré par IA restent cohérents ([DIC](/learn/dependency-injection-container)).

#### Exemple non sécurisé

Voici pourquoi nous utilisons des requêtes préparées SQL pour nous protéger contre des exemples apparemment anodins comme celui-ci :

```php
// l'utilisateur final remplit un formulaire web.
// pour la valeur du formulaire, le hacker met quelque chose comme ceci :
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Après la construction de la requête, elle ressemble à ceci
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Cela semble étrange, mais c'est une requête valide qui fonctionnera. En fait,
// c'est une attaque par injection SQL très courante qui retournera tous les utilisateurs.

var_dump($users); // cela videra tous les utilisateurs de la base de données, pas seulement le nom d'utilisateur unique
```

### Secrets et configuration

- Placez les secrets dans **`.env`** (ou le véritable environnement), pas dans les exemples `config.php` commités.
- Règle du squelette : valeurs par défaut littérales dans `config.php` ; fusionnez l'environnement au bootstrap ; **ne lisez pas** `$_ENV` dans les contrôleurs — injectez la configuration à la place. Voir [Configuration](/learn/configuration).
- Ne commitez jamais de clés API, mots de passe de base de données ou clés de chiffrement de session. Orientez les outils d'IA vers **`SECURITY.md`** afin qu'ils n'inventent pas de raccourcis non sécurisés.

### Validation du callback JSONP

Si vous utilisez la méthode `Flight::jsonp()`, sachez que Flight valide le nom du paramètre de callback JSONP par rapport à une liste blanche regex stricte (`/^[A-Za-z_$][\w$.]{0,127}$/`). Tout nom de callback qui ne correspond pas à ce modèle amènera Flight à lever une exception, empêchant l'injection de JavaScript arbitraire via une valeur de callback malveillante.

Cette validation est intégrée et ne nécessite aucune configuration supplémentaire, mais il est bon de le savoir lors du débogage d'erreurs inattendues provenant de points de terminaison JSONP.

### CORS

Le partage de ressources entre origines (CORS) est un mécanisme qui permet à de nombreuses ressources (par exemple, les polices, JavaScript, etc.) d'une page web d'être demandées depuis un autre domaine extérieur au domaine d'origine de la ressource. Flight n'a pas de fonctionnalité intégrée, mais cela peut facilement être géré avec un hook à exécuter avant que la méthode `Flight::start()` ne soit appelée.

```php
// app/Utils/CorsUtil.php  (squelette : dossier Utils en PascalCase → App\Utils)

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
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

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// bootstrap / routes — à exécuter avant start
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Durcissement de la configuration de Flight

Flight expose plusieurs paramètres du moteur qui ont des implications directes en matière de sécurité. Les configurer correctement est l'un des moyens les plus simples de durcir votre application.

#### `flight.allow_method_override`

Par défaut, Flight permet aux clients de remplacer la méthode HTTP d'une requête en utilisant l'en-tête `X-HTTP-Method-Override` ou un champ `_method` dans le corps d'un POST. Bien que pratique pour les formulaires HTML qui ne peuvent envoyer que `GET`/`POST`, cela peut être dangereux si vous ne vous y attendez pas — un attaquant pourrait forger des requêtes `DELETE` ou `PUT` via un formulaire ordinaire.

Si votre application ne dépend pas de ce comportement (par exemple, si vous construisez une API consommée par des clients modernes ou des frontaux JavaScript qui peuvent envoyer n'importe quel verbe HTTP), vous devriez le désactiver :

```php
// Dans votre index.php ou fichier de bootstrap, avant Flight::start()
Flight::set('flight.allow_method_override', false);
```

La valeur par défaut est `true` pour la compatibilité ascendante, mais **il est fortement recommandé de la définir à `false`** pour toute application qui n'a pas explicitement besoin de la fonctionnalité de remplacement.

#### `flight.debug`

Flight dispose d'un paramètre `flight.debug` qui contrôle si des informations d'erreur détaillées (message d'exception, code et trace de pile complète) sont affichées dans le navigateur lorsqu'une exception non gérée se produit. La valeur par défaut est `false`, ce qui signifie qu'un message générique `500 Internal Server Error` est affiché — aucun détail interne n'est divulgué au client.

N'activez jamais cela sur un serveur de production. Utilisez-le uniquement en local ou dans un environnement de staging :

```php
// Sûr uniquement pour le développement local — JAMAIS en production
Flight::set('flight.debug', true);
```

Lorsque `flight.debug` est `false` (la valeur par défaut), vous pouvez toujours capturer les erreurs en activant `flight.log_errors` :

```php
// Journaliser les erreurs côté serveur sans les exposer au client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Configuration de production recommandée

```php
// index.php ou appliquée depuis la configuration de l'application / bootstrap
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Gestion des erreurs
Masquez les détails d'erreur sensibles en production pour éviter de divulguer des informations aux attaquants. En production, journalisez les erreurs au lieu de les afficher avec `display_errors` défini sur `0`.

```php
// Dans votre bootstrap.php ou index.php

// ajoutez ceci à votre app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Désactiver l'affichage des erreurs
    ini_set('log_errors', 1);     // Journaliser les erreurs à la place
    ini_set('error_log', '/path/to/error.log');
}

// Dans vos routes ou contrôleurs
// Utilisez Flight::halt() pour des réponses d'erreur contrôlées
Flight::halt(403, 'Access denied');
```

### Assainissement des entrées
Ne faites jamais confiance aux entrées utilisateur. Assainissez-les en utilisant [filter_var](https://www.php.net/manual/en/function.filter-var.php) avant de les traiter afin d'empêcher l'infiltration de données malveillantes. Préférez lire les entrées via `$app->request()` (ou `Flight::request()`) plutôt que `$_GET` / `$_POST` bruts dans le code de l'application.

```php

// Supposons une requête $_POST avec $_POST['input'] et $_POST['email']

// Assainir une entrée chaîne de caractères
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Assainir un email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Hachage des mots de passe
Stockez les mots de passe de manière sécurisée et vérifiez-les en toute sécurité en utilisant les fonctions intégrées de PHP comme [password_hash](https://www.php.net/manual/en/function.password-hash.php) et [password_verify](https://www.php.net/manual/en/function.password-verify.php). Les mots de passe ne doivent jamais être stockés en texte brut, ni être chiffrés avec des méthodes réversibles. Le hachage garantit que même si votre base de données est compromise, les mots de passe réels restent protégés.

```php
$password = Flight::request()->data->password;
// Hacher un mot de passe lors du stockage (par exemple, lors de l'inscription)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Vérifier un mot de passe (par exemple, lors de la connexion)
if (password_verify($password, $stored_hash)) {
    // Le mot de passe correspond
}
```

### Limitation du débit
Protégez-vous contre les attaques par force brute ou les attaques par déni de service en limitant les taux de requêtes avec un cache.

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
- [Templates](/learn/templates) - Échappement automatique avec Twig/Latte et XSS.
- [SimplePdo](/learn/simple-pdo) - Assistants de base de données avec requêtes préparées.
- [PdoWrapper](/learn/pdo-wrapper) - Déprécié ; utilisez SimplePdo pour tout nouveau code.
- [Middleware](/learn/middleware) - Comment utiliser le middleware pour simplifier l'ajout d'en-têtes de sécurité.
- [Configuration](/learn/configuration) - `.env` vs configuration littérale, indicateurs de production.
- [IA et expérience développeur](/learn/ai) - Gardez la politique de sécurité dans `SECURITY.md` pour les agents.
- [Réponses](/learn/responses) - Comment personnaliser les réponses HTTP avec des en-têtes sécurisés.
- [Requêtes](/learn/requests) - Comment gérer et assainir les entrées utilisateur.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Fonction PHP pour l'assainissement des entrées.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Fonction PHP pour le hachage sécurisé des mots de passe.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Fonction PHP pour vérifier les mots de passe hachés.

## Dépannage
- Reportez-vous à la section « Voir aussi » ci-dessus pour des informations de dépannage liées aux problèmes avec les composants du framework Flight.
- Si CSP bloque vos scripts, ajoutez un nonce (motif du squelette) ou une liste blanche d'origines spécifiques — ne définissez pas `script-src *` sans un plan.

## Journal des modifications
- Docs – Squelette `App\Middleware`, notes Twig CSRF/XSS, SimplePdo, secrets/`.env`, et `SECURITY.md` pour les projets adaptés à l'IA.
- v3.18.1 - Ajout de la section Durcissement de la configuration de Flight couvrant `flight.allow_method_override`, `flight.debug` et la validation du callback JSONP.
- v3.1.0 - Ajout de sections sur CORS, la gestion des erreurs, l'assainissement des entrées, le hachage des mots de passe et la limitation du débit.
- v2.0 - Ajout de l'échappement pour les vues par défaut afin de prévenir XSS.