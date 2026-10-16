# Kapsul : concept, critiques et plan de construction

Kapsul est un launcher Android pour consoles portables (Retroid, AYN, Anbernic, AYANEO…), pensé pour la manette d'abord, le tactile ensuite, et compatible avec les machines à double écran.

Ce document reprend ton idée de départ, la restructure en modèle, puis propose des améliorations argumentées et une architecture pour la construire. La maquette jouable est dans [`prototype/index.html`](../prototype/index.html).

---

## 1. Le modèle

### 1.1 Arborescence

```
Accueil
├── Consoles (carrousel / PSP / grille)
│   ├── Jeux Android          ← toujours en 1re position
│   ├── Switch, PS2, GameCube, PSP, 3DS, DS, GBA…
│   ├── Collections           ← "Tous les Sonic", "Jeux de combat"…
│   └── Toutes les applis     ← petits carrés
├── Liste des jeux d'une console
│   └── Fiche du jeu (boîte, titre, description, année, genre, joueurs, temps de jeu)
└── Options (accessible partout avec Start)
    ├── Affichage : mode du menu, taille des icônes
    ├── Thème : logo sobre, console, personnage, logo couleur
    ├── Fond : préréglages, image importée, vidéo importée
    ├── Sons : effets on/off + volume, musique on/off + volume, sons perso
    ├── Manette : icônes Xbox / Nintendo / PlayStation, bouton de validation
    ├── Émulateurs : émulateur par console, icône perso
    ├── Double écran : rôle du second écran
    └── Infos des jeux : quels champs afficher
```

### 1.2 Les trois modes du menu principal

| Mode | Ce qu'on voit | Pour qui |
|---|---|---|
| **Carrousel** | Gros rectangles, un par console, défilement gauche-droite, nom + infos sous la carte | Mode par défaut, le plus lisible sur 3,5" à 6" |
| **PSP (XMB)** | Rangée d'icônes horizontale, liste des jeux verticale sous l'icône | Nostalgiques, gros catalogues : on lance un jeu sans changer d'écran |
| **Grille** | Petits carrés, plusieurs lignes | Beaucoup de consoles, écrans 4:3 / 1:1 (RG Cube) |

### 1.3 Ce que la maquette montre déjà

- Les 3 modes, les 4 styles de thème, 3 tailles d'icônes.
- Les fonds : teinte qui suit la couleur de la console sélectionnée, préréglages, import d'image ou de vidéo.
- Effets sonores et musique réglables, import de ses propres sons.
- Icônes de boutons Xbox / Nintendo / PlayStation **et**, séparément, le choix du bouton qui valide.
- Collections automatiques (« titre contient Sonic », « genre = Combat »).
- Changement de la boîte d'un jeu (touche X, image importée).
- Double écran avec deux rôles pour l'écran du bas : fiche détaillée ou navigation tactile.

---

## 2. Les améliorations, avec les arguments

Classées par impact. Les 6 premières devraient être dans la v1.

### 2.1 Séparer « icônes des boutons » et « bouton qui valide »

Ton idée « changer A/B Nintendo, Xbox, PlayStation » mélange deux réglages différents :

1. **Le style des icônes** affichées (A vert Xbox, ✕ bleu PlayStation, A/B Nintendo).
2. **La position physique** du bouton qui valide (bas ou droite).

**Argument :** selon les machines, les boutons sont marqués à la Nintendo (A à droite) ou à la Xbox (A en bas), et certaines permettent même d'inverser la disposition dans leurs réglages. Un joueur peut vouloir les icônes PlayStation tout en validant avec le bouton du bas. Si les deux sont liés, une des combinaisons est impossible et l'utilisateur affiche des icônes fausses. La maquette fait déjà la différence : le bouton du bas est toujours « A » côté Xbox, « B » côté Nintendo, « ✕ » côté PlayStation.

**Bonus :** détecter la manette au premier lancement (`InputDevice.getName()`, vendor/product ID) et proposer le bon préréglage. L'utilisateur n'a rien à configurer dans 90 % des cas.

### 2.2 Collections intelligentes plutôt que manuelles

Tu proposes « tous les Sonic, tous les jeux de combat ». À la main, c'est intenable sur 3 000 jeux.

**Proposition :** deux types de collections.
- **Manuelle** : on ajoute un jeu avec un appui long (favoris, « à finir »…).
- **Intelligente** : une règle. `titre contient "Sonic"`, `genre = Combat`, `joueurs ≥ 2`, `année entre 1990 et 1999`, `console ∈ {PS1, PS2}`, combinables en ET/OU.

**Argument :** une collection intelligente se remplit toute seule quand on ajoute des ROMs. Le genre et le nombre de joueurs viennent du scraping (voir 2.4), donc ça ne coûte rien à l'utilisateur. Collections auto à livrer d'office : *Récemment joués*, *Favoris*, *Multijoueur*, *Jamais lancés*, *Les plus joués*.

### 2.3 Une rangée « Reprendre » sur l'accueil

**Proposition :** au-dessus ou en dessous du carrousel, les 5 derniers jeux lancés, toutes consoles confondues, avec la capture du dernier état de sauvegarde si elle existe (RetroArch écrit une `.png` à côté de chaque save state).

**Argument :** c'est l'action la plus fréquente sur une console portable : reprendre ce qu'on faisait. Aujourd'hui il faut 3 à 4 appuis (console → défiler → jeu). Avec la rangée, c'est 1. C'est ce que font la Switch et le Steam Deck, et ce qui manque le plus à ES-DE.

### 2.4 Ne pas réinventer le scraping : lire ce qui existe déjà

Pour « infos du jeu, titre, description, boîtes » :

- **Importer les données ES-DE** (`gamelist.xml` + dossier `downloaded_media`). La plupart des utilisateurs de handheld ont déjà passé des heures à scraper avec ES-DE ou Skraper. Les relire, c'est récupérer leur bibliothèque en 10 secondes.
- **Scraper intégré** via ScreenScraper (gratuit avec compte, le plus complet pour le rétro), IGDB ou TheGamesDB en secours. Identifier par hash CRC/MD5 pour les cartouches, par nom pour les ISO lourds.
- **Édition manuelle** de chaque fiche (titre, description, boîte, genre, année) qui n'est **jamais écrasée** par un nouveau scraping.

**Argument :** le scraping est la partie la plus longue et la plus fragile d'un frontend. En la rendant compatible avec l'existant, tu supprimes la plus grosse barrière à l'adoption : « je ne vais pas tout refaire ».

### 2.5 Choisir son émulateur par console, et par jeu

**Proposition :** chaque console a un émulateur par défaut (avec son icône, remplaçable par ta propre image, comme tu l'as demandé), et chaque jeu peut avoir une exception. Exemple : la PS2 sur NetherSX2, sauf un jeu qui tourne mieux ailleurs ; la GBA sur RetroArch avec le cœur mGBA.

**Argument :** sur Android, aucun émulateur n'est le meilleur pour tous les jeux. ES-DE le permet et c'est une des raisons pour lesquelles les gens y restent. Sans ça, Kapsul serait limité aux joueurs occasionnels.

**Technique :** un fichier de règles de lancement (package, activité, extras de l'Intent) livré avec l'app et mis à jour à part, parce que les émulateurs changent de nom de package (Citra → Lime3DS → Azahar, Yuzu → forks…).

### 2.6 Second écran : des rôles, et savoir s'effacer

Sur une console double écran (AYN Thor, AYANEO Pocket DS…), l'écran du bas doit avoir un **rôle choisi** :

| Rôle | Contenu de l'écran du bas |
|---|---|
| Fiche détaillée | Boîte, description, temps de jeu du jeu sélectionné (dans la maquette) |
| Navigation tactile | Liste ou grille cliquable des éléments affichés en haut (dans la maquette) |
| Vitrine | Vidéo de gameplay ou capture du jeu sélectionné |
| Éteint | Économie de batterie |

**Règle essentielle :** quand on lance un émulateur DS/3DS qui gère lui-même les deux écrans, Kapsul doit **libérer l'écran du bas** immédiatement. Sinon le jeu ne s'affiche que sur un écran.

**Argument :** sans rôle clair, un second écran est soit vide, soit un doublon. Les machines double écran sont précisément celles qu'on achète pour la DS et la 3DS : rater ce cas tue l'intérêt.

### 2.7 Thèmes sous forme de packs partageables

Tes styles « logo sobre, console, personnage, logo de la console » ne devraient pas être codés en dur. **Proposition :** un thème est un dossier zip :

```
mon-theme.kapsul/
├── theme.json           ← couleurs, polices, mode par défaut, sons
├── systems/ps2/
│   ├── logo.svg
│   ├── console.png
│   ├── personnage.png   ← le style "personnage" utilise cette image
│   └── fond.mp4
├── sounds/move.ogg, confirm.ogg, back.ogg
└── music/menu.ogg
```

**Arguments :**
- **Juridique** : les logos et personnages Nintendo, Sony ou Sega sont des marques déposées. Kapsul ne doit pas les livrer. Le pack par défaut utilise des silhouettes génériques (comme la maquette), et la communauté crée les packs « officiels ».
- **Communauté** : c'est ce qui a fait vivre ES-DE, Pegasus et Daijishō. Un format simple et documenté = des dizaines de thèmes gratuits sans que tu les dessines.
- **Ta demande de personnalisation** (fond, sons, icônes) devient un cas particulier : modifier un réglage crée un thème « Perso » qui surcharge le thème actif.

### 2.8 La couleur de la console comme fil conducteur

Dans la maquette, le fond se teinte de la couleur de la console sélectionnée (bleu PS2, rouge Switch, orange Dreamcast) et la boîte de chaque jeu reprend le bandeau de sa console.

**Argument :** c'est un repère qui ne coûte presque rien à l'écran (pas d'image à charger) et qui aide à savoir où on est sans lire. Utile en mode PSP où les icônes sont petites.

### 2.9 Fond vidéo : oui, mais avec des garde-fous

**Proposition :** autoriser la vidéo, mais la mettre en pause pendant le défilement rapide, la couper à 20 % de batterie et la limiter à 720p / 30 i/s.

**Argument :** une vidéo en boucle consomme de la batterie et du processeur. Sur une puce d'entrée de gamme (Unisoc T618, Dimensity 900), elle peut faire saccader le carrousel. Un launcher qui rame, on le désinstalle. Les fonds animés légers (dégradés, vagues comme le mode PSP) donnent 80 % de l'effet pour 5 % du coût.

### 2.10 Sons : par défaut discrets, et un volume séparé du système

**Proposition :** trois curseurs indépendants du volume Android : effets, musique, prévisualisations vidéo. Musique de fond coupée par défaut, pause automatique quand un casque Bluetooth se déconnecte, ou quand une autre app joue de l'audio (gestion du focus audio).

**Argument :** le premier réflexe de beaucoup d'utilisateurs est de couper la musique d'un launcher. Il vaut mieux qu'ils l'activent par choix. Le focus audio évite de jouer par-dessus Spotify ou une vidéo.

### 2.11 Autres idées utiles, en vrac

- **Bouton « jeu au hasard »** (Select sur l'accueil) : très apprécié sur les gros catalogues.
- **Recherche à la manette** avec un clavier en grille et l'autocomplétion. Taper au stick est pénible, il faut 3 lettres max pour trouver un jeu.
- **Masquer les consoles vides** automatiquement, et permettre de réordonner les consoles (par date de sortie, constructeur, ou manuellement).
- **Temps de jeu** suivi par Kapsul pour les ROMs, et via `UsageStatsManager` pour les jeux Android.
- **Mode enfant** : une seule collection visible, Options verrouillées par un code.
- **Sauvegarde/restauration** de toute la config (thème, collections, fiches éditées) dans un fichier, pour changer de console sans tout refaire.
- **RetroAchievements** : afficher les succès d'un jeu sur la fiche (API publique).
- **Jeux Android en grand format** : utiliser la *bannière Android TV* (320×180, déjà au bon format pour les gros rectangles) quand l'app en fournit une, sinon une jaquette scrapée sur IGDB. Les icônes carrées étirées sont la première chose qui fait « cheap ».

### 2.12 Ce que je déconseille ou repousserais

- **Un « Quick Settings » maison** (luminosité, Wi‑Fi, mode performance) : sans root ni droits système, Android limite fortement ce qu'une app peut changer. Mieux vaut un raccourci vers les réglages du constructeur.
- **Inclure des ROMs, BIOS ou liens de téléchargement** : risque juridique et retrait du Play Store immédiat.
- **Trop de modes au lancement** : trois modes bien finis valent mieux que six moyens. Le mode « Grille » peut attendre la v1.1 si le temps manque.

---

## 3. Architecture technique proposée

### 3.1 Choix de la stack

| Besoin | Choix | Pourquoi |
|---|---|---|
| Langage | Kotlin | Standard Android, accès direct aux API système |
| Interface | Jetpack Compose + `androidx.tv` (foundation) | Gestion du focus à la manette déjà faite, animations fluides |
| Base de données | Room (SQLite) | Requêtes rapides sur des milliers de jeux, collections intelligentes = requêtes SQL |
| Réglages | DataStore | Asynchrone, typé |
| Images | Coil | Cache disque et mémoire, redimensionnement |
| Vidéo et audio | Media3 / ExoPlayer, SoundPool pour les effets | Faible latence pour les sons de menu |
| Tâches longues | WorkManager | Scan des ROMs et scraping en arrière-plan |

### 3.2 Modèle de données

```
System      (id, nom, fabricant, année, couleur, dossiers_roms, extensions, emulateur_defaut)
Game        (id, system_id, chemin, hash, titre, description, année, genre, joueurs,
             developpeur, boite, capture, video, temps_de_jeu, dernier_lancement,
             champs_modifies_a_la_main)
Emulator    (id, nom, package, activite, modele_intent, icone_perso)
Collection  (id, nom, type[manuelle|regle], regle_json, icone, ordre)
CollectionGame (collection_id, game_id)
Theme       (id, chemin, actif)
```

`champs_modifies_a_la_main` sert à ne jamais écraser une fiche éditée lors d'un nouveau scraping.

### 3.3 Points délicats côté Android

1. **Être le launcher par défaut** : `intent-filter` avec `CATEGORY_HOME` et `CATEGORY_DEFAULT`. Gérer le bouton Home quand on est déjà sur l'accueil (retour au premier élément).
2. **Accès aux ROMs** : depuis Android 11, le stockage est cloisonné. Utiliser le Storage Access Framework (`ACTION_OPEN_DOCUMENT_TREE`) et passer les fichiers aux émulateurs en `content://` avec `FLAG_GRANT_READ_URI_PERMISSION`. Certains émulateurs exigent un chemin de fichier : le fichier de règles de lancement indique lequel utiliser pour chacun.
3. **Lancer RetroArch** : Intent explicite vers son activité avec les extras `ROM`, `LIBRETRO` (chemin du cœur) et `CONFIGFILE`. Les émulateurs autonomes ont chacun leur format, d'où le fichier de règles (2.5).
4. **Double écran** : `DisplayManager.getDisplays()` pour détecter le second écran, puis soit une `Presentation` (simple, pour la fiche détaillée), soit une seconde activité lancée avec `ActivityOptions.setLaunchDisplayId()` (pour la navigation tactile). Écouter `DisplayListener` pour gérer le branchement/la fermeture du clapet.
5. **Lister les jeux Android** : `PackageManager` + `ApplicationInfo.CATEGORY_GAME`, complété par une liste maintenue (beaucoup de jeux ne déclarent pas la catégorie) et un tri manuel « c'est un jeu / ce n'est pas un jeu ». Depuis Android 11, déclarer les `<queries>` nécessaires.
6. **Performances** : précalculer des miniatures à la taille d'affichage, ne jamais décoder une boîte en pleine résolution dans une liste, viser 60 i/s sur une puce d'entrée de gamme. Tester tôt sur la machine la plus faible visée.

### 3.4 Format des écrans à prévoir

Les handhelds Android vont du 4:3 (Retroid Pocket Flip de 1re génération), au 1:1 (Anbernic RG Cube), au 16:9 (Odin 2, Thor). **Mettre en page en fonction du ratio, pas de la taille en pixels** : le carrousel à 16:9, la grille en 1:1, le mode PSP partout. La maquette utilise déjà des unités relatives à la largeur de l'écran pour cette raison.

---

## 4. Feuille de route

| Version | Contenu | Objectif |
|---|---|---|
| **v0.1** | Launcher HOME, carrousel, scan d'un dossier de ROMs par console, lancement RetroArch + 4 émulateurs autonomes, jeux Android | Utilisable au quotidien par toi |
| **v0.2** | Import ES-DE, fiches de jeu, icônes manette + bouton de validation séparé, sons et musique réglables | Tester avec 5 à 10 personnes |
| **v0.3** | Mode PSP, collections (manuelles + intelligentes), rangée Reprendre, émulateur par jeu | Premières retours publics (Reddit r/SBCGaming, Discord Retro Handhelds) |
| **v0.4** | Double écran (fiche + navigation), fonds image/vidéo, sons perso | Machines double écran |
| **v1.0** | Packs de thèmes, scraper intégré, mode Grille, sauvegarde de config, recherche | Publication Play Store / GitHub |

**Conseil de méthode :** publie tôt une APK sur GitHub et récolte les retours sur une ou deux machines populaires (Retroid Pocket 5, AYN Odin 2) avant d'élargir. Les bugs de manette et d'écran sont propres à chaque appareil : ils ne se découvrent qu'entre les mains des gens.

---

## 5. Comment se démarquer de l'existant

| Launcher | Force | Faiblesse dont Kapsul peut profiter |
|---|---|---|
| ES-DE | Très complet, thèmes, scraping | Interface dense, réglages techniques, payant sur Android |
| Daijishō | Joli, gratuit | Mises à jour irrégulières, configuration des émulateurs pénible |
| Beacon | Simple, moderne | Personnalisation limitée |
| Pegasus | Thèmes très libres | Développement lent, peu accessible |

**Positionnement proposé pour Kapsul :** *aussi simple que Beacon au premier lancement, aussi personnalisable qu'ES-DE quand on creuse, et le seul pensé dès le départ pour le double écran.* Les trois arguments clés à mettre en avant : import ES-DE en un clic, collections intelligentes, double écran.
