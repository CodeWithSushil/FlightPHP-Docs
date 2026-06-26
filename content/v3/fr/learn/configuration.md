# Configuration

## Aperçu

Flight fournit un moyen simple de configurer divers aspects du framework pour répondre aux besoins de votre application. Certains sont définis par défaut, mais vous pouvez les remplacer si nécessaire. Vous pouvez également définir vos propres variables pour les utiliser dans toute votre application.

## Compréhension

Vous pouvez personnaliser certains comportements de Flight en définissant des valeurs de configuration
via la méthode `set`.

```php
Flight::set('flight.log_errors', true);
```

Dans le fichier `app/config/config.php`, vous pouvez voir toutes les variables de configuration par défaut disponibles pour vous.

## Utilisation de base

### Options de configuration de Flight

Voici une liste de tous les paramètres de configuration disponibles :

- **flight.base_url** `?string` - Remplace l'URL de base de la requête si Flight s'exécute dans un sous-répertoire. (par défaut : null)
- **flight.case_sensitive** `bool` - Correspondance sensible à la casse pour les URL. (par défaut : false)
- **flight.handle_errors** `bool` - Permet à Flight de gérer toutes les erreurs en interne. (par défaut : true)
  - Si vous voulez que Flight gère les erreurs au lieu du comportement PHP par défaut, cela doit être true.
  - Si vous avez [Tracy](/awesome-plugins/tracy) installé, vous voulez le définir à false pour que Tracy puisse gérer les erreurs.
  - Si vous avez le plugin [APM](/awesome-plugins/apm) installé, vous voulez le définir à true pour que l'APM puisse enregistrer les erreurs.
- **flight.log_errors** `bool` - Enregistre les erreurs dans le fichier journal d'erreurs du serveur web. (par défaut : false)
  - Si vous avez [Tracy](/awesome-plugins/tracy) installé, Tracy enregistrera les erreurs selon les configurations de Tracy, et non cette configuration.
- **flight.debug** `bool` - Affiche des informations d'erreur détaillées (message d'exception, code et trace de pile) dans le navigateur lorsqu'une erreur se produit. (par défaut : false)
  - **Ne jamais activer cela en production** — cela divulgue des détails internes de l'application. Utilisez-le uniquement pour le développement local ou la mise en scène.
  - Lorsque `false`, une réponse `500 Internal Server Error` générique est affichée à la place. Associez-le à `flight.log_errors` pour capturer les erreurs côté serveur.
- **flight.allow_method_override** `bool` - Permet de remplacer la méthode HTTP via l'en-tête de requête `X-HTTP-Method-Override` ou un champ `_method` dans le corps POST. (par défaut : true)
  - **Définir ceci à `false` est recommandé** pour les applications qui n'ont pas besoin de la substitution de méthode basée sur les formulaires HTML, car cela empêche les clients de falsifier les requêtes `DELETE` ou `PUT` via un formulaire POST standard.
  - Voir [Security](/learn/security#flight-configuration-hardening) pour plus de détails.
- **flight.views.path** `string` - Répertoire contenant les fichiers de modèles de vue. (par défaut : ./views)
- **flight.views.extension** `string` - Extension de fichier des modèles de vue. (par défaut : .php)
- **flight.content_length** `bool` - Définit l'en-tête `Content-Length`. (par défaut : true)
  - Si vous utilisez [Tracy](/awesome-plugins/tracy), cela doit être défini à false pour que Tracy puisse s'afficher correctement.
- **flight.v2.output_buffering** `bool` - Utilise la mise en mémoire tampon de sortie héritée. Voir [migrating to v3](migrating-to-v3). (par défaut : false)

### Configuration du chargeur

Il existe également un autre paramètre de configuration pour le chargeur. Cela vous permettra 
de charger automatiquement les classes avec `_` dans le nom de la classe.

```php
// Activer le chargement de classe avec des underscores
// Par défaut à true
Loader::$v2ClassLoading = false;
```

### Variables

Flight vous permet d'enregistrer des variables afin qu'elles puissent être utilisées n'importe où dans votre application.

```php
// Enregistrer votre variable
Flight::set('id', 123);

// Ailleurs dans votre application
$id = Flight::get('id');
```
Pour voir si une variable a été définie, vous pouvez faire :

```php
if (Flight::has('id')) {
  // Faire quelque chose
}
```

Vous pouvez effacer une variable en faisant :

```php
// Efface la variable id
Flight::clear('id');

// Efface toutes les variables
Flight::clear();
```

> **Note :** Ce n'est pas parce que vous pouvez définir une variable que vous devriez le faire. Utilisez cette fonctionnalité avec parcimonie. La raison est que tout ce qui est stocké ici devient une variable globale. Les variables globales sont mauvaises car elles peuvent être modifiées de n'importe où dans votre application, ce qui rend difficile le suivi des bugs. De plus, cela peut compliquer des choses comme les [tests unitaires](/guides/unit-testing).

### Erreurs et exceptions

Toutes les erreurs et exceptions sont capturées par Flight et transmises à la méthode `error`.
si `flight.handle_errors` est défini à true.

Le comportement par défaut consiste à envoyer une réponse générique `HTTP 500 Internal Server Error`
avec quelques informations d'erreur.

Vous pouvez [remplacer](/learn/extending) ce comportement selon vos besoins :

```php
Flight::map('error', function (Throwable $error) {
  // Gérer l'erreur
  echo $error->getTraceAsString();
});
```

Par défaut, les erreurs ne sont pas enregistrées sur le serveur web. Vous pouvez l'activer en
modifiant la configuration :

```php
Flight::set('flight.log_errors', true);
```

#### 404 Non trouvé

Lorsqu'une URL ne peut pas être trouvée, Flight appelle la méthode `notFound`. Le comportement
par défaut consiste à envoyer une réponse `HTTP 404 Not Found` avec un message simple.

Vous pouvez [remplacer](/learn/extending) ce comportement selon vos besoins :

```php
Flight::map('notFound', function () {
  // Gérer non trouvé
});
```

## Voir aussi
- [Extending Flight](/learn/extending) - Comment étendre et personnaliser les fonctionnalités principales de Flight.
- [Unit Testing](/guides/unit-testing) - Comment écrire des tests unitaires pour votre application Flight.
- [Tracy](/awesome-plugins/tracy) - Un plugin pour la gestion avancée des erreurs et le débogage.
- [Tracy Extensions](/awesome-plugins/tracy_extensions) - Extensions pour intégrer Tracy avec Flight.
- [APM](/awesome-plugins/apm) - Un plugin pour la surveillance des performances d'application et le suivi des erreurs.

## Dépannage
- Si vous avez des problèmes pour trouver toutes les valeurs de votre configuration, vous pouvez faire `var_dump(Flight::get());`

## Changelog
- v3.18.1 - Ajout des options de configuration `flight.debug` et `flight.allow_method_override`.
- v3.5.0 - Ajout de la configuration pour `flight.v2.output_buffering` pour prendre en charge le comportement de mise en mémoire tampon de sortie hérité.
- v2.0 - Configurations principales ajoutées.