# IA et expérience développeur avec Flight

## Vue d'ensemble

Flight est conçu pour fonctionner *avec* les outils de codage IA—pas contre eux. Une API petite et prévisible, une structure d'application claire dans le [squelette officiel](https://github.com/flightphp/skeleton), et des fichiers d'instructions spécifiques au projet permettent à des assistants comme GitHub Copilot, Cursor, Windsurf, Claude Code et Gemini de suivre les mêmes modèles que ceux que vous écririez à la main.

Grâce aux commandes Runway intégrées pour vous connecter aux fournisseurs de LLM et générer des instructions de projet, Flight vous aide, vous et votre équipe, à obtenir une aide cohérente et pertinente sans avoir à coller le même contexte dans chaque discussion.

## Comprendre

Les assistants de codage IA sont plus utiles lorsqu'ils comprennent le contexte, les conventions et les objectifs de votre projet. Les assistants IA de Flight vous permettent de :

- Connecter votre projet à des fournisseurs LLM populaires (OpenAI, Grok, Claude, etc.)
- Générer et mettre à jour des instructions spécifiques au projet afin que tout le monde reçoive les mêmes orientations
- Garder le code écrit à la main et le code généré par l'IA sur une seule structure (surtout avec le squelette)

Ces fonctionnalités sont incluses dans le CLI de base de Flight (via [Runway](/awesome-plugins/runway)) et sont préconfigurées dans le starter officiel [flightphp/skeleton](https://github.com/flightphp/skeleton).

### Ce que le squelette fournit pour l'IA

Le starter officiel considère **`AGENTS.md` comme la source de vérité** pour les outils IA :

| Fichier | Rôle |
|------|------|
| **`AGENTS.md`** (racine du projet) | Règles globales, flux de démarrage, espaces de noms, injection de dépendances, « ce qu'il ne faut pas faire » |
| **`AGENTS.md`** limité sous `app/`, `migrations/`, `tests/`, etc. | Conseils légers et spécifiques au dossier lorsque vous travaillez dans cette arborescence |
| **`SECURITY.md`** | Secrets, en-têtes, XSS/SQL, signalement—la sécurité reste délibérée et séparée |

Il n'y a **aucun** fichier de style maison distinct pour Copilot / Cursor / Gemini / Windsurf dans le squelette. Pointez votre assistant vers `AGENTS.md` à la racine (et laissez-le suivre les liens vers les fichiers limités). Les humains peuvent ignorer complètement ces fichiers et utiliser le [README](https://github.com/flightphp/skeleton); la structure est la même dans les deux cas.

> **Les docs enseignent les API ; le squelette enseigne la structure.** Les exemples courts de `Flight::` dans ces docs sont parfaits pour apprendre. Dans une application squelette, préférez les classes `App\…`, l'injection de constructeur et `$this->app` plutôt que la façade statique dans les contrôleurs. Voir [Installation](/install) et [Autoloading](/learn/autoloading).

## Utilisation de base

### Configuration des identifiants LLM

La commande `ai:init` vous guide pour connecter votre projet à un fournisseur de LLM.

```bash
php runway ai:init
```

Vous serez invité à :

- Choisir votre fournisseur (OpenAI, Grok, Claude, etc.)
- Saisir votre clé API
- Définir l'URL de base et le nom du modèle

Cela crée les identifiants utilisés pour les requêtes LLM ultérieures (par exemple pour générer des instructions).

**Exemple :**
```
Bienvenue dans AI Init !
Quelle API LLM souhaitez-vous utiliser ? [1] openai, [2] grok, [3] claude : 1
Saisissez l'URL de base de l'API LLM [https://api.openai.com] :
Saisissez votre clé API pour openai : sk-...
Saisissez le nom du modèle que vous souhaitez utiliser (par exemple gpt-4, claude-3-opus, etc.) [gpt-4o] :
Identifiants enregistrés dans .runway-creds.json
```

### Génération d'instructions IA spécifiques au projet

La commande `ai:generate-instructions` crée ou met à jour des instructions pour les assistants de codage IA, adaptées à *votre* projet.

```bash
php runway ai:generate-instructions
```

Vous répondrez à quelques questions (description, base de données, moteur de templates, sécurité, taille de l'équipe, etc.). Flight utilise votre fournisseur de LLM pour générer des instructions et les écrit principalement dans :

- **`AGENTS.md`** à la racine du projet (indépendant de l'outil ; ce que le squelette officiel et la plupart des agents modernes attendent)

Selon la version du CLI et les options, la commande peut également écrire des copies spécifiques à des outils pour les anciens flux de travail (par exemple les fichiers de règles Copilot, Cursor, Windsurf ou Gemini). Pour **les nouveaux projets issus du squelette**, considérez **`AGENTS.md`** (ainsi que les éventuels fichiers `AGENTS.md` limités que vous conservez sous `app/`) comme la source de vérité unique—ne maintenez pas cinq fichiers d'instructions divergents à la main.

**Exemple :**
```
Veuillez décrire à quoi sert votre projet ? Mon API géniale
Quelle base de données prévoyez-vous d'utiliser ? MySQL
Quel moteur de templates HTML prévoyez-vous d'utiliser (le cas échéant) ? twig
La sécurité est-elle un élément important de ce projet ? (o/n) o
...
Instructions IA mises à jour avec succès.
```

Désormais, les outils IA peuvent suggérer du code qui correspond à votre pile technologique et à votre structure réelles—et non à un tutoriel PHP générique.

## Utilisation avancée

- Personnalisez les identifiants ou les chemins de sortie avec les options de commande (voir `--help` sur chaque commande).
- Les assistants fonctionnent avec tout fournisseur LLM qui parle une API compatible OpenAI.
- Relancez `ai:generate-instructions` à mesure que le projet évolue pour que les agents restent synchronisés.
- Dans le squelette, conservez la politique de sécurité dans **`SECURITY.md`** et la structure du code dans **`AGENTS.md`** afin qu'aucun de ces documents ne devienne un fourre-tout.
- Préférez [docs.flightphp.com](https://docs.flightphp.com) et le serveur MCP de Flight lorsque les agents ont besoin de détails sur l'API ; vérifiez les méthodes inventées par rapport à `vendor/flightphp/core`.

## Voir aussi

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Starter officiel avec `AGENTS.md`, Twig, SimplePdo et Dice câblés pour une structure compatible IA
- [Installation](/install) – Disposition recommandée avec `create-project`
- [Autoloading](/learn/autoloading) – La **casse** des dossiers correspond aux espaces de noms (`App\Controller` ↔ `app/Controller/`)
- [Runway CLI](/awesome-plugins/runway) – CLI qui alimente les commandes `ai:*` et de génération de structure
- [Sécurité](/learn/security) – Paramètres sécurisés par défaut que les agents (et les humains) ne devraient pas affaiblir

## Dépannage

- Si vous voyez « Missing .runway-creds.json », exécutez d'abord `php runway ai:init`.
- Assurez-vous que votre clé API est valide et qu'elle a accès au modèle sélectionné.
- Si les instructions ne se mettent pas à jour, vérifiez les permissions des fichiers dans votre répertoire de projet.
- Si un agent invente des API Flight ou une mauvaise structure de dossiers, orientez-le vers **`AGENTS.md`** à la racine et vers ce site de documentation ; la structure du squelette prévaut pour le code sous `app/`.

## Journal des modifications

- v3.18.4 – `ai:generate-instructions` écrit les instructions du projet dans `AGENTS.md` à la racine du projet.
- v3.16.0 – Ajout des commandes CLI `ai:init` et `ai:generate-instructions` pour l'intégration IA.