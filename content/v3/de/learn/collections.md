# Sammlungen

## Übersicht

Die `Collection`-Klasse in Flight ist ein praktisches Hilfsmittel zur Verwaltung von Datensätzen. Sie ermöglicht den Zugriff auf Daten sowohl mit Array- als auch mit Objektnotation, was deinen Code sauberer und flexibler macht.

## Verständnis

Eine `Collection` ist im Grunde ein Wrapper um ein Array, aber mit einigen zusätzlichen Funktionen. Du kannst sie wie ein Array verwenden, darüber iterieren, die Anzahl der Elemente zählen und sogar auf Elemente zugreifen, als wären es Objekteigenschaften. Das ist besonders nützlich, wenn du strukturierte Daten in deiner App weitergeben möchtest oder wenn du deinen Code etwas lesbarer machen willst.

Collections implementieren mehrere PHP-Schnittstellen:
- `ArrayAccess` (damit du Array-Syntax verwenden kannst)
- `Iterator` (damit du mit `foreach` iterieren kannst)
- `Countable` (damit du `count()` verwenden kannst)
- `JsonSerializable` (damit du einfach in JSON konvertieren kannst)

## Grundlegende Verwendung

### Eine Collection erstellen

Du kannst eine Collection erstellen, indem du einfach ein Array an den Konstruktor übergibst:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Auf Elemente zugreifen

Du kannst auf Elemente entweder mit Array- oder Objektnotation zugreifen:

```php
// Array-Notation
echo $collection['name']; // Ausgabe: FlightPHP

// Objekt-Notation
echo $collection->version; // Ausgabe: 3
```

Wenn du versuchst, auf einen Schlüssel zuzugreifen, der nicht existiert, erhältst du `null` anstatt eines Fehlers.

### Elemente setzen

Du kannst Elemente ebenfalls mit beiden Notationen setzen:

```php
// Array-Notation
$collection['author'] = 'Mike Cao';

// Objekt-Notation
$collection->license = 'MIT';
```

### Vorhandensein prüfen und Elemente entfernen

Prüfen, ob ein Element existiert:

```php
if (isset($collection['name'])) {
  // Etwas tun
}

if (isset($collection->version)) {
  // Etwas tun
}
```

Ein Element entfernen:

```php
unset($collection['author']);
unset($collection->license);
```

### Über eine Collection iterieren

Collections sind iterierbar, du kannst sie also in einer `foreach`-Schleife verwenden:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Elemente zählen

Du kannst die Anzahl der Elemente in einer Collection zählen:

```php
echo count($collection); // Ausgabe: 4
```

### Alle Schlüssel oder Daten abrufen

Alle Schlüssel abrufen:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Alle Daten als Array abrufen:

```php
$data = $collection->getData();
```

### Collection leeren

Alle Elemente entfernen:

```php
$collection->clear();
```

### JSON-Serialisierung

Collections können einfach in JSON konvertiert werden:

```php
echo json_encode($collection);
// Ausgabe: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Fortgeschrittene Verwendung

Du kannst das interne Datenarray bei Bedarf vollständig ersetzen:

```php
$collection->setData(['foo' => 'bar']);
```

Collections sind besonders nützlich, wenn du strukturierte Daten zwischen Komponenten weitergeben möchtest oder wenn du eine objektorientiertere Schnittstelle für Array-Daten bereitstellen willst.

## Siehe auch

- [Anfragen](/learn/requests) – Erfahre, wie du HTTP-Anfragen verarbeitest und wie Collections zur Verwaltung von Anfragedaten verwendet werden können.
- [SimplePdo](/learn/simple-pdo) – Datenbank-Helfer, der Abfragezeilen als Collections zurückgibt.

## Fehlerbehebung

- Wenn du versuchst, auf einen Schlüssel zuzugreifen, der nicht existiert, erhältst du `null` anstatt eines Fehlers.
- Denke daran, dass Collections nicht rekursiv sind: Verschachtelte Arrays werden nicht automatisch in Collections umgewandelt.
- Wenn du die Collection zurücksetzen musst, verwende `$collection->clear()` oder `$collection->setData([])`.

## Änderungsprotokoll

- v3.0 – Verbesserte Typ-Hinweise und PHP 8+-Unterstützung.
- v1.0 – Erstveröffentlichung der Collection-Klasse.