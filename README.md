# Le Petit Énarque — dépôt de contenu

Ce dépôt contient tout le **contenu** (questions, cartes de révision, badges,
quêtes) de l'application Android privée *Le Petit Énarque*, un jeu de
révision pour préparer les concours (INSP). L'application le lit directement
ici via `raw.githubusercontent.com` à chaque lancement : modifier ces
fichiers et les pousser sur `main` suffit à mettre à jour l'app installée,
sans recompiler ni réinstaller quoi que ce soit.

Le dépôt est public mais n'est pas un projet open source à proprement
parler : c'est juste le moyen le plus simple de servir du JSON statique
gratuitement à une seule application privée.

## Comment la synchronisation fonctionne

Au démarrage, l'app télécharge `manifest.json` et compare son champ
`version` à la dernière version connue localement. Si elle a augmenté, l'app
retélécharge tous les fichiers listés dans `manifest.json` (thèmes de quiz,
decks de cartes) puis met à jour son cache local. En cas d'échec réseau,
elle retombe silencieusement sur le contenu déjà en cache (ou celui fourni
avec l'APK). Le champ `gamification_version` fonctionne pareil mais
uniquement pour `gamification/badges.json` et `gamification/quests.json`.

**Règle d'or : toute modification de contenu doit s'accompagner d'un
incrément de `version` (et/ou `gamification_version`) dans `manifest.json`,
sinon l'app ne la verra jamais.**

## Structure

```
manifest.json                  Index : thèmes/decks disponibles + versions
topics/<id>.json                Un thème de quiz (liste de questions)
flashcards/<id>.json            Un deck de cartes de révision
gamification/badges.json        Toutes les définitions de badges
gamification/quests.json        Toutes les définitions de quêtes
```

### `manifest.json`

```json
{
  "version": 8,
  "topics": ["constitution", "ct", "..."],
  "flashcard_decks": ["arrets", "constitution", "..."],
  "gamification_version": 8
}
```

- `topics` / `flashcard_decks` : ids qui doivent correspondre exactement à
  un fichier `topics/<id>.json` / `flashcards/<id>.json`.
- Ajouter un nouveau thème = créer le fichier JSON + l'ajouter à la liste +
  incrémenter `version`.

### `topics/<id>.json` — un thème de quiz

```json
{
  "id": "constitution",
  "title": "Droit constitutionnel",
  "description": "...",
  "questions": [ /* voir types ci-dessous */ ]
}
```

Quatre types de questions, distingués par le champ `type` :

| Type | Champs | Notes |
|---|---|---|
| `mcq` | `choices` (liste), `correctIndex` | QCM classique |
| `true_false` | `correctAnswer` (bool) | Vrai/Faux |
| `open` | `answer` (texte) | Réponse affichée, l'utilisateur s'auto-évalue ("j'avais juste" / "je me suis trompé·e") — pas de correction automatique |
| `fill_blank` | `answer`, `acceptedAnswers` (liste, optionnelle) | Réponse tapée au clavier, corrigée automatiquement après normalisation (minuscules, accents et ponctuation ignorés) |

Champ commun optionnel : `explanation` (affiché après la réponse).

### `flashcards/<id>.json` — un deck de cartes de révision

```json
{
  "id": "constitution",
  "title": "Droit constitutionnel",
  "description": "...",
  "cards": [
    { "id": "card_1", "front": "Question / recto", "back": "Réponse / verso" }
  ]
}
```

### `gamification/badges.json` — badges

Chaque badge a une `condition` (et, optionnellement, une `revokeCondition`
qui fait perdre le badge s'il devient vraie une fois le badge obtenu).
Évaluées après chaque réponse à une question.

| Type de condition | Champs | Sens |
|---|---|---|
| `volume` | `topic` (optionnel), `count` | N questions répondues (au total, ou sur un thème donné si `topic` est précisé) |
| `streak` | `topic` (optionnel), `count` | N bonnes réponses d'affilée |
| `wrong_streak` | `topic` (optionnel), `count` | N mauvaises réponses d'affilée (badges "d'échec") |
| `accuracy_above` / `accuracy_below` | `topic` (optionnel), `window`, `threshold` | Taux de réussite sur les N dernières réponses (`window`), au-dessus/en-dessous du seuil |
| `all_of` | `conditions` (liste) | Toutes les sous-conditions doivent être vraies (badges croisés, plusieurs thèmes à la fois) |

Omettre `topic` rend la condition globale (tous thèmes confondus) plutôt que
limitée à un seul.

### `gamification/quests.json` — quêtes

Quêtes journalières/hebdomadaires, structure plus simple (`target` avec
`type: "answers_count"`, `period` et `count`).

## Ajouter un nouveau thème, étape par étape

1. Créer `topics/<id>.json` avec au moins quelques questions.
2. (Optionnel mais recommandé) créer `flashcards/<id>.json` en pendant.
3. Ajouter `<id>` aux tableaux `topics` / `flashcard_decks` de
   `manifest.json`.
4. Incrémenter `manifest.json.version`.
5. Committer et pousser sur `main`.
6. Ouvrir l'app (avec une connexion réseau) : le nouveau contenu apparaît
   automatiquement, sans mise à jour de l'APK.

Ajouter des badges/quêtes suit le même principe, en incrémentant
`gamification_version` à la place.
