# Flight vs. Fat-Free

## Was ist Fat-Free?
[Fat-Free](https://fatfreeframework.com) (liebevoll **F3** genannt) ist ein leistungsstarkes und dennoch einfach zu verwendendes PHP-Mikro-Framework, das dir hilft, dynamische und robuste Webanwendungen schnell zu erstellen!

Flight lässt sich in vielerlei Hinsicht mit Fat-Free vergleichen und ist wahrscheinlich der engste Verwandte in Bezug auf Funktionen und Einfachheit. Fat-Free hat viele Funktionen, die Flight nicht hat, aber es hat auch viele Funktionen, die Flight hat. Fat-Free beginnt, sein Alter zu zeigen und ist nicht mehr so beliebt wie früher.

Updates werden seltener und die Community ist nicht mehr so aktiv wie früher. Der Code ist einfach genug, aber manchmal kann der Mangel an Syntax-Disziplin das Lesen und Verstehen erschweren. Es funktioniert mit PHP 8.3, aber der Code selbst sieht immer noch aus, als stamme er aus PHP 5.3.

## Vorteile im Vergleich zu Flight

- Fat-Free hat ein paar mehr Sterne auf GitHub als Flight.
- Fat-Free hat eine ordentliche Dokumentation, aber es mangelt ihr in einigen Bereichen an Klarheit.
- Fat-Free hat einige wenige Ressourcen wie YouTube-Tutorials und Online-Artikel, die zum Erlernen des Frameworks genutzt werden können.
- Fat-Free hat [einige hilfreiche Plugins](https://fatfreeframework.com/3.8/api-reference) integriert, die manchmal nützlich sind.
- Fat-Free hat ein eingebautes ORM namens Mapper, das zur Interaktion mit deiner Datenbank verwendet werden kann. Flight hat [active-record](/awesome-plugins/active-record).
- Fat-Free hat Sessions, Caching und Lokalisierung integriert. Flight erfordert die Verwendung von Drittanbieter-Bibliotheken, aber dies wird in der [Dokumentation](/awesome-plugins) behandelt.
- Fat-Free hat eine kleine Gruppe von [Community-erstellten Plugins](https://fatfreeframework.com/3.8/development#Community), die zur Erweiterung des Frameworks verwendet werden können. Flight hat einige in der [Dokumentation](/awesome-plugins) und auf den [Beispielseiten](/examples) behandelt.
- Fat-Free hat wie Flight keine Abhängigkeiten.
- Fat-Free ist wie Flight darauf ausgerichtet, dem Entwickler die Kontrolle über seine Anwendung zu geben und eine einfache Entwicklererfahrung zu bieten.
- Fat-Free behält wie Flight die Abwärtskompatibilität bei (teilweise weil Updates [weniger häufig](https://github.com/bcosca/fatfree/releases) werden).
- Fat-Free ist wie Flight für Entwickler gedacht, die zum ersten Mal in die Welt der Frameworks eintauchen.
- Fat-Free hat eine eingebaute Template-Engine, die robuster ist als die Template-Engine von Flight. Flight empfiehlt [Latte](/awesome-plugins/latte), um dies zu erreichen.
- Fat-Free hat einen einzigartigen CLI-„Route“-Befehl, mit dem du CLI-Anwendungen direkt in Fat-Free erstellen und sie wie eine `GET`-Anfrage behandeln kannst. Flight erreicht dies mit [runway](/awesome-plugins/runway).

## Nachteile im Vergleich zu Flight

- Fat-Free hat einige Implementierungstests und sogar eine eigene, sehr grundlegende [Test](https://fatfreeframework.com/3.8/test)-Klasse. Es ist jedoch nicht zu 100 % mit Unit-Tests abgedeckt wie Flight.
- Du musst eine Suchmaschine wie Google verwenden, um die Dokumentationsseite tatsächlich zu durchsuchen.
- Flight hat einen Dunkelmodus auf seiner Dokumentationsseite. (Mikrofon-Drop)
- Fat-Free hat einige Module, die hoffnungslos ungepflegt sind.
- Flight hat [SimplePdo](/learn/simple-pdo) für den Datenbankzugriff, was etwas einfacher ist als die eingebaute `DB\SQL`-Klasse von Fat-Free (und gegenüber dem veralteten PdoWrapper bevorzugt wird).
- Flight hat ein [Berechtigungs-Plugin](/awesome-plugins/permissions), mit dem du deine Anwendung absichern kannst. Fat-Free erfordert die Verwendung einer Drittanbieter-Bibliothek.
- Flight hat ein ORM namens [active-record](/awesome-plugins/active-record), das sich mehr wie ein ORM anfühlt als der Mapper von Fat-Free. Der zusätzliche Vorteil von `active-record` besteht darin, dass du Beziehungen zwischen Datensätzen für automatische Joins definieren kannst, während der Mapper von Fat-Free erfordert, dass du [SQL-Views](https://fatfreeframework.com/3.8/databases#ProsandCons) erstellst.
- Erstaunlicherweise hat Fat-Free keinen Root-Namespace. Flight ist durchgehend namespaced, um nicht mit deinem eigenen Code zu kollidieren. Die `Cache`-Klasse ist hier der größte Übeltäter.
- Fat-Free hat keine Middleware. Stattdessen gibt es `beforeroute`- und `afterroute`-Hooks, die verwendet werden können, um Anfragen und Antworten in Controllern zu filtern.
- Fat-Free kann keine Routen gruppieren.
- Fat-Free hat einen Dependency-Injection-Container-Handler, aber die Dokumentation zur Verwendung ist unglaublich spärlich.
- Das Debuggen kann etwas knifflig werden, da praktisch alles in dem sogenannten [`HIVE`](https://fatfreeframework.com/3.8/quick-reference) gespeichert wird.