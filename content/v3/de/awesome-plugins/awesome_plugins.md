# Fantastische Plugins

Flight ist unglaublich erweiterbar. Es gibt eine Reihe von Plugins, die verwendet werden können, um Funktionalität zu Ihrer Flight-Anwendung hinzuzufügen. Einige werden offiziell vom Flight-Team unterstützt und andere sind Micro-/Lite-Bibliotheken, die Ihnen den Einstieg erleichtern.

## KI-Tools

Flight kann mit KI-gestützten Plugins noch cooler gemacht werden.

- [Flight MCP](/awesome-plugins/mcp) - Ein Plugin zur Integration von MCP (Model Control Protocol) mit Flight, das nahtlose KI-gestützte Funktionalität ermöglicht. Hauptsächlich auf die Dokumentationsseiten ausgerichtet, hilft es, Token-Kosten niedrig zu halten, indem es die aktuellsten Informationen über Ihre Flight-Projekte bereitstellt.

## API-Dokumentation

API-Dokumentation ist für jede API entscheidend. Sie hilft Entwicklern zu verstehen, wie sie mit Ihrer API interagieren und was sie als Rückgabe erwarten können. Es gibt ein paar Tools, die Ihnen helfen, API-Dokumentation für Ihre Flight-Projekte zu generieren.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Blog-Beitrag von Daniel Schreiber darüber, wie man die OpenAPI-Spezifikation mit FlightPHP verwendet, um Ihre API mit einem API-First-Ansatz aufzubauen.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI ist ein großartiges Tool, das Ihnen hilft, API-Dokumentation für Ihre Flight-Projekte zu generieren. Es ist sehr einfach zu bedienen und kann an Ihre Bedürfnisse angepasst werden. Dies ist die PHP-Bibliothek, die Ihnen hilft, die Swagger-Dokumentation zu generieren.

## Anwendungsleistungsüberwachung (APM)

Anwendungsleistungsüberwachung (APM) ist für jede Anwendung entscheidend. Sie hilft Ihnen zu verstehen, wie Ihre Anwendung performt und wo die Engpässe sind. Es gibt eine Reihe von APM-Tools, die mit Flight verwendet werden können.
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM ist eine einfache APM-Bibliothek, die zur Überwachung Ihrer Flight-Anwendungen verwendet werden kann. Sie kann zur Leistungsüberwachung Ihrer Anwendung verwendet werden und hilft Ihnen, Engpässe zu identifizieren.

## Async

Flight ist bereits ein schnelles Framework, aber wenn man ihm einen Turbomotor verpasst, macht alles noch mehr Spaß (und ist herausfordernder)!

- [flightphp/async](/awesome-plugins/async) - Offizielle Flight Async-Bibliothek. Diese Bibliothek ist eine einfache Möglichkeit, asynchrone Verarbeitung zu Ihrer Anwendung hinzuzufügen. Sie verwendet Swoole/Openswoole unter der Haube, um eine einfache und effektive Möglichkeit zur asynchronen Ausführung von Aufgaben bereitzustellen.

## Autorisierung/Berechtigungen

Autorisierung und Berechtigungen sind für jede Anwendung entscheidend, die Kontrollen erfordert, wer auf was zugreifen kann.

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - Offizielle Flight Permissions-Bibliothek. Diese Bibliothek ist eine einfache Möglichkeit, Benutzer- und Anwendungsebene-Berechtigungen zu Ihrer Anwendung hinzuzufügen.

## Authentifizierung

Authentifizierung ist für Anwendungen unerlässlich, die Benutzeridentität überprüfen und API-Endpunkte sichern müssen.

- [firebase/php-jwt](/awesome-plugins/jwt) - JSON Web Token (JWT)-Bibliothek für PHP. Eine einfache und sichere Möglichkeit, token-basierte Authentifizierung in Ihren Flight-Anwendungen zu implementieren. Perfekt für zustandslose API-Authentifizierung, Routen-Schutz mit Middleware und die Implementierung von OAuth-ähnlichen Autorisierungsflüssen.

## Caching

Caching ist eine großartige Möglichkeit, Ihre Anwendung zu beschleunigen. Es gibt eine Reihe von Caching-Bibliotheken, die mit Flight verwendet werden können.

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Leichte, einfache und eigenständige PHP-Datei-Caching-Klasse

## CLI

CLI-Anwendungen sind eine großartige Möglichkeit, mit Ihrer Anwendung zu interagieren. Sie können sie verwenden, um Controller zu generieren, alle Routen anzuzeigen und mehr.

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway ist eine CLI-Anwendung, die Ihnen hilft, Ihre Flight-Anwendungen zu verwalten.

## Cookies

Cookies sind eine großartige Möglichkeit, kleine Datenmengen auf der Client-Seite zu speichern. Sie können verwendet werden, um Benutzereinstellungen, Anwendungseinstellungen und mehr zu speichern.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie ist eine PHP-Bibliothek, die eine einfache und effektive Möglichkeit zur Cookie-Verwaltung bietet.

## Debugging

Debugging ist entscheidend, wenn Sie in Ihrer lokalen Umgebung entwickeln. Es gibt ein paar Plugins, die Ihr Debugging-Erlebnis verbessern können.

- [tracy/tracy](/awesome-plugins/tracy) - Dies ist ein vollständiger Fehlerhandler, der mit Flight verwendet werden kann. Er hat eine Reihe von Panels, die Ihnen helfen, Ihre Anwendung zu debuggen. Er ist auch sehr einfach zu erweitern und eigene Panels hinzuzufügen.
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Wird mit dem [Tracy](/awesome-plugins/tracy) Fehlerhandler verwendet, fügt dieses Plugin ein paar zusätzliche Panels hinzu, um speziell bei Flight-Projekten zu debuggen.

## Datenbanken

Datenbanken sind das Herzstück der meisten Anwendungen. Hier speichern und rufen Sie Daten ab. Einige Datenbankbibliotheken sind einfach Wrapper zum Schreiben von Abfragen und einige sind vollwertige ORMs.

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Offizieller Flight PDO-Helper, der Teil des Kerns ist. Dies ist ein moderner Wrapper mit praktischen Hilfsmethoden wie `insert()`, `update()`, `delete()` und `transaction()`, um Datenbankoperationen zu vereinfachen. Alle Ergebnisse werden als Collections zurückgegeben für flexiblen Array-/Objektzugriff. Kein ORM, nur eine bessere Möglichkeit, mit PDO zu arbeiten.
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Offizieller Flight PDO-Wrapper, der Teil des Kerns ist (veraltet ab v3.18.0). Verwenden Sie stattdessen SimplePdo.
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - Offizielles Flight ActiveRecord ORM/Mapper. Eine großartige kleine Bibliothek zum einfachen Abrufen und Speichern von Daten in Ihrer Datenbank.
- [byjg/php-migration](/awesome-plugins/migrations) - Plugin zur Nachverfolgung aller Datenbankänderungen für Ihr Projekt.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Leichtgewichtiger, fließender SQL-Query-Builder, der SQL und Parameter für vorbereitete Anweisungen generiert. Funktioniert hervorragend mit [SimplePdo](/learn/simple-pdo).

## Verschlüsselung

Verschlüsselung ist für jede Anwendung entscheidend, die sensible Daten speichert. Das Verschlüsseln und Entschlüsseln der Daten ist nicht besonders schwer, aber die ordnungsgemäße Speicherung des Verschlüsselungsschlüssels [kann](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [schwierig](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [sein](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). Das Wichtigste ist, Ihren Verschlüsselungsschlüssel niemals in einem öffentlichen Verzeichnis zu speichern oder ihn in Ihr Code-Repository zu übertragen.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - Dies ist eine Bibliothek, die zum Verschlüsseln und Entschlüsseln von Daten verwendet werden kann. Der Einstieg ist ziemlich einfach, um mit dem Verschlüsseln und Entschlüsseln von Daten zu beginnen.

## Job-Queue

Job-Queues sind wirklich hilfreich, um Aufgaben asynchron zu verarbeiten. Dies kann das Senden von E-Mails, die Verarbeitung von Bildern oder alles sein, was nicht in Echtzeit erledigt werden muss.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue ist eine Bibliothek, die zur asynchronen Verarbeitung von Jobs verwendet werden kann. Sie kann mit beanstalkd, MySQL/MariaDB, SQLite und PostgreSQL verwendet werden.

## Session

Sessions sind für APIs nicht wirklich nützlich, aber für den Aufbau einer Webanwendung können Sessions entscheidend für die Aufrechterhaltung von Zustand und Login-Informationen sein.

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - Offizielle Flight Session-Bibliothek. Dies ist eine einfache Session-Bibliothek, die zum Speichern und Abrufen von Session-Daten verwendet werden kann. Sie verwendet die integrierte Session-Verwaltung von PHP.
- [Ghostff/Session](/awesome-plugins/ghost-session) - PHP Session-Manager (non-blocking, flash, segment, session encryption). Verwendet PHP open_ssl für optionale Verschlüsselung/Entschlüsselung von Session-Daten.

## Templating

Templating ist das Herzstück jeder Webanwendung mit einer Benutzeroberfläche. Es gibt eine Reihe von Templating-Engines, die mit Flight verwendet werden können.

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - Dies ist eine sehr grundlegende Templating-Engine, die Teil des Kerns ist. Es wird nicht empfohlen, sie zu verwenden, wenn Sie mehr als ein paar Seiten in Ihrem Projekt haben.
- [latte/latte](/awesome-plugins/latte) - Latte ist eine vollwertige Templating-Engine, die sehr einfach zu bedienen ist und sich näher an der PHP-Syntax als Twig oder Smarty anfühlt. Sie ist auch sehr einfach zu erweitern und eigene Filter und Funktionen hinzuzufügen.
- [twig/twig](/awesome-plugins/twig) - Twig ist eine flexible, schnelle und sichere Template-Engine (dieselbe, die von Symfony verwendet wird). KI-Tools und viele PHP-Entwickler kennen sie gut, sie maskiert Ausgabe standardmäßig automatisch und hat ein riesiges Ökosystem von Erweiterungen.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate ist eine leistungsstarke PHP-Template-Engine mit Asset-Kompilierung, Template-Vererbung und Variablenverarbeitung. Bietet automatische CSS/JS-Minifizierung, Caching, Base64-Kodierung und optionale Flight PHP Framework-Integration.

## WordPress-Integration

Möchten Sie Flight in Ihrem WordPress-Projekt verwenden? Es gibt ein praktisches Plugin dafür!

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Dieses WordPress-Plugin ermöglicht es Ihnen, Flight direkt neben WordPress auszuführen. Es ist perfekt, um benutzerdefinierte APIs, Microservices oder sogar vollständige Anwendungen zu Ihrer WordPress-Website mit dem Flight-Framework hinzuzufügen. Super nützlich, wenn Sie das Beste aus beiden Welten wollen!

## Mitwirken

Haben Sie ein Plugin, das Sie teilen möchten? Senden Sie einen Pull-Request, um es zur Liste hinzuzufügen!