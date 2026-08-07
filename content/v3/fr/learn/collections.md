# Collections

## Aperçu

La classe `Collection` dans Flight est un utilitaire pratique pour gérer des ensembles de données. Elle vous permet d'accéder aux données et de les manipuler en utilisant à la fois la notation tableau et la notation objet, rendant votre code plus propre et plus flexible.

## Compréhension

Une `Collection` est fondamentalement un wrapper autour d'un tableau, mais avec des pouvoirs supplémentaires. Vous pouvez l'utiliser comme un tableau, itérer dessus, compter ses éléments, et même accéder aux éléments comme s'il s'agissait de propriétés d'objet. C'est particulièrement utile lorsque vous souhaitez transmettre des données structurées dans votre application, ou lorsque vous voulez rendre votre code un peu plus lisible.

Les collections implémentent plusieurs interfaces PHP :
- `ArrayAccess` (ainsi vous pouvez utiliser la syntaxe de tableau)
- `Iterator` (ainsi vous pouvez boucler avec `foreach`)
- `Countable` (ainsi vous pouvez utiliser `count()`)
- `JsonSerializable` (ainsi vous pouvez facilement convertir en JSON)

## Utilisation de base

### Créer une collection

Vous pouvez créer une collection en passant simplement un tableau à son constructeur :

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Accéder aux éléments

Vous pouvez accéder aux éléments en utilisant la notation tableau ou la notation objet :

```php
// Notation tableau
echo $collection['name']; // Sortie : FlightPHP

// Notation objet
echo $collection->version; // Sortie : 3
```

Si vous essayez d'accéder à une clé qui n'existe pas, vous obtiendrez `null` au lieu d'une erreur.

### Définir des éléments

Vous pouvez également définir des éléments en utilisant l'une ou l'autre notation :

```php
// Notation tableau
$collection['author'] = 'Mike Cao';

// Notation objet
$collection->license = 'MIT';
```

### Vérifier et supprimer des éléments

Vérifiez si un élément existe :

```php
if (isset($collection['name'])) {
  // Faire quelque chose
}

if (isset($collection->version)) {
  // Faire quelque chose
}
```

Supprimer un élément :

```php
unset($collection['author']);
unset($collection->license);
```

### Itérer sur une collection

Les collections sont itérables, vous pouvez donc les utiliser dans une boucle `foreach` :

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Compter les éléments

Vous pouvez compter le nombre d'éléments dans une collection :

```php
echo count($collection); // Sortie : 4
```

### Obtenir toutes les clés ou les données

Obtenir toutes les clés :

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Obtenir toutes les données sous forme de tableau :

```php
$data = $collection->getData();
```

### Vider la collection

Supprimer tous les éléments :

```php
$collection->clear();
```

### Sérialisation JSON

Les collections peuvent être facilement converties en JSON :

```php
echo json_encode($collection);
// Sortie : {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Utilisation avancée

Vous pouvez remplacer entièrement le tableau de données interne si nécessaire :

```php
$collection->setData(['foo' => 'bar']);
```

Les collections sont particulièrement utiles lorsque vous souhaitez transmettre des données structurées entre les composants, ou lorsque vous voulez fournir une interface plus orientée objet pour les données de type tableau.

## Voir aussi

- [Requêtes](/learn/requests) - Apprenez à gérer les requêtes HTTP et comment les collections peuvent être utilisées pour gérer les données de requête.
- [SimplePdo](/learn/simple-pdo) - Helper de base de données qui renvoie les lignes de requête sous forme de collections.

## Dépannage

- Si vous essayez d'accéder à une clé qui n'existe pas, vous obtiendrez `null` au lieu d'une erreur.
- Rappelez-vous que les collections ne sont pas récursives : les tableaux imbriqués ne sont pas automatiquement convertis en collections.
- Si vous devez réinitialiser la collection, utilisez `$collection->clear()` ou `$collection->setData([])`.

## Journal des modifications

- v3.0 - Amélioration des indications de type et prise en charge de PHP 8+.
- v1.0 - Première version de la classe Collection.