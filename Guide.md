# Navigo — Atelier du Web

Prototype 0.1.0 d’un navigateur pédagogique pour les 10–15 ans. Une vraie zone Chromium affiche les sites, tandis que le panneau à gauche accompagne l’élève et valide ses missions. Le passage à la mission suivante est automatique, après un retour positif de 1,8 seconde.

## Utiliser sous Windows

1. Extraire le paquet `Navigo-Windows.zip` dans un dossier accessible en écriture, par exemple sur le Bureau ou une clé USB.
2. Double-cliquer sur `Navigo-0.1.0-portable.exe`. Aucune installation de Node.js ni de navigateur supplémentaire n’est nécessaire.
3. Commencer les dix missions de « Premiers pas sur le Web ». Le parcours fonctionne sans connexion Internet.
4. Pour naviguer sur de vrais sites, saisir une adresse `https://…` dans la barre d’adresse. Une connexion est alors nécessaire.

L’exécutable est Windows x64 et n’est pas signé. Il embarque l’application et Chromium ; le dossier `scenarios` sert uniquement à ajouter vos propres parcours. Le lanceur portable extrait temporairement ses composants lors de son lancement.

Les mots saisis sans adresse ouvrent le petit moteur d’entraînement local. Il recherche uniquement dans la collection intégrée (Belgique, Bruxelles, Atomium, encyclopédie), pas sur le Web public.

## Modifier les exercices sans programmer

1. Cliquer sur **Espace enseignant**. Le parcours actif sert de point de départ.
2. Modifier le titre, l’adresse de départ, les consignes, les indices et les messages de réussite.
3. Choisir une condition : adresse exacte, fragment d’adresse, domaine, titre ou action du navigateur.
4. Ajouter, déplacer ou supprimer les missions.
5. Cliquer sur **Tester ce parcours** pour l’exécuter immédiatement.
6. Revenir à l’espace enseignant et cliquer sur **Exporter YAML** pour enregistrer vos modifications. Le test seul ne sauvegarde pas le fichier.
7. Placer le fichier exporté dans `scenarios`, à côté du `.exe`. Au prochain lancement, il apparaît dans **Parcours**. Il est aussi possible de l’importer directement depuis l’éditeur.

Le fichier exporté se partage avec un autre enseignant. L’importation prend en charge YAML et JSON ; les fichiers invalides sont rejetés avec une explication. Fermer l’éditeur sans exporter ni tester abandonne les modifications en cours.

Pour les actions **Trouver un mot dans la page** et **Lancer une recherche**, le champ facultatif « Texte à rechercher » exige un texte précis, sans tenir compte des majuscules. Pour la recherche dans la page, le texte doit réellement avoir été trouvé.

## Les conditions YAML

```yaml
version: 1
title: Mon premier parcours
startUrl: training://atelier/index.html
steps:
  - instruction: Ouvre la petite encyclopédie.
    hint: Utilise le lien de la page d’accueil.
    success: Tu as suivi un lien vers une nouvelle page.
    validation:
      type: url_equals
      value: training://atelier/encyclopedie.html
  - instruction: Reviens en arrière.
    validation:
      type: browser_action
      action: back
```

| Type | Paramètre | Réussite |
| --- | --- | --- |
| `url_equals` | `value` | Adresse normalisée exactement identique |
| `url_contains` | `value` | Fragment présent dans l’adresse, sensible aux majuscules |
| `domain` | `value` | Domaine exact ou sous-domaine réel ; pas un nom similaire |
| `title_contains` | `value` | Texte présent dans le titre, sans distinction de majuscules |
| `browser_action` | `action` | Action effective du navigateur |
| `all` / `any` | `conditions` | Toutes / au moins une des conditions du même événement |

Actions disponibles : `back`, `forward`, `reload`, `new_tab`, `close_tab`, `switch_tab`, `bookmark`, `find_in_page`, `search`. Pour `find_in_page` et `search`, `value` est un texte attendu facultatif.

Les combinaisons `all` et `any` s’éditent dans le YAML ; l’éditeur graphique les préserve. Elles évaluent un seul événement, pas une suite d’actions mémorisées. Par exemple, `domain` et `url_contains` se combinent sur une navigation. Une condition d’action ne se combine pas avec une condition de page : il faut alors utiliser deux missions successives.

```yaml
validation:
  type: all
  conditions:
    - type: domain
      value: wikipedia.org
    - type: url_contains
      value: /wiki/Belgique
```

Les règles d’adresse constatent l’arrivée sur une page. Elles ne distinguent pas encore un clic sur un lien de la saisie de la même adresse. Une mission d’action nécessite un nouvel événement après son activation : une page déjà visitée ne valide pas automatiquement une nouvelle mission.

## Raccourcis et fonctions

| Raccourci | Fonction |
| --- | --- |
| Ctrl+L | Sélectionner la barre d’adresse |
| Ctrl+T | Nouvel onglet |
| Ctrl+W | Fermer l’onglet actif (le dernier reste ouvert) |
| Ctrl+R / F5 | Actualiser |
| Alt+← / Alt+→ | Précédent / Suivant |
| Ctrl+D | Ajouter / retirer un favori |
| Ctrl+F | Ouvrir la recherche dans la page |
| Échap | Fermer la recherche ou l’espace enseignant |

Les onglets sont limités à 12. L’historique retient les 80 dernières navigations de cette session. Favoris, historique, cookies et progression ne sont pas conservés après fermeture. Recommencer ou tester un parcours ferme les onglets et remet à zéro les favoris et l’historique de l’interface.

## Développer et compiler

Prérequis pour les sources : Node.js 22 ou 24, npm ; Windows 10/11 x64 pour tester la distribution Windows. Le prototype est écrit en JavaScript CommonJS, HTML et CSS, sans serveur de développement supplémentaire.

```sh
npm ci
npm test
npm start
npm run build:win
```

Le résultat est `dist/Navigo-0.1.0-portable.exe`. Sous Linux, `npm run build:linux` crée une application décompressée ; une session graphique et les bibliothèques système Electron sont nécessaires. Compiler Windows depuis Windows est la procédure recommandée pour les versions à distribuer. Le workflow GitHub Actions fourni effectue les tests et la compilation Windows lors d’un lancement manuel ou d’un tag `v*` ; il ne s’active que si vous placez ce projet dans votre propre dépôt GitHub.

Pour reproduire les dépendances, utiliser `npm ci` et conserver `package-lock.json`. Mettre à jour Electron et ses dépendances régulièrement, puis refaire les tests et un essai Windows avant redistribution.

## Structure

- `src/main.js` : fenêtre Electron, vues Chromium, onglets, protocole local et commandes.
- `src/preload.js` : petite passerelle réservée à l’interface de l’application.
- `src/lesson.js` : validation des scénarios, conditions de réussite et progression.
- `src/scenarios.js` : lecture / écriture YAML et contrôles.
- `ui/` : panneau élève et éditeur enseignant.
- `training/` : pages d’entraînement intégrées.
- `scenarios/` : scénario d’exemple.
- `test/` : 16 tests du moteur, de l’interface avec DOM simulé et des commandes avec Electron simulé.

## Vérifications et limites de cette version

Les tests couvrent le parcours complet, les conditions, les domaines trompeurs, les adresses interdites, l’import/export YAML, les scénarios invalides, la création et le déplacement des missions dans l’éditeur et l’affichage des textes sans interprétation HTML.

Le `.exe` a été compilé dans un environnement Linux. Le lancement natif sous Windows et les interactions Chromium réelles (Ctrl+F, liens, onglets, retour/suivant) restent à vérifier sur un poste Windows. Les tests de l’interface utilisent un DOM simulé ; ils ne remplacent pas un test graphique de l’application Electron.

Téléchargements, impression, zoom pédagogique, détection de clics sur des éléments précis, sauvegarde de progression et suivi de classe ne sont pas implémentés. Les téléchargements sont bloqués avec un message. Les demandes de caméra, microphone, notifications et autres permissions sont refusées. Le prototype n’est pas un dispositif de contrôle parental : une adresse HTTP/HTTPS peut être ouverte librement.

Les pages Web s’exécutent dans une session distincte, sans Node.js, sans passerelle Electron et avec isolation de contexte et sandbox. L’interface dispose d’une politique CSP et n’accepte pas de navigation externe. Les scénarios contiennent des données, pas de code exécutable.

## Essai d’acceptation sur Windows

1. Démarrer le `.exe`, vérifier que le panneau gauche et la page d’accueil sont affichés.
2. Effectuer les dix missions jusqu’au message final, notamment la recherche de **Bruxelles** via Ctrl+F puis la recherche de **Atomium** dans la barre d’adresse.
3. Ouvrir l’espace enseignant, modifier un indice et ajouter une mission. Tester, puis exporter le fichier.
4. Importer ce fichier, vérifier le contenu, fermer l’application et placer le YAML dans `scenarios`.
5. Relancer et sélectionner le nouveau parcours. Tester également une adresse HTTPS avec une connexion Internet.

## Références techniques

- Electron WebContentsView : https://www.electronjs.org/docs/latest/api/web-contents-view
- Historique de navigation : https://www.electronjs.org/docs/latest/tutorial/navigation-history
- Sécurité Electron : https://www.electronjs.org/docs/latest/tutorial/security
- Distribution portable : https://www.electron.build/docs/targets/

## Licence

Le code du projet est fourni sous licence MIT. Electron, Chromium et les dépendances conservent leurs licences propres, incluses dans la distribution générée.
