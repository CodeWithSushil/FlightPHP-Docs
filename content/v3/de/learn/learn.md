# Lerne Flight kennen

Flight ist ein schnelles, einfaches, erweiterbares Framework für PHP. Es ist sehr vielseitig und kann zum Erstellen jeder Art von Webanwendung verwendet werden. Es ist auf Einfachheit ausgelegt und so geschrieben, dass es leicht zu verstehen und zu verwenden ist – von Menschen und von [KI-Programmierassistenten](/learn/ai).

> **Hinweis:** Du wirst Beispiele sehen, die `Flight::` als statische Variable verwenden, und einige, die das `$app->` Engine-Objekt verwenden. Beide funktionieren austauschbar. `$app` und `$this->app` in einem Controller/Middleware sind der empfohlene Ansatz des Flight-Teams (und worauf das offizielle Skelett + `AGENTS.md` für neue Projekte standardisieren).

## Kernkomponenten

### [Routing](/learn/routing)

Lerne, wie du Routen für deine Webanwendung verwaltest. Dies beinhaltet auch das Gruppieren von Routen, Routenparameter und Middleware.

### [Middleware](/learn/middleware)

Lerne, wie du Middleware verwendest, um Anfragen und Antworten in deiner Anwendung zu filtern.

### [Autoloading](/learn/autoloading)

Lerne, wie du deine eigenen Klassen automatisch lädst. Die **Groß-/Kleinschreibung** der Ordner muss mit deinen Namespaces übereinstimmen – das Skelett verwendet `App\` und PascalCase-Ordner wie `app/Controller/`.

### [Requests](/learn/requests)

Lerne, wie du Anfragen und Antworten in deiner Anwendung behandelst.

### [Responses](/learn/responses)

Lerne, wie du Antworten an deine Benutzer sendest.

### [HTML-Templates](/learn/templates)

Lerne, wie du HTML mit Twig (Skelett-Standard), Latte oder anderen Engines renderst – nicht nur mit den eingebauten PHP-Views.

### [Sicherheit](/learn/security)

Lerne, wie du deine Anwendung vor häufigen Sicherheitsbedrohungen schützt.

### [Konfiguration](/learn/configuration)

Lerne, wie du das Framework für deine Anwendung konfigurierst.

### [Event Manager](/learn/events)

Lerne, wie du das Event-System verwendest, um deiner Anwendung benutzerdefinierte Events hinzuzufügen.

### [Flight erweitern](/learn/extending)

Lerne, wie du das Framework erweitern kannst, indem du deine eigenen Methoden und Klassen hinzufügst.

### [Methoden-Hooks und Filtern](/learn/filtering)

Lerne, wie du Event-Hooks zu deinen Methoden und internen Framework-Methoden hinzufügst.

### [Dependency-Injection-Container (DIC)](/learn/dependency-injection-container)

Lerne, wie du Dependency-Injection-Container (DIC) verwendest, um die Abhängigkeiten deiner Anwendung zu verwalten.

## Utility-Klassen

### [Collections](/learn/collections)

Collections werden verwendet, um Daten zu speichern und sie zur einfacheren Nutzung als Array oder als Objekt zugänglich zu machen.

### [JSON-Wrapper](/learn/json)

Diese Klasse bietet ein paar einfache Funktionen, um das Kodieren und Dekodieren deines JSON konsistent zu gestalten.

### [SimplePdo](/learn/simple-pdo)

PDO kann manchmal mehr Kopfschmerzen bereiten als nötig. SimplePdo ist eine moderne PDO-Hilfsklasse mit praktischen Methoden wie `insert()`, `update()`, `delete()` und `transaction()`, die Datenbankoperationen viel einfacher machen.

### [PdoWrapper](/learn/pdo-wrapper) (Veraltet)

Der ursprüngliche PDO-Wrapper ist seit v3.18.0 veraltet. Bitte verwende stattdessen [SimplePdo](/learn/simple-pdo).

### [Uploaded Datei-Handler](/learn/uploaded-file)

Eine einfache Klasse, die hilft, hochgeladene Dateien zu verwalten und an einen dauerhaften Ort zu verschieben.

## Wichtige Konzepte

### [Warum ein Framework?](/learn/why-frameworks)

Hier ist ein kurzer Artikel darüber, warum du ein Framework verwenden solltest. Es ist eine gute Idee, die Vorteile der Verwendung eines Frameworks zu verstehen, bevor du eines verwendest.

Zusätzlich wurde ein exzellentes Tutorial von [@lubiana](https://git.php.fail/lubiana) erstellt. Auch wenn es nicht im Detail auf Flight eingeht, hilft dir dieser Leitfaden, einige der wichtigsten Konzepte rund um ein Framework zu verstehen und warum sie vorteilhaft sind. Du findest das Tutorial [hier](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight im Vergleich zu anderen Frameworks](/learn/flight-vs-another-framework)

Wenn du von einem anderen Framework wie Laravel, Slim, Fat-Free oder Symfony zu Flight migrierst, hilft dir diese Seite, die Unterschiede zwischen den beiden zu verstehen.

## Weitere Themen

### [Unit Testing](/learn/unit-testing)

Folge dieser Anleitung, um zu lernen, wie du deinen Flight-Code Unit-testest, damit er felsenfest ist.

### [KI & Entwicklererlebnis](/learn/ai)

Flight ist dafür gebaut, mit Code-LLMs zusammenzuarbeiten: `AGENTS.md`, Runway `ai:*`-Befehle und ein klares Skelett-Layout, damit Agenten dem Muster folgen.

### [Migration v2 -> v3](/learn/migrating-to-v3)

Die Abwärtskompatibilität wurde größtenteils beibehalten, aber es gibt einige Änderungen, die du beachten solltest, wenn du von v2 auf v3 migrierst.