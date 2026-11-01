# re-play sur Mac : préparation et essais

Préparé sous Windows le 04/10/2026, **pas encore essayé sur un Mac**. À dérouler sur le Mac, dans l'ordre.

## Construire
```bash
cd "<dossier de re-play>/frontend_ui"
./build_mac.sh
```
- Compile dans `~/Builds/re-play` (pas sur le NAS) : backend C++ (facultatif), icône `re-play.icns` (sips + iconutil,
  depuis `replay_final.png`), application `re-play.app`, installeur `.dmg`.
- L'application déclare les liens `replay://` et les formats (Info.plist, `re-play.spec`), puis `lsregister` les inscrit.
- Application non signée : premier lancement par clic droit > Ouvrir.
- **Variante testeur** (paquet à envoyer hors du studio, sans profil ni extension du studio) : `./build_mac.sh --testeur`
  (ou `REPLAY_VARIANTE=testeur ./build_mac.sh`). Le script lance `tests/test_paquet_testeur.py` avant, pose la date et la
  variante pour « À propos », vérifie le `.app` avec `tools/verifier_paquet.py` et écrit `SHA256SUMS.txt`. Non essayé.
- Passation pour une session Claude Code sur le Mac : `docs/HANDOFF_MAC.md`.

## Ce qui est prêt
| Sujet | Où | Comment |
|---|---|---|
| Liens `replay://` | `re-play.spec` (CFBundleURLTypes) | macOS les envoie à l'app (QFileOpenEvent) → `lancement.FiltreOuverture` |
| Ouvrir avec / glisser sur le Dock | `re-play.spec` (CFBundleDocumentTypes) | re-play se propose pour exr, dpx, mov, mp4…, et ouvre `.repl` |
| Fenêtre unique | `lancement.py` | même mécanisme que Windows (QLocalServer) ; `open -a` passe par QFileOpenEvent |
| Prism | plugin CEG | « Ouvrir dans re-play » → `open -a /Applications/re-play.app …`, sinon `python3 main.py` depuis le NAS |
| Profils de studio, connecteurs | `re-play.spec` | embarqués (ils manquaient aussi dans un exe Windows) |
| Chemins NAS | `plateforme.py` | correspondances de lecteurs Windows ↔ `/Volumes/<partage>` |
| Préférences > Studio > Liens | `studio_prefs.py` | « Actif : déclaré par l'application » + « Réenregistrer » |

## À essayer (cocher)
- [ ] `./build_mac.sh` va au bout ; l'icône est la bonne dans le Dock.
- [ ] Ouvrir un `.mov`, un `.exr` de séquence et un `.repl` par double-clic / « Ouvrir avec ».
- [ ] re-play ouvert : glisser un 2e média sur l'icône du Dock → il arrive dans **la même fenêtre**.
- [ ] Terminal : `open "replay://open?p=/Volumes/<partage>/PROJETS/ALPHA/…/plan.mov"` → s'ouvre dans re-play.
- [ ] Safari / Chrome : un lien `replay://` → question « Ouvrir re-play ? » puis ouverture.
- [ ] Prism (Mac) : clic droit sur une version → « Ouvrir dans re-play ».
- [ ] Viewport OpenGL 3.3 core (Apple Silicon, GL 4.1 max) : image, volet, différence, vue grain.
- [ ] Lecture : vidéo 10 bits, EXR, son ; transcodage (ffmpeg doit être dans le PATH ou dans l'app).
- [ ] Kitsu : badge « Kitsu · … », « Qui suis-je… » (trousseau macOS pour les secrets).
- [ ] Raccourcis : Ctrl de Windows = Cmd sur Mac (Qt les traduit) ; vérifier E/G/S/N + glisser.

## Reste à faire après l'essai
- Signature et notarisation Apple (sinon l'alerte « développeur non identifié » à chaque nouvelle machine).
- ffmpeg / ffprobe embarqués dans l'app (aujourd'hui : ceux du système).

## Premier essai depuis les sources (30/10/2026, iMac Apple M1, macOS 26.6.2)

Fait par une session Claude Code sur le Mac, branche `feat-mac`. Ce qui a été installé, dans cet ordre :

1. `uv` pour l'utilisateur (`python3 -m pip install --user uv`, sans mot de passe), puis Python 3.14 :
   `uv python install 3.14` et un environnement à part : `uv venv --python 3.14 ~/re-play-venv`.
2. Dépendances dans ce venv : `PySide6==6.11.0 OpenImageIO opencv-python numpy PyOpenGL PyOpenGL_accelerate opencolorio gazu
   keyring pybind11 cmake openpyxl ruff pyinstaller pillow "mcp<2"` (`pillow` : pack de démo ; `mcp<2` : outil de dev, la 2.x a
   renommé FastMCP).
3. PyAV LGPL (`docs/PYAV_LGPL.md` § 4) : `av @ …/v18.1.0-lgpl-2/av-18.1.0-cp311-abi3-macosx_14_0_arm64.whl#sha256=24393e72…61c34` ;
   `tools/verifier_paquet.py --av` : OK, aucun x264 / x265 dans `av/.dylibs`.
4. ffmpeg : Homebrew (`brew install ffmpeg`, ffmpeg 9.0.2, version complète GPL : bien pour travailler, **à ne pas embarquer**).
5. Module C++ : `PATH=~/re-play-venv/bin:$PATH bash backend_cpp/build_mac.sh` : compile sans erreur
   (`light_player_core.cpython-314-darwin.so`), trouvé par `engine.py` (le cache C++ reste désactivé par `cache_cpp_utilisable`, comme sous Windows).

Lancer : `PATH=/opt/homebrew/bin:$PATH ~/re-play-venv/bin/python frontend_ui/main.py`.

Résultat :
- Diagnostic : OpenGL 4.1 Metal, Apple M1, plugin cocoa ; PyAV 18.1.0, OIIO 3.2.2, OCIO 2.6.0, trousseau OK.
- **Corrigé : image noire.** Qt laisse une erreur GL en attente à la création du contexte ; PyOpenGL l'attribuait à
  `glClearColor`, `initializeGL` s'arrêtait. `plateforme.vider_erreurs_gl()` en tête de `initializeGL` (`viewport.py`, `vue3d.py`).
- Lecture du pack de démo : MP4 H.264 24,6 i/s et séquence EXR 24,4 i/s pour 25 voulues, sans saccade.
- `tests/test_plateforme.py` : le test Linux « casse ignorée » échouait sur APFS (insensible à la casse) : comparaison adaptée sous Mac.
- Reste vu à l'écran : le panneau de droite est rogné à gauche (fenêtre 1024 points de large) : cosmétique, à revoir.
- `.mcp.json` lance `python`, absent du Mac : le serveur `re-play-dev` ne démarre pas ici (non modifié : réglage partagé avec Windows).
- `tests/smoke_test.py` (avec écran) : 359 / 364 au premier passage, **363 / 364** après ces corrections :
  - **Raccourcis Ctrl = Cmd** : `raccourcis.depuis_evenement` et `raccourcis_actions.traiter` lisaient la touche Ctrl physique
    du Mac (`MetaModifier`) ; Qt rend Cmd en `ControlModifier`, comme les menus : Cmd+Z (trait), Cmd+↑ / ↓, Cmd+Tab… n'allaient
    pas aux bons gestes. Ctrl = Cmd partout, comme le dit la documentation.
  - **Plein écran** : changer le drapeau « sans bord » recrée la fenêtre, qui n'était plus active (plus aucun raccourci, Échap
    compris, jusqu'au prochain clic) : `toggle_fullscreen` la réactive sous Mac.
  - Menus : la barre du système, Qt la dit toujours cachée ; le test vérifie maintenant qu'elle est bien native.
  - Reste : 1280 × 800, l'heure de la barre de lecture (`time_label`) est rognée.
- Échecs déjà présents avant ces corrections, à voir à part : `test_raccourcis_prefs` (préréglage Resolve : `forme_volet` =
  Ctrl+Maj+Z, pris à « Rétablir » sur Mac) ; `test_raccourcis_version` (Ctrl+↑ : la version v2 n'est pas trouvée, 10 / 11).
  Lancé avec `QT_QPA_PLATFORM=offscreen`, le même test s'arrête net (code 134) après 226 vérifications : sur Mac, le lancer avec écran.
- Pas encore fait : liens `replay://`, Ouvrir avec (déclarés dans l'Info.plist, pas essayés).

## Première version pour testeurs : 0.9.30 (30/10/2026)

Fabriquer (sur ce Mac, environ 15 min la première fois) :
1. ffmpeg LGPL, une seule fois : `bash tools/build_ffmpeg_mac.sh` (code source de ffmpeg.org 8.1.3, sans GPL ni
   bibliothèque externe ; VideoToolbox pour H.264 / H.265 / ProRes) -> `ffmpeg_lgpl/` (non versionné).
2. Numéro et date : `frontend_ui/version.py` (`JOUR`, `DATE`) et `CHANGELOG.md`.
3. `cd frontend_ui && PYTHON=~/re-play-venv/bin/python ./build_mac.sh --testeur`
   -> `~/Builds/re-play/re-play_<version>_Mac_<date>.dmg` (≈ 200 Mo) et `SHA256SUMS.txt`.

Ce que fait `build_mac.sh` : prend le Python de `$PYTHON` (venv uv sans pip accepté) et ne touche pas à la roue PyAV LGPL
(il vérifie qu'elle n'a ni x264 ni x265) ; embarque le module C++ et ffmpeg ; retire `direct_url.json` ; signature « ad hoc »
(`codesign -s -`) ; contrôle « rien de privé » (`tools/verifier_paquet.py`, qui connaît maintenant les dossiers d'un `.app`).
Le numéro et la date sont dans le nom du `.dmg`, dans À propos et dans Finder > Lire les informations.

Essayé par Seb sur ce Mac : image, Cmd+Z, À propos (0.9.30 · 30/10/2026), transcodage H.264 (ffmpeg embarqué) : tout marche.

Corrigé en chemin : `retour.masquer` ne masquait pas un nom de session Windows quand il tournait sur Mac ; le paquet testeur
embarquait `variante_testeur` (outil de construction, Windows compris) ; texte de la LGPL v3 ajouté (`LICENSE-LGPL-3.0.txt`).

**Expiration des versions de test** (Mac, Windows, Linux) : le build testeur pose `expiration.txt` = date du build + 60 jours
(`version.DUREE_TEST_JOURS`). Les 14 derniers jours (`AVERTIR_JOURS`), un message au lancement donne la date ; après,
« Cette version de test de re-play a expiré » et l'app se ferme. Sans internet ; la version du studio et les sources n'expirent
jamais. Date visible dans À propos et `--version`. Essayé à l'écran par Seb (les deux messages). Test : `tests/test_expiration.py`.

**Installeur Windows** : `frontend_ui/installeur.iss` (Inno Setup 6) existait ; il produit maintenant
`re-play_<version>_Windows_<date>.exe`. Jamais compilé : sur le PC Windows, `build_testeur.bat` puis `build_installeur.bat`.

Copie sur le NAS (le Mac n'est pas toujours accessible) : `08_OUTILS/re-play_livraisons/0.9.30_Mac/` (le `.dmg`, son
`SHA256SUMS.txt`, les guides) et `08_OUTILS/re-play_livraisons/ffmpeg_lgpl_mac/` (ffmpeg Mac déjà compilé : le copier dans
`ffmpeg_lgpl/` d'un Mac pour refaire un `.dmg` sans recompiler).

31/10 : nouveau `.dmg` 0.9.30 (`re-play_0.9.30_Mac_2026-10-31.dmg`, expire le 30/12/2026) avec la nouvelle interface et
l'anglais, copié sur le NAS à côté du précédent. Corrigés : menu re-play > Réglages (Qt y rangeait « Préférences (profil
perso…) », qui ouvre le JSON : rôles de menu désignés dans `app_menus.py`) ; fichier ouvert depuis le Finder au lancement
(écouté trop tard, puis envoyé aux Arrivées : `lancement.FiltreOuverture` posé dès le départ, `prendre_lancement()`) ;
`--commande` répond de nouveau dans le terminal sur Mac. Essayé par Seb : vidéo, Préférences, anglais (après relance).

Limites pour les testeurs : Mac à puce Apple (M1 et plus) et macOS 14 ou plus ; pas de version Intel. App non notarisée
(pas de compte Apple Developer) : au premier lancement, Réglages Système > Confidentialité et sécurité > « Ouvrir quand même »
(expliqué dans `GUIDE_TESTEUR.md`).

Reste : `tests/test_ffmpeg_exe.py` échoue déjà avant ces changements (4 vérifications : PATH, et le faux paquet n'a pas les
textes de licence exigés depuis) ; l'heure rognée à 1280 × 800 ; liens `replay://` et Ouvrir avec à essayer.
