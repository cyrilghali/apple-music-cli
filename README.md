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

### Playlists par IA (POC)

| Commande | Effet |
|---|---|
| `am mix "<description>" [--limit N] [--name <nom>] [--dry-run]` | Décris la playlist que tu veux ; l'IA génère la tracklist, la résout contre le catalogue Apple Music et la crée dans ta bibliothèque cloud |

```sh
am mix "du funk égyptien des années 70 pour cuisiner" --limit 15
am mix "montée progressive pour courir 45 minutes" --dry-run
```

Nécessite le Media User Token (`am login`) et une clé API Anthropic
(`export ANTHROPIC_API_KEY='sk-ant-...'`). Le SDK `@anthropic-ai/sdk` est
auto-installé par Bun au premier lancement — le CLI reste un seul fichier.
`--dry-run` génère et résout sans rien créer.

### Export Spotify

| Commande | Effet |
|---|---|
| `am spotify-login [client-id]` | Connecter ton compte Spotify / afficher l'état |
| `am spotify <playlist> [--name <nom>] [--private] [--dry-run]` | Exporter une playlist Apple Music vers Spotify |
| `am spotify-library [--liked-only\|--playlists-only] [--public] [--dry-run]` | Migrer toute la bibliothèque : titres likés + playlists |
| `am spotify-mix "<description>" [--limit N] [--name <nom>] [--public] [--dry-run]` | `am mix`, mais la playlist est créée sur Spotify (même clé Anthropic) |

Configuration unique (compte Spotify Premium requis depuis février 2026) :

1. Va sur <https://developer.spotify.com/dashboard> → « Create app »
2. Redirect URI **exactement** `http://127.0.0.1:8888/callback` (Spotify n'accepte plus `localhost`), coche « Web API »
3. Copie le **Client ID**
4. `am spotify-login <client-id>` — une fenêtre s'ouvre pour autoriser l'accès

```sh
am spotify "Funk Egyptien 70s"
am spotify "Funk Egyptien 70s" --name "My Funk Playlist" --private --dry-run
am spotify-library --dry-run   # apparie tout, n'écrit rien
am spotify-library             # like chaque morceau de la bibliothèque, recrée chaque playlist (privée)
```

L'export cherche chaque morceau par titre + artiste (puis titre simplifié + artiste principal), l'album départage les doublons. Quelques pistes peuvent manquer si elles ne sont pas disponibles sur Spotify : `✗` dans la sortie de `am spotify`, fichier `~/.config/am/spotify-unmatched.txt` pour `am spotify-library`.

`am spotify-library` est relançable : les appariements sont mémorisés dans `~/.config/am/spotify-matches.json`, liker un titre déjà liké est sans effet et une playlist portant déjà le même nom sur Spotify est ignorée (ou complétée si elle est restée à moitié remplie). Les titres sont likés du plus ancien au plus récent pour retrouver l'ordre d'Apple Music.

Les apps Spotify en Development Mode ont un quota journalier de recherches (de l'ordre de 600 morceaux par 24 h, puis `429` avec un `Retry-After` d'une journée). Quand il tombe, la commande enregistre où elle en est et s'arrête : relance-la le lendemain, ou planifie-la chaque soir jusqu'à ce qu'elle annonce la fin.

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
