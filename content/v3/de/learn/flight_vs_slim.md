# Flight vs. Slim

## Was ist Slim?
[Slim](https://slimframework.com) ist ein PHP-Micro-Framework, das dir hilft, schnell einfache, aber leistungsstarke Webanwendungen und APIs zu erstellen.

Ein Großteil der Inspiration für einige der v3-Funktionen von Flight stammt tatsächlich von Slim. Das Gruppieren von Routen und das Ausführen von Middleware in einer 
bestimmten Reihenfolge sind zwei Funktionen, die von Slim inspiriert wurden. Slim v3 wurde mit Fokus auf Einfachheit veröffentlicht, aber es gibt 
[gemischte Bewertungen](https://github.com/slimphp/Slim/issues/2770) bezüglich v4.

## Vorteile im Vergleich zu Flight

- Slim hat eine größere Community von Entwicklern, die wiederum praktische Module erstellen, die dir helfen, das Rad nicht neu zu erfinden.
- Slim folgt vielen Schnittstellen und Standards, die in der PHP-Community üblich sind, was die Interoperabilität erhöht.
- Slim hat ordentliche Dokumentation und Tutorials, die zum Erlernen des Frameworks genutzt werden können (aber nichts im Vergleich zu Laravel oder Symfony).
- Slim hat verschiedene Ressourcen wie YouTube-Tutorials und Online-Artikel, die zum Erlernen des Frameworks genutzt werden können.
- Slim lässt dich beliebige Komponenten verwenden, um die Kern-Routing-Funktionen zu handhaben, da es PSR-7-konform ist.

## Nachteile im Vergleich zu Flight

- Überraschenderweise ist Slim nicht so schnell, wie man es für ein Micro-Framework erwarten würde. Siehe die 
  [TechEmpower-Benchmarks](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) 
  für weitere Informationen.
- Flight richtet sich an Entwickler, die eine leichtgewichtige, schnelle und einfach zu bedienende Webanwendung erstellen möchten.
- Flight hat keine Abhängigkeiten, während [Slim ein paar Abhängigkeiten hat](https://github.com/slimphp/Slim/blob/4.x/composer.json), die du installieren musst.
- Flight ist auf Einfachheit und Benutzerfreundlichkeit ausgelegt.
- Eine der Kernfunktionen von Flight ist, dass es sein Bestes tut, um Abwärtskompatibilität zu gewährleisten. Slim v3 zu v4 war eine bahnbrechende Änderung.
- Flight ist für Entwickler gedacht, die zum ersten Mal in die Welt der Frameworks eintauchen.
- Flight kann auch Anwendungen auf Unternehmensebene erstellen, hat aber nicht so viele Beispiele und Tutorials wie Slim.
  Es erfordert auch mehr Disziplin seitens des Entwicklers, um Dinge organisiert und gut strukturiert zu halten.
- Flight gibt dem Entwickler mehr Kontrolle über die Anwendung, während Slim im Hintergrund etwas Magie einschleusen kann.
- Flight hat [SimplePdo](/learn/simple-pdo) für den Datenbankzugriff (bevorzugt gegenüber dem veralteten PdoWrapper). Slim erfordert die Verwendung einer 
  Drittanbieter-Bibliothek.
- Flight hat ein [Berechtigungs-Plugin](/awesome-plugins/permissions), das zum Sichern deiner Anwendung verwendet werden kann. Slim erfordert die Verwendung einer 
  Drittanbieter-Bibliothek.
- Flight hat ein ORM namens [active-record](/awesome-plugins/active-record), das zur Interaktion mit deiner Datenbank verwendet werden kann. Slim erfordert die Verwendung einer 
  Drittanbieter-Bibliothek.
- Flight hat eine CLI-Anwendung namens [runway](/awesome-plugins/runway), mit der du deine Anwendung über die Befehlszeile ausführen kannst. Slim hat das nicht.