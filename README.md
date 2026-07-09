# am — Apple Music CLI

Pilote Music.app depuis le terminal, via JXA (`osascript`). Zéro dépendance, un seul fichier, exécuté par [Bun](https://bun.sh).

## Installation

```sh
ln -sf "$PWD/am" ~/.local/bin/am
```

À la première utilisation, macOS demande d'autoriser le terminal à contrôler Musique
(Réglages Système → Confidentialité et sécurité → Automatisation).

## Commandes

### Lecture

| Commande | Effet |
|---|---|
| `am` / `am now` | Morceau en cours (titre, artiste, position, ♥) |
| `am play` / `am pause` / `am toggle` | Contrôle de la lecture |
| `am next` / `am prev` | Changer de morceau |
| `am volume [0-100]` | Afficher ou régler le volume |
| `am shuffle [on\|off]` | Mode aléatoire |
| `am repeat [off\|one\|all]` | Mode répétition |

### Bibliothèque

| Commande | Effet |
|---|---|
| `am search <termes> [--limit N]` | Chercher dans la bibliothèque |
| `am recent [--limit N]` | Derniers ajouts |
| `am stats` | Nombre de morceaux, artistes, albums, heures |
| `am love` / `am unlove` | Favori sur le morceau en cours |
| `am rate <0-5>` | Noter le morceau en cours |

### Cloud (catalogue Apple Music)

| Commande | Effet |
|---|---|
| `am catalog <termes> [--limit N]` | Chercher dans le catalogue Apple Music |
| `am cloud-add <termes \| #id>` | Ajouter un morceau du catalogue à la bibliothèque cloud |
| `am delete <termes>` | Supprimer un morceau de la bibliothèque (propagé au cloud si « Synchroniser la bibliothèque » est activé) |
| `am remove <termes> --from <playlist>` | Retirer un morceau d'une playlist |
| `am login [token]` | Enregistrer le Media User Token / afficher l'état |

La lecture du catalogue (`catalog`) fonctionne sans compte : le token développeur anonyme
de music.apple.com est récupéré et mis en cache automatiquement (`~/.config/am/config.json`).

Les **écritures cloud** (`cloud-add`) demandent ton *Media User Token* :

1. Ouvre <https://music.apple.com> et connecte-toi
2. Outils de développement (⌥⌘I) → Application/Stockage → Cookies → music.apple.com
3. Copie la valeur du cookie `media-user-token`
4. `am login '<token>'`

### Playlists

| Commande | Effet |
|---|---|
| `am playlists` | Lister les playlists |
| `am playlist <nom>` | Contenu d'une playlist |
| `am new-playlist <nom>` | Créer une playlist |
| `am add <termes> --to <playlist>` | Ajouter le premier résultat de recherche à une playlist |

## Exemples

```sh
am search amr diab --limit 5
am add wala ala balo --to Car
am love
am recent
```

## Limites connues

- `search`/`add` opèrent sur la bibliothèque **locale** (= la bibliothèque cloud si « Synchroniser la bibliothèque » est activé) ; `catalog`/`cloud-add` passent par l'API Apple Music.
- `add` et `cloud-add` prennent le **premier** résultat de recherche ; affine les termes ou passe un `#id` de `am catalog`.
- L'API Apple Music publique ne permet pas de **supprimer** un morceau de la bibliothèque cloud — `am delete` passe par Music.app, dont la suppression se synchronise au cloud.
- Le `media-user-token` expire au bout de quelques mois ; refais `am login` si les écritures renvoient 403.
