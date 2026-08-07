# Tests unitaires

## Aperçu

Les tests unitaires dans Flight vous aident à garantir que votre application se comporte comme prévu, à détecter les bugs rapidement et à rendre votre base de code plus facile à maintenir. Flight est conçu pour fonctionner parfaitement avec [PHPUnit](https://phpunit.de/), le framework de test PHP le plus populaire.

## Compréhension

Les tests unitaires vérifient le comportement de petites parties de votre application (comme les contrôleurs ou les services) de manière isolée. Dans Flight, cela signifie tester comment vos routes, contrôleurs et logiques répondent à différentes entrées, sans dépendre de l'état global ou de véritables services externes.

Principes clés :
- **Testez le comportement, pas l'implémentation :** Concentrez-vous sur ce que votre code fait, pas sur la façon dont il le fait.
- **Évitez l'état global :** Utilisez l'injection de dépendances au lieu de `Flight::set()` ou `Flight::get()`.
- **Simulez les services externes :** Remplacez des éléments comme les bases de données ou les envois d'e-mails par des doubles de test.
- **Gardez les tests rapides et ciblés :** Les tests unitaires ne doivent pas toucher de vraies bases de données ou API.

## Utilisation de base

### Configuration de PHPUnit

1. Installez PHPUnit avec Composer :
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Créez un répertoire `tests` à la racine de votre projet.
3. Ajoutez un script de test à votre `composer.json` :
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Créez un fichier `phpunit.xml` :
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

Vous pouvez maintenant exécuter vos tests avec `composer test`.

### Tester un simple gestionnaire de route

Supposons que vous ayez une route qui valide un e-mail :

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
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Un test simple pour ce contrôleur :

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Conseils :**
- Simulez les données POST en utilisant `$app->request()->data`.
- Évitez d'utiliser les statiques `Flight::` dans vos tests—utilisez l'instance `$app`.

### Utiliser l'injection de dépendances pour des contrôleurs testables

Injectez les dépendances (comme la base de données ou l'expéditeur d'e-mails) dans vos contrôleurs pour les rendre faciles à simuler dans les tests :

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

Et un test avec des mocks :

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Utilisation avancée

- **Mocking :** Utilisez les mocks intégrés de PHPUnit ou des classes anonymes pour remplacer les dépendances.
- **Tester les contrôleurs directement :** Instanciez les contrôleurs avec un nouvel `Engine` et simulez les dépendances.
- **Évitez le sur-mock :** Laissez la logique réelle s'exécuter lorsque c'est possible ; ne simulez que les services externes.

## Voir aussi

- [Guide de tests unitaires](/guides/unit-testing) - Un guide complet sur les meilleures pratiques de tests unitaires.
- [Conteneur d'injection de dépendances](/learn/dependency-injection-container) - Comment utiliser les DIC pour gérer les dépendances et améliorer la testabilité.
- [Extension](/learn/extending) - Comment ajouter vos propres helpers ou remplacer les classes de base.
- [SimplePdo](/learn/simple-pdo) - Simplifie les interactions avec la base de données et est plus facile à simuler dans les tests.
- [Requêtes](/learn/requests) - Gérer les requêtes HTTP dans Flight.
- [Réponses](/learn/responses) - Envoyer des réponses aux utilisateurs.
- [Tests unitaires et principes SOLID](/learn/unit-testing-and-solid-principles) - Apprenez comment les principes SOLID peuvent améliorer vos tests unitaires.

## Dépannage

- Évitez d'utiliser l'état global (`Flight::set()`, `$_SESSION`, etc.) dans votre code et vos tests.
- Si vos tests sont lents, vous écrivez peut-être des tests d'intégration—simulez les services externes pour garder les tests unitaires rapides.
- Si la configuration des tests est complexe, envisagez de refactoriser votre code pour utiliser l'injection de dépendances.

## Journal des modifications

- v3.15.0 - Ajout d'exemples pour l'injection de dépendances et le mocking.