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

- `search`/`add` opèrent sur la **bibliothèque locale** uniquement — le scripting de Music.app n'expose pas la recherche dans le catalogue Apple Music.
- `add` prend le **premier** résultat de recherche ; affine les termes si besoin.
