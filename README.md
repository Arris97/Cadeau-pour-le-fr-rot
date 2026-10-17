# Handoff : App cadeau « Sylvain × Kaaris »

## Overview
Une app mobile offerte à Sylvain avec son billet pour le concert de **Kaaris au Stade de France, samedi 22 mai 2027, vers 20h00**. Elle commence par une révélation du cadeau (il ne sait rien), puis donne accès à 6 onglets : Billet (compte à rebours), Jour J, Pads (sampler), Quiz, Son (lecteur + pronos setlist), Cover (générateur de pochette).
Cible : une **PWA** installée sur l'écran d'accueil d'un iPhone ou d'un Android, en plein écran.

## About the Design Files
Les fichiers de `reference/` sont des **références de design créées en HTML** : un prototype qui montre le rendu et le comportement attendus, pas du code de production à copier tel quel. `reference/index.html` tourne sur un petit runtime maison (`support.js`, templates `{{ }}` et classe `Component`). **Ne pas le réutiliser.**
La tâche : **recréer ce prototype dans un vrai projet**. Il n'existe pas encore de codebase. Recommandé : **Vite + React + TypeScript**, en PWA (manifest + service worker, via `vite-plugin-pwa`), déployée en statique (Netlify / Vercel / GitHub Pages).
Pour lancer la référence : `npx serve reference` puis ouvrir `http://localhost:3000` (en dessous de 600 px de large, la vue mobile plein écran s'active).

## Fidelity
**High-fidelity.** Les couleurs, la typo, les espacements, les textes et les interactions sont définitifs. À reproduire au pixel près. Tous les textes sont en français, au ton « street » (argot, private jokes : « K double rotor », « J'ai la vision, j'ai le byakugan », « frérot »).

---

## Design Tokens

**Couleurs**
| Token | Hex | Usage |
|---|---|---|
| `ink` | `#151413` | fond de l'app, texte sur rouge/os |
| `page` | `#0c0b0a` | fond de page desktop autour du téléphone |
| `bone` | `#ebe5d6` | texte principal, cartes « papier » |
| `red` | `#d8321f` | accent unique (tampons, onglet actif, CTA, barres) |
| `pad` | `#1f1d1c` | fond des pads inactifs |
| `bone/60` | `rgba(235,229,214,.6)` | texte secondaire |
| `bone/40` | `rgba(235,229,214,.4)` | bordures des chips inactifs |
| `bone/25` | `rgba(235,229,214,.25)` | séparateurs pointillés, bordures des pads |
| `bone/12` | `rgba(235,229,214,.12)` | pistes des barres de progression |

**Texture** : un grain de scanlines sur le fond, `repeating-linear-gradient(0deg, rgba(255,255,255,.02) 0 1px, transparent 1px 3px)`.

**Typographie** (Google Fonts)
- Display : **Saira Stencil One** 400, toujours en `uppercase`. Titres d'onglet 72 à 80 px / line-height .82 ; hero « KAARIS » 92 px / .8 ; compte à rebours « J-217 » 92 px / .8 ; titres de section 26 à 30 px ; libellés de pads 17 px / .95.
- Texte / UI : **Space Mono** 400/700. Corps 12 à 14 px / 1.45 à 1.55 ; labels 9 à 11 px en 700 uppercase ; onglets de la nav 9.5 px en 700.

**Formes** : aucun border-radius dans l'app (style pochoir). Seuls le cadre du téléphone (52 px, desktop seulement) et la notif de l'écran verrouillé (20 px) sont arrondis. Bordures pleines de 2 px en `bone` ou en `red`. Les éléments « collés » sont légèrement tournés : `rotate(-1.2deg)` à `rotate(1deg)`, tampons à -14° / 12°. Ombre des tampons et blocs : `0 4px 0 rgba(0,0,0,.35)`.

**Espacement** : padding latéral de l'écran 20 px ; gap vertical entre blocs 16 à 22 px ; padding des cartes 12 à 16 px ; grilles en gap 6 à 8 px.

---

## Layout global
- **Mobile (< 600 px)** : l'app occupe `100vw × 100dvh`. Pas de cadre, pas de fausse barre d'état.
- **Desktop** : un panneau d'info de 300 px à gauche, plus un « téléphone » de 390 × 844 px (radius 52, ombre `0 0 0 11px #1f1e1c, 0 0 0 12px #34322f, 0 50px 100px rgba(0,0,0,.6)`) avec une fausse barre d'état (heure réelle, « SDF » et 4 barres de réseau).
- **Zone de contenu** scrollable : `top 0 / bottom 82px`, padding `56px 20px 96px`.
- **Barre d'onglets** fixée en bas, hauteur 82 px, `border-top: 2px bone`, une grille de 6 colonnes avec un gap de 3 px, padding bas `calc(16px + env(safe-area-inset-bottom))`. Onglet actif : fond `red`, texte `ink`. Inactif : texte `bone/60`. Onglets : BILLET, JOUR J, PADS, QUIZ, SON, COVER.
- **Toast** : en haut (top 58), fond rouge, Space Mono 12/700, `rotate(-1deg)`, disparaît après 2 s.
- **Lightbox photo** : plein écran `rgba(21,20,19,.97)`, image en `contain` niveaux de gris, « [ TOUCHE POUR FERMER ] ».
- **Mini-lecteur** : visible quand un vrai morceau joue et qu'on n'est pas sur l'onglet Son. Bloc `bone` à 10 px des bords, 92 px au-dessus du bas : vignette 38 px, titre + artiste + temps, 6 barres d'égaliseur, bouton pause noir 40 × 40, progression rouge de 3 px en bas.

## Screens / Views

### 1. Révélation (overlay plein écran, avant d'entrer dans l'app)
1. **Écran verrouillé** : fond `radial-gradient(circle at 50% 30%, #2a2624, #151413 70%)`. Date du jour (en `fr-FR`, uppercase), heure en Saira 104 px, puis une carte de notif (radius 20, fond `bone/10`, flou d'arrière-plan) : icône carrée rouge « K », « LIVRAISON SPÉCIALE · maintenant », « Sylvain, un colis lourd t'attend. Ouvre pas ça devant tout le monde. ». En bas : « TOUCHE LA NOTIF ». Le tap ouvre le colis. **C'est ce tap qui crée l'AudioContext**, pour respecter la politique autoplay d'iOS.
2. **Colis** : carte de 330 × 440 en `bone`. Textes « EXPÉDITEUR : TON FRÉROT / DESTINATAIRE : SYLVAIN », « FRAGILE ×3 » à 15 % d'opacité, « ⚠ CONTENU : TRÈS LOURD ». Trois bandes de scotch rouges (hauteur 60, plus larges que la carte de 50 px de chaque côté, aux tops 20 / 45 / 70 %, rotations -8° / 6° / -4°) avec le texte « NE PAS OUVRIR · NE PAS OUVRIR ». Au-dessus du colis : « COLIS N° 270 · N SCOTCHS À ARRACHER ».
   - Chaque tap : **son de déchirure de scotch** (voir Audio), le colis tremble (`rotate(-2.5deg) scale(.97)` ou `rotate(1.5deg)`, transition .12 s), et une bande part en translateX ±140 % avec rotate ×3 puis opacité 0 (.5 s, `cubic-bezier(.7,0,.3,1)`).
   - Au 3e scotch : flash blanc (opacité 1 → 0 en 1.1 s), la photo montage apparaît (fondu .9 s, zoom 1.25 → 1.08 en 4 s), dégradé bas, puis « KAARIS · STADE DE FRANCE · 22.05.27 » et « T'Y VAS, SYLVAIN. » (Saira 60). **« Zoo » démarre à 0:19 au volume 0.5.**
3. **Maintenir pour accepter** : un bouton de 64 px de haut avec bordure rouge, rempli de rouge de gauche à droite pendant **1.4 s** d'appui (pointerdown / up / leave). Si on relâche avant, le remplissage repart à zéro. Libellé « MAINTIENS POUR ACCEPTER », puis « ENCORE… ENCORE… » pendant l'appui.
   - À la fin : l'overlay se ferme, on arrive sur l'onglet Billet, et **« Zoo » continue sans coupure**, avec un fondu de volume de 0.5 à 1 (+0.05 toutes les 60 ms). **Ni byakugan ni coupure ici : c'est une demande explicite.**

### 2. Billet
- **Hero** : 400 px de haut, collé en haut de l'écran (marges négatives). Photo montage en niveaux de gris, contraste 1.35, cadrée à 40 % vertical. Dégradé haut et bas vers `ink`. Tampon rouge « STADE DE FRANCE / 22.05.27 » tourné de -4° en haut à gauche ; « N° 270 / SYLVAIN + 1 » en haut à droite ; « KAARIS » en Saira 92 px en bas.
- **Compte à rebours** : bloc rouge tourné de -1.2°. « COMPTE À REBOURS / 20H00 · SAINT-DENIS », « J-{jours} » en Saira 92, `hh:mm:ss` en Space Mono 26/700 (les secondes en `bone`). Barre de progression depuis la date du cadeau (17/10/2026) jusqu'au concert, libellés « CADEAU OUVERT · x% DU CHEMIN · LE 22 ».
  - En **mode Jour J** : « MODE JOUR J ACTIVÉ / C'EST CE SOIR. / Prochaine étape : … ».
- **Carte billet** : bordure `bone` de 2 px, reflet rouge (`radial-gradient` à 35 % d'opacité) qui suit le pointeur. Champs TITULAIRE SYLVAIN / OFFERT PAR TON FRÉROT / LIEU STADE DE FRANCE / PORTE-PLACE SUR LE BILLET, un faux code-barres (`repeating-linear-gradient`) et « 2·7·0 / 93-SDF ».
- Ligne finale : « K DOUBLE ROTOR » + bouton « PADS → ».

### 3. Jour J
- **Programme** sur une ligne pointillée rouge à gauche, avec un point carré par étape :
  - 14:00 Pré-chauffe
  - 16:30 Départ
  - 18:00 Portes (horaire à confirmer)
  - 20:00 Kaaris (« Le show. J'ai la vision. »)
  - 23:30 Retour

  En mode Jour J, l'étape en cours a son point rouge et le badge « MAINTENANT », et les étapes passées sont à 40 % d'opacité.
- **Transports** : RER D Gare Stade de France–Saint-Denis · RER B La Plaine–Stade de France · Métro 13 Saint-Denis–Porte de Paris. Badge carré de 38 px.
- **Check-list « DANS LE SAC »** : bandeau `bone` tourné de 1°, compteur n/7, barre de progression. Éléments : Le billet (app chargée à bloc), Carte d'identité, Batterie externe, Ticket RER / Navigo, Bouchons d'oreilles (ou pas), La tenue du soir, Les yeux en mode Byakugan. Case de 26 px avec un ✕ rouge, texte barré à 40 % d'opacité quand c'est coché. Tout coché : tampon « PRÊT POUR LE 93 » (bordure rouge de 4 px, -14°), toast « SAC BOUCLÉ. T'ES PRÊT POUR LE 93. » et l'extrait du drop de Zoo (2.5 s).

### 4. Pads (« SAMPLER »)
- Grille 3×3 de pads carrés. Chacun affiche son numéro (01 à 09), son libellé (Saira 17) et une méta « SOURCE · m:ss.s ». Le pad 01 a un fond `bone`, les autres un fond `pad`. Au tap (pointerdown) : fond rouge et `scale(.93)` pendant 180 ms. Une barre rouge de 5 px en bas du pad avance pendant la lecture.
- Compteur « SAMPLES BALANCÉS » avec un message selon le palier : moins de 10 « Échauffement. », moins de 50 « Ça chauffe dans le salon. », moins de 150 « Les voisins appellent. », au-delà « T'es prêt pour la fosse. ».
- **Mode ÉDITER** (bouton en haut à droite) : taper un pad le sélectionne (bordure rouge de 4 px). Le panneau d'édition contient :
  - les sources ZOO / VISION / TCHOIN ;
  - le nom (24 caractères max) ;
  - une **forme d'onde** de 110 barres (la zone du sample en rouge), cliquable pour placer le début ;
  - DÉBUT / DURÉE ;
  - les boutons −1s / −0.1 / +0.1 / +1s (chacun rejoue l'extrait) ;
  - un curseur de durée de 0.3 à 8 s ;
  - les boutons ▶ ÉCOUTER et RÉINITIALISER.

  Les réglages sont sauvegardés.
- **Pads par défaut** (source, début en s, durée en s) :
  | # | Libellé | Source | Début | Durée |
  |---|---|---|---|---|
  | 01 | J'ai la vision | vision | 80.8 | 3.6 | *(+ overlay byakugan)*
  | 02 | Zoo · le drop | zoo | 38.6 | 4 |
  | 03 | Zoo · la montée | zoo | 19 | 4 |
  | 04 | Vision · le beat | vision | 28 | 4 |
  | 05 | Zoo · relance | zoo | 84.3 | 3 |
  | 06 | Tchoin · refrain | tchoin | 31.8 | 4 |
  | 07 | Zoo · après le break | zoo | 144.9 | 4 |
  | 08 | Tchoin · 1:04 | tchoin | 64.2 | 4 |
  | 09 | Tchoin · relance | tchoin | 136.4 | 4 |
  Ces points ont été trouvés en analysant le volume des morceaux, pas les paroles. Seul le 01 a été validé par l'utilisateur.
- **Overlay byakugan** (pad 01, quiz parfait, pronos verrouillés ; jamais pendant la révélation) :
  - fond `radial-gradient(circle at 50% 42%, #d8321f, #5a140c 45%, #151413 80%)` ;
  - grand disque de rayons (`repeating-conic-gradient`) qui tourne de 40° en 3.5 s ;
  - œil de 210 px (anneaux `bone`, pupille, veines rouges en conic-gradient) qui passe de `scale(.4)` à `.9` puis `1.12` ;
  - « J'AI LA VISION », puis à 1.5 s « J'AI LE BYAKUGAN » (sur fond `bone`, apparaît en passant de `scale(1.4)` à `scale(1) rotate(-2deg)`) ;
  - durée = celle du sample ; `pointer-events: none`.

### 5. Quiz (« VRAI OU TOURISTE ? »)
- **Intro** : 6 questions, 12 s chacune, meilleur rang, bouton « C'EST PARTI ».
- **Question** : « Q n/6 · SÉRIE ×n · Xs », barre de temps (elle passe au rouge sous 4 s), question en Saira 34, 3 réponses en bordure de 2 px. Après la réponse : la bonne passe en fond rouge avec ✓, la mauvaise choisie est barrée avec ✕, les autres tombent à 30 % d'opacité. Sans réponse à 12 s : « Trop lent ! » et la série repart à zéro. Retour : « Validé. » (« Ça rotor ! » à partir de 3 bonnes réponses d'affilée) ou « Raté, frérot. ». Bouton SUIVANTE →, puis VERDICT →. Une bonne réponse joue 0.9 s du drop de Zoo.
- **Questions** (index de la bonne réponse) :
  1. Kaaris a grandi dans quelle ville du 9-3 ? Aulnay / **Sevran** / Bobigny
  2. « Or Noir », l'album qui a tout lancé, sort en… 2011 / **2013** / 2016
  3. « 2.7.0 », ça renvoie à quoi ? Son âge / **Le code postal de Sevran** / Une date de sortie
  4. Il est né où ? **Côte d'Ivoire** / Sénégal / Cameroun
  5. Il explose en featuring sur « Kalash » avec… **Booba** / Rohff / La Fouine
  6. Le tube club de 2017, c'est… **Tchoin** / Zoo / Binks

  *À faire relire par l'utilisateur.*
- **Verdict** : tampon rouge à -5°.
  - 6/6 « K DOUBLE ROTOR »
  - 5 « VRAI DE SEVRAN »
  - 3 ou 4 « FAN DU 93 »
  - 0 à 2 « TOURISTE »

  Plus : score, meilleure série, une phrase par rang et REJOUER. On garde le meilleur rang. Un 6/6 déclenche le byakugan.

### 6. Son
- **Lecteur** (carte `bone`) : vignette 64 px, « EN ÉCOUTE / EN PAUSE », titre en Saira 28, album, **égaliseur de 28 barres** (une sur 4 en rouge) piloté par un `AnalyserNode` (fftSize 64) sur les vrais morceaux, une barre de progression **cliquable pour avancer ou reculer**, le temps écoulé et la durée (en partant de l'offset), et les boutons ◀◀ / ▶ ou ❚❚ / ▶▶.
- **Pistes** :
  | Titre | Album / artiste | Vrai | Fichier | Offset |
  |---|---|---|---|---|
  | Zoo | Or Noir · 2013 | oui | audio/zoo.mp3 | 0 s |
  | Vision | La Fouine & Kaaris | oui | audio/vision.mp3 | **20 s** (l'intro n'est pas pertinente) |
  | Tchoin | Dozo · 2017 | oui | audio/tchoin.mp3 | 0 s |
  | Binks, Paradis ou enfer, Bling Bling | — | non (fausse lecture de 30 s) | — | — |
  Chaque ligne a aussi un lien « SPOTIFY ↗ » vers `https://open.spotify.com/search/Kaaris%20<titre>`.
- **Pronos setlist** (carte avec bordure de 2 px) : un son d'ouverture (choix unique), « LES 3 SÛRS » (3 choix max), « COMBIEN DE « J'AI LA VISION » ? » (curseur de 0 à 60). Bouton « VERROUILLER MES PRONOS » :
  - il faut une ouverture et 3 sons, sinon toast « CHOISIS L'OUVERTURE + 3 SONS AVANT DE VERROUILLER. » ;
  - une fois verrouillé : tampon « VERROUILLÉ » avec la date (12°), les contrôles sont figés, toast et byakugan ;
  - le même bouton sert à déverrouiller.

### 7. Cover (« TA POCHETTE »)
- **Pochette carrée** : photo montage en `mix-blend-mode` sur une couleur de fond selon le filtre :
  - **OS** : fond `bone`, `grayscale(1) contrast(1.35)`, multiply, texte `ink`.
  - **ROUGE** (par défaut) : fond `red`, `grayscale(1) contrast(1.5)`, multiply, texte `bone`.
  - **NÉGATIF** : fond `ink`, `grayscale(1) invert(1) contrast(1.25)`, normal, texte `red`.
- Le titre est saisi par l'utilisateur, en majuscules, 22 caractères max (par défaut « LES 3 DU 270 »), en Saira 58 en haut. En bas à gauche : « KAARIS × SYLVAIN × FRÉROT / LIVE · STADE DE FRANCE · 22.05.27 ». En bas à droite : une étiquette « ATTENTION / SON TRÈS FORT ».
- Bouton « TITRE AU HASARD ↻ ». Titres possibles : Les 3 du 270, Or Sylvain, Fosse 93, K double rotor, Dozo Sylvain, Le bruit du stade, Saint-Denis Club, Trio Sevran.
- **Galerie** : défilement horizontal de 10 photos de 130 × 170, en polaroïd `bone` avec un padding de 5 px, rotations alternées de -2° à 2°. Un tap ouvre la lightbox.
- *Amélioration conseillée :* un export PNG de la pochette (html-to-image ou canvas) avec partage via `navigator.share`.

---

## Audio (point critique)
- **Déverrouillage iOS** : créer et `resume()` l'`AudioContext` au premier tap, sur la notif de l'écran verrouillé.
- **Lecteur principal** : un seul `HTMLAudioElement` dont on change la `src`, branché sur `createMediaElementSource → AnalyserNode → destination`. Ne poser `currentTime` qu'après `loadedmetadata`.
- **Samples** : un `HTMLAudioElement` préchargé par source (zoo / vision / tchoin). Un sample se lance avec seek puis `play()`, et `pause()` après sa durée. Un seul sample à la fois. Pendant un sample, le lecteur principal baisse à 0.12 puis revient à 1.
  - *Mieux :* décoder les 3 MP3 en `AudioBuffer` et jouer les samples avec `AudioBufferSourceNode.start(0, offset, duration)` : c'est sans latence, et plusieurs pads peuvent se superposer.
- **Son de déchirure de scotch** (synthétisé) :
  - bruit blanc de 0.38 à 0.5 s, enveloppe `(1-x)^1.4`, attaque de 2 % ;
  - craquements : 20 % des échantillons à amplitude 1, les autres à 0.22 ;
  - `playbackRate` aléatoire entre 0.85 et 1.15 ;
  - passe-bande Q 1.1, balayé de 700 à 3600 Hz, puis passe-haut 350 Hz, gain 1.6.
- **Sons du reste de l'app** : uniquement des extraits des vrais morceaux. **Aucune voix de synthèse, aucun bruitage inventé** (refusés par l'utilisateur), sauf le son de scotch.
- **Tweak « son »** : coupe tout le son (mute global).

## State Management
- **Révélation** : `phase` ('lock' | 'box'), `tapes` (0 à 3), `shake`, `white`, `holdAt` (timestamp), `revealed`.
- **Navigation** : `tab`, `toast`, `photo` (lightbox).
- **Jour J** : `checks: Record<number, boolean>`.
- **Quiz** : `{mode: 'intro'|'q'|'end', i, t0, ans, score, streak, bestStreak}`, `best` (rang).
- **Lecteur** : `playing`, `cur`, `prog` (pour les fausses pistes).
- **Pronos** : `opener`, `picks[]`, `guess`, `locked`, `lockedAt`.
- **Pads** : `pads[]` ({label, src, start, len, eye}), `editing`, `sel`, `padPlay {i, t0, len}`, `hits`.
- **Cover** : `coverTitle`, `coverFx`.
- **Persistance** dans `localStorage` (clé `kaaris-sylvain-v5`) : revealed, checks, picks, opener, guess, locked, lockedAt, coverTitle, coverFx, hits, best, pads.
- **Horloge** : un tick toutes les 100 ms (compte à rebours, timer du quiz, appui long, égaliseur). On peut le remplacer par `requestAnimationFrame` pour l'égaliseur.
- **Constantes** :
  - date du concert : `2027-05-22T20:00:00+02:00` ;
  - date du cadeau : `2026-10-17` ;
  - mode Jour J (prop / flag de debug) : compte à rebours forcé à « C'EST CE SOIR » et étape courante calculée selon l'heure ;
  - `startRevealed` (debug) : on saute la révélation.

## Assets
- `reference/uploads/Gemini_Generated_Image_kxg3ockxg3ockxg3.jpeg` : **montage principal** (Sylvain, son ami et Kaaris devant le « 270 » du Stade de France). Sert au hero, à la révélation, au lecteur et à la pochette.
- `reference/uploads/photos-*.jpg` (9) : visuels promo de Kaaris, pour la galerie.
- `reference/audio/zoo.mp3`, `vision.mp3`, `tchoin.mp3` : les morceaux. **Usage privé uniquement** (cadeau). Ne pas publier sur un dépôt ou un site public.
- `reference/audio/waveforms.json` : `{zoo|vision|tchoin: {dur, wf: number[600]}}`, l'enveloppe RMS normalisée qui sert à dessiner l'onde dans l'éditeur. On peut la recalculer au build ou au runtime avec `decodeAudioData`.
- `reference/icon.png` : icône 512 px (montage + bandeau rouge « 22.05.27 »), à utiliser comme apple-touch-icon et icône du manifest.

## PWA / déploiement
- Manifest : `name` « Sylvain × Kaaris », `display: standalone`, `background_color` et `theme_color` `#151413`, `orientation: portrait`.
- Balises meta iOS : `apple-mobile-web-app-capable`, `status-bar-style: black-translucent`, `viewport-fit=cover`. Utiliser `env(safe-area-inset-*)`.
- Service worker : mettre en cache l'app, les images et les MP3 pour que l'app marche hors-ligne le jour J (le réseau au Stade est saturé).
- Déployer sur un **lien privé ou non indexé**.

## Files
- `reference/index.html` : le prototype complet. Le template HTML contient tous les styles en inline. La logique se trouve dans `<script data-dc-script>`, dans la classe `Component` : `renderVals`, `sample`, `rip`, `editor`, `player`, `finishReveal`.
- `reference/support.js` : le runtime du prototype. **Ne pas le porter.**
