# Tests unitaires dans Flight PHP avec PHPUnit

Ce guide présente les tests unitaires dans Flight PHP avec [PHPUnit](https://phpunit.de/), destiné aux débutants qui souhaitent comprendre *pourquoi* les tests unitaires sont importants et comment les appliquer concrètement. Nous nous concentrerons sur le test du *comportement*—s'assurer que votre application fait ce que vous attendez, comme envoyer un e-mail ou enregistrer un enregistrement—plutôt que sur des calculs triviaux. Nous commencerons par un [gestionnaire de route](/learn/routing) simple et progresserons vers un [contrôleur](/learn/routing) plus complexe, en intégrant l'[injection de dépendances](/learn/dependency-injection-container) (DI) et le mock de services tiers.

## Pourquoi faire des tests unitaires ?

Les tests unitaires garantissent que votre code se comporte comme prévu, en détectant les bugs avant qu'ils n'atteignent la production. Ils sont particulièrement précieux dans Flight, où le routage léger et la flexibilité peuvent conduire à des interactions complexes. Pour les développeurs solo ou les équipes, les tests unitaires agissent comme un filet de sécurité, documentant le comportement attendu et prévenant les régressions lorsque vous revisitez votre code plus tard. Ils améliorent également la conception : un code difficile à tester signale souvent des classes trop complexes ou fortement couplées.

Contrairement aux exemples simplistes (par exemple, tester `x * y = z`), nous nous concentrerons sur des comportements réels, tels que la validation des entrées, l'enregistrement de données ou le déclenchement d'actions comme l'envoi d'e-mails. Notre objectif est de rendre les tests accessibles et significatifs.

## Principes directeurs généraux

1. **Tester le comportement, pas l'implémentation** : Concentrez-vous sur les résultats (par exemple, « e-mail envoyé » ou « enregistrement sauvegardé ») plutôt que sur les détails internes. Cela rend les tests robustes face au refactoring.
2. **Arrêtez d'utiliser `Flight::`** : Les méthodes statiques de Flight sont terriblement pratiques, mais rendent les tests difficiles. Vous devriez vous habituer à utiliser la variable `$app` à partir de `$app = Flight::app();`. `$app` a toutes les mêmes méthodes que `Flight::`. Vous pourrez toujours utiliser `$app->route()` ou `$this->app->json()` dans votre contrôleur, etc. Vous devriez également utiliser le vrai routeur de Flight avec `$router = $app->router()`, puis utiliser `$router->get()`, `$router->post()`, `$router->group()`, etc. Voir [Routage](/learn/routing).
3. **Gardez les tests rapides** : Des tests rapides encouragent une exécution fréquente. Évitez les opérations lentes comme les appels de base de données dans les tests unitaires. Si vous avez un test lent, c'est le signe que vous écrivez un test d'intégration, pas un test unitaire. Les tests d'intégration sont ceux où vous impliquez réellement de vraies bases de données, de vrais appels HTTP, de vrais envois d'e-mails, etc. Ils ont leur place, mais ils sont lents et peuvent être instables, ce qui signifie qu'ils échouent parfois pour une raison inconnue.
4. **Utilisez des noms descriptifs** : Les noms des tests doivent décrire clairement le comportement testé. Cela améliore la lisibilité et la maintenabilité.
5. **Évitez les variables globales comme la peste** : Minimisez l'utilisation de `$app->set()` et `$app->get()`, car elles agissent comme un état global, nécessitant des mocks dans chaque test. Privilégiez l'injection de dépendances ou un conteneur d'injection de dépendances (voir [Conteneur d'injection de dépendances](/learn/dependency-injection-container)). Même l'utilisation de la méthode `$app->map()` est techniquement une "globale" et doit être évitée au profit de l'injection de dépendances. Utilisez une bibliothèque de session telle que [flightphp/session](https://github.com/flightphp/session) afin de pouvoir simuler l'objet session dans vos tests. **N'appelez pas** [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) directement dans votre code, car cela injecte une variable globale dans votre code, ce qui rend les tests difficiles.
6. **Utilisez l'injection de dépendances** : Injectez les dépendances (par exemple, [`PDO`](https://www.php.net/manual/en/class.pdo.php), les expéditeurs d'e-mails) dans les contrôleurs pour isoler la logique et simplifier les mocks. Si vous avez une classe avec trop de dépendances, envisagez de la refactoriser en classes plus petites qui ont chacune une responsabilité unique selon les [principes SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Mockez les services tiers** : Mockez les bases de données, les clients HTTP (cURL) ou les services d'e-mails pour éviter les appels externes. Testez une ou deux couches en profondeur, mais laissez votre logique principale s'exécuter. Par exemple, si votre application envoie un SMS, vous ne voulez **PAS** vraiment envoyer un SMS à chaque exécution de vos tests, car ces frais s'accumuleraient (et ce serait plus lent). Au lieu de cela, mockez le service d'envoi de SMS et vérifiez simplement que votre code a appelé le service avec les bons paramètres.
8. **Visez une couverture élevée, pas la perfection** : Une couverture de lignes à 100 % est bonne, mais cela ne signifie pas réellement que tout votre code est testé comme il le devrait (allez donc vous renseigner sur la [couverture des branches/chemins dans PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Privilégiez les comportements critiques (par exemple, l'inscription des utilisateurs, les réponses API et la capture des réponses en échec).
9. **Utilisez des contrôleurs pour les routes** : Dans vos définitions de routes, utilisez des contrôleurs plutôt que des fermetures (closures). Le `flight\Engine $app` est injecté dans chaque contrôleur via le constructeur par défaut. Dans les tests, utilisez `$app = new Flight\Engine()` pour instancier Flight dans un test, injectez-le dans votre contrôleur et appelez les méthodes directement (par exemple, `$controller->register()`). Voir [Étendre Flight](/learn/extending) et [Routage](/learn/routing).
10. **Choisissez un style de mock et tenez-vous-y** : PHPUnit prend en charge plusieurs styles de mock (par exemple, prophecy, mocks intégrés), ou vous pouvez utiliser des classes anonymes qui ont leurs propres avantages comme la complétion de code, la rupture si vous changez la définition de la méthode, etc. Soyez simplement cohérent dans vos tests. Voir [Objets mock PHPUnit](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Utilisez la visibilité `protected` pour les méthodes/propriétés que vous souhaitez tester dans les sous-classes** : Cela vous permet de les redéfinir dans des sous-classes de test sans les rendre publiques, c'est particulièrement utile pour les mocks de classes anonymes.

## Configuration de PHPUnit

D'abord, configurez [PHPUnit](https://phpunit.de/) dans votre projet Flight PHP en utilisant Composer pour faciliter les tests. Consultez le [guide de démarrage de PHPUnit](https://phpunit.readthedocs.io/en/12.3/installation.html) pour plus de détails.

1. Dans le répertoire de votre projet, exécutez :
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Cela installe la dernière version de PHPUnit comme dépendance de développement.

2. Créez un répertoire `tests` à la racine de votre projet pour les fichiers de test.

3. Ajoutez un script de test à `composer.json` pour plus de commodité :
   ```json
   // autre contenu de composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Créez un fichier `phpunit.xml` à la racine :
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Maintenant, lorsque vos tests sont créés, vous pouvez exécuter `composer test` pour lancer les tests.

## Tester un gestionnaire de route simple

Commençons par une [route](/learn/routing) de base qui valide la saisie de l'e-mail d'un utilisateur. Nous testerons son comportement : renvoyer un message de succès pour les e-mails valides et une erreur pour les invalides. Pour la validation des e-mails, nous utilisons [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
	protected $app;

	public function __construct(flight\Engine $app) {
		$this->app = $app;
	}

	public function register() {
		$email = $this->app->request()->data->email;
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

Pour tester cela, créez un fichier de test. Voir [Tests unitaires et principes SOLID](/learn/unit-testing-and-solid-principles) pour plus d'informations sur la structure des tests :

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simule les données POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Simule les données POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Points clés** :
- Nous simulons les données POST à l'aide de la classe requête. N'utilisez pas de variables globales comme `$_POST`, `$_GET`, etc., car cela rend les tests plus compliqués (vous devez toujours réinitialiser ces valeurs ou d'autres tests pourraient échouer).
- Tous les contrôleurs auront par défaut l'instance `flight\Engine` injectée, même sans conteneur d'injection de dépendances configuré. Cela rend beaucoup plus facile de tester les contrôleurs directement.
- Il n'y a aucune utilisation de `Flight::`, ce qui rend le code plus facile à tester.
- Les tests vérifient le comportement : le statut et le message corrects pour les e-mails valides/invalides.

Exécutez `composer test` pour vérifier que la route se comporte comme prévu. Pour plus d'informations sur les [requêtes](/learn/requests) et les [réponses](/learn/responses) dans Flight, consultez la documentation correspondante.

## Utiliser l'injection de dépendances pour des contrôleurs testables

Pour des scénarios plus complexes, utilisez l'[injection de dépendances](/learn/dependency-injection-container) (DI) pour rendre les contrôleurs testables. Évitez les variables globales de Flight (par exemple, `Flight::set()`, `Flight::map()`, `Flight::register()`) car elles agissent comme un état global, nécessitant des mocks pour chaque test. Utilisez plutôt le conteneur d'injection de dépendances de Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) ou l'injection manuelle.

Utilisons [`flight\database\SimplePdo`](/learn/simple-pdo) plutôt que le PDO brut. Cette aide est beaucoup plus facile à simuler et à tester unitairement (et elle est préférée à la `PdoWrapper` obsolète).

Voici un contrôleur qui enregistre un utilisateur dans une base de données et envoie un e-mail de bienvenue :

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// l'ajout du return ici aide les tests unitaires à arrêter l'exécution
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Points clés** :
- Le contrôleur dépend d'une instance de [`SimplePdo`](/learn/simple-pdo) et d'une `MailerInterface` (un service de messagerie tiers simulé).
- Les dépendances sont injectées via le constructeur, évitant ainsi les variables globales.

### Tester le contrôleur avec des mocks

Maintenant, testons le comportement du `UserController` : validation des e-mails, enregistrement dans la base de données et envoi d'e-mails. Nous allons simuler la base de données et le service de messagerie pour isoler le contrôleur.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Parfois, mélanger les styles de mock est nécessaire
		// Ici, nous utilisons le mock intégré de PHPUnit pour PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Utilisation d'une classe anonyme pour simuler SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Lorsque nous le simulons de cette façon, nous ne faisons pas réellement d'appel à la base de données.
			// Nous pouvons configurer davantage ce mock PDOStatement pour simuler des échecs, etc.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Un constructeur vide contourne le constructeur parent
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// Il faut mapper jsonHalt pour éviter la sortie
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**Points clés** :
- Nous simulons `SimplePdo` et `MailerInterface` pour éviter les vrais appels à la base de données ou aux e-mails.
- Les tests vérifient le comportement : les e-mails valides déclenchent des insertions en base et des envois d'e-mails ; les e-mails invalides ignorent les deux.
- Simulez les dépendances tierces (par exemple, `SimplePdo`, `MailerInterface`), en laissant la logique du contrôleur s'exécuter.

### Trop simuler

Soyez prudent et ne simulez pas trop votre code. Laissez-moi vous donner un exemple ci-dessous pour expliquer pourquoi cela pourrait être une mauvaise chose avec notre `UserController`. Nous allons transformer cette vérification en une méthode appelée `isEmailValid` (utilisant `filter_var`) et les autres ajouts en une méthode séparée appelée `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// l'ajout du return ici aide les tests unitaires à arrêter l'exécution
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

Et maintenant, le test unitaire excessivement mocké qui ne teste en réalité rien :

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// nous sautons l'injection de dépendances supplémentaire ici parce que c'est "facile"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Contourner les dépendances dans le constructeur
			public function __construct($app) {
				$this->app = $app;
			}

			// Nous allons simplement forcer ceci à être valide.
			protected function isEmailValid($email) {
				return true; // Retourne toujours true, en contournant la validation réelle
			}

			// Contourne les appels réels à la base de données et au service de messagerie
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

Hourra, nous avons des tests unitaires et ils passent ! Mais attendez, que se passe-t-il si je modifie réellement le fonctionnement interne de `isEmailValid` ou `registerUser` ? Mes tests passeront toujours parce que j'ai mocké toutes les fonctionnalités. Laissez-moi vous montrer ce que je veux dire.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... autres méthodes ...

	protected function isEmailValid($email) {
		// Logique modifiée
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Maintenant, il ne doit accepter qu'un domaine spécifique
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Si j'exécutais mes tests unitaires ci-dessus, ils passeraient toujours ! Mais comme je ne testais pas le comportement (en laissant réellement une partie du code s'exécuter), j'ai potentiellement écrit un bug qui n'attend que de se produire en production. Le test devrait être modifié pour tenir compte du nouveau comportement, et aussi du cas où le comportement n'est pas celui attendu.

## Exemple complet

Vous pouvez trouver un exemple complet d'un projet Flight PHP avec tests unitaires sur GitHub : [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide). Pour une compréhension plus approfondie, consultez [Tests unitaires et principes SOLID](/learn/unit-testing-and-solid-principles).

## Pièges courants

- **Sur-mockage (Over-Mocking)** : Ne mockez pas chaque dépendance ; laissez une partie de la logique (par exemple, la validation du contrôleur) s'exécuter pour tester le comportement réel. Voir [Tests unitaires et principes SOLID](/learn/unit-testing-and-solid-principles).
- **État global** : L'utilisation intensive de variables globales PHP (par exemple, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) rend les tests fragiles. Il en va de même pour `Flight::`. Refactorisez pour passer les dépendances explicitement.
- **Configuration complexe** : Si la configuration des tests est lourde, votre classe a peut-être trop de dépendances ou de responsabilités, violant les [principes SOLID](/learn/unit-testing-and-solid-principles).

## Passer à l'échelle avec les tests unitaires

Les tests unitaires sont précieux dans les projets plus importants ou lorsqu'on revisite son code après des mois. Ils documentent le comportement et détectent les régressions, vous évitant d'avoir à réapprendre votre application. Pour les développeurs solo, testez les chemins critiques (par exemple, l'inscription des utilisateurs, le traitement des paiements). Pour les équipes, les tests garantissent un comportement cohérent entre les contributions. Voir [Pourquoi les frameworks ?](/learn/why-frameworks) pour plus d'informations sur les avantages des frameworks et des tests.

Contribuez avec vos propres conseils de test au dépôt de documentation Flight PHP !

_Écrit par [n0nag0n](https://github.com/n0nag0n) 2025_