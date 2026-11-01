# re-play sur Linux (Ubuntu 22.04 / 24.04)

Préparé sous Windows le 19/10/2026 par analyse du code, puis **essayé pour la première fois sous Ubuntu 26.04 le 22/10/2026**
(voir « Vérifié sous Linux »). Ce document suit la mise au point sous Linux (DIF-10). Ce qui a été fait pour le Mac est dans `docs/MAC.md`.

## Prérequis
| Quoi | Détail |
|---|---|
| Système | Ubuntu 22.04 ou 24.04, 64 bits (x86_64). Arm : non essayé. |
| GPU | **OpenGL 3.3 core minimum** (le viewport demande ce profil). NVIDIA : pilote propriétaire (`ubuntu-drivers install`) ; AMD / Intel : Mesa suffit (≥ 22). Pas de rendu logiciel (llvmpipe) pour juger : trop lent. |
| Session | X11 recommandé. Wayland fonctionne via XWayland : `run_linux.sh` force le plugin Qt `xcb` (`REPLAY_WAYLAND=1` pour tenter le Wayland natif). re-play règle tout seul `PYOPENGL_PLATFORM` (`glx` avec xcb, `egl` avec Wayland) : sans cela, l'image restait noire en session Wayland. |
| Python | **3.14** (uv le télécharge ; sinon PPA deadsnakes). |
| Bibliothèques Qt (xcb) | `libxcb-cursor0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-render-util0 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0 libegl1 libgl1 libfontconfig1 libdbus-1-3` |
| Vidéo | `ffmpeg` (apt) : ffmpeg et ffprobe dans le PATH. |
| Trousseau | `libsecret-1-0 gnome-keyring` (ou KWallet) pour les mots de passe Kitsu etc. Sans trousseau : repli en mémoire avec avertissement, rien n'est gardé à la fermeture. |
| Polices | `fonts-inter fonts-ubuntu fonts-noto-core fonts-dejavu-core` (re-play prend la première installée : Inter, Ubuntu, Noto Sans, Cantarell, DejaVu Sans). |
| NAS | `cifs-utils` |
| Ouvrir un dossier, liens `replay://` | `xdg-utils` |

OpenImageIO et OpenColorIO viennent des **roues Python** (`pip`) : pas besoin des paquets apt `libopenimageio-dev` /
`opencolorio`, sauf pour compiler le backend C++ (facultatif).

## Installation pas à pas
```bash
sudo apt update
sudo apt install -y git curl ffmpeg xdg-utils cifs-utils libsecret-1-0 gnome-keyring \
  libxcb-cursor0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-render-util0 libxcb-xinerama0 libxcb-xkb1 \
  libxkbcommon-x11-0 libegl1 libgl1 libfontconfig1 libdbus-1-3 \
  fonts-inter fonts-ubuntu fonts-noto-core fonts-dejavu-core
curl -LsSf https://astral.sh/uv/install.sh | sh        # puis rouvrir le terminal
# les sources : copie du dépôt (NAS monté ou git clone), par exemple
cd ~ && git clone <adresse du dépôt> re-play && cd re-play
./run_linux.sh --diagnostic        # crée le venv (.venv, Python 3.14, paquets pip) puis affiche le diagnostic
./run_linux.sh                     # lance re-play
./run_linux.sh /mnt/ssd1/PROJETS/X/plan.mov    # ouvre un média
```
Sans uv : `sudo add-apt-repository ppa:deadsnakes/ppa && sudo apt install -y python3.14 python3.14-venv`, puis `./run_linux.sh`.
Les préférences sont dans `~/.config/re-play/` (ou l'ancien `~/.re-play/` s'il existe), le cache dans `~/.cache/replay/`,
le journal avec les préférences.

Backend C++ (facultatif, re-play tourne sans) : `./backend_cpp/build_linux.sh --apt` puis copier le `.so` indiqué dans `frontend_ui/`.

## Monter le NAS (SMB)
```bash
sudo mkdir -p /mnt/ssd1 /mnt/nas /mnt/ssd2
sudo mount -t cifs //<adresse-du-serveur>/<partage> /mnt/ssd1 -o username=<utilisateur>,uid=$(id -u),gid=$(id -g),vers=3.0,ro
```
(`ro` pour les premiers essais : lecture seule.) Permanent : `/etc/fstab` avec `credentials=/root/.smbcred` (fichier en `chmod 600`).
Hors réseau local : l'adresse Tailscale du NAS.

Les sessions, revues et profils écrivent des chemins Windows (par exemple `E:\PROJETS\…`). Sous Linux re-play les ramène aux points de montage
d'après la table `correspondances` du profil, **dont les volumes Mac `/Volumes/xxx/` sont lus comme `/mnt/xxx/`** :

| Windows | Linux (par défaut) |
|---|---|
| `E:\` et `\\<serveur>\<partage>\` | `/mnt/<partage>/` |
| autre lecteur | `/mnt/<autre partage>/` (attention à la casse : Linux la respecte) |

Sans rien monter à la main : si le NAS est ouvert dans **Fichiers** (Ubuntu / GNOME, « Autres emplacements » >
`smb://<serveur>/<partage>`…), re-play le trouve tout seul dans `/run/user/<uid>/gvfs/smb-share:server=…,share=<partage>`
(ordre essayé : `/mnt/<partage>`, `/media/<utilisateur>/<partage>`, puis ce montage GNOME ; casse du partage ignorée).
Ces dossiers sont comptés comme réseau (lecture en parallèle réglée en conséquence). Vérifier avec `--diagnostic`,
ligne « NAS » (`E:\=présent`…), une fois le profil du studio choisi (sinon le profil « vierge » n'a pas de lecteurs).

Autre point de montage : Préférences (`"correspondances"` dans `preferences.json`, même forme : `{"E:\\": "/media/moi/partage/"}`).
Pour que les projets apparaissent dans l'accueil : `racines_projets` (champ « Dossiers de projets » de l'accueil), par exemple
`/mnt/ssd1/PROJETS`. Les noms de dossiers et de fichiers sont **sensibles à la casse** sous Linux (`Traces_test` n'est pas `TRACES_TEST`).

## Vérifié sous Linux (22/10/2026)
Poste : Ubuntu 26.04, NVIDIA RTX 2070 (pilote 580, OpenGL 3.3 core), session Wayland, Qt en `xcb`, Python 3.14.8, PySide6 6.12.

| Quoi | Résultat |
|---|---|
| Diagnostic (`--diagnostic`) | Complet : OpenGL, ffmpeg 8, PyAV, OpenImageIO, OpenColorIO, trousseau. |
| Affichage de l'image | **Corrigé** : en session Wayland, PyOpenGL prenait le pilote EGL alors que Qt tournait en xcb ; la texture n'était jamais créée (écran noir). `plateforme.pilote_opengl()` règle `PYOPENGL_PLATFORM`. |
| Plein écran | **Corrigé** : une fenêtre sans bord est replacée par GNOME dans la zone de travail (barre et dock restent visibles). Sous Linux, `showFullScreen()` est utilisé (Windows et Mac inchangés). |
| Tests | `tests/smoke_test.py` : 364 / 364 (dont 4 vérifications « non applicable », voir plus bas) ; `test_theme.py` et `test_sans_ceg.py` verts. |
| Lecture (banc court, médias locaux) | H.264 1080p, ProRes 422 HQ 1080p, H.264 4K : 23,8 i/s à 24, 0 image perdue, fluidité 100 %. EXR half 2K : 23,4 i/s (98 %). PNG 16 bits 2K : 23,5 i/s (99 %). Première image en 0,06 à 0,21 s. |
| Pas encore mesuré | Lecture depuis un partage réseau, banc complet (3 passes), backend C++ (non compilé : cache Python, plus lent). |

## Lancer les tests sous Linux
- **Écran allumé et déverrouillé pendant les tests.** Quand l'écran est verrouillé ou en veille (économiseur actif), GNOME ralentit les fenêtres à
  environ **1 image par seconde** : le fil d'interface semble figé ~1 s à intervalles réguliers, et tous les tests qui comptent des délais
  (« pire pause < 100 ms », « le clip arrive », lecture, focus) échouent à tort. Signe typique : des durées de 1000 ms partout. Avant une longue série :
  désactiver le verrouillage automatique (Réglages > Confidentialité > Verrouillage de l'écran) ou lancer sous `systemd-inhibit --what=idle`.
  Contrôle : `gdbus call --session --dest org.gnome.ScreenSaver --object-path /org/gnome/ScreenSaver --method org.gnome.ScreenSaver.GetActive` doit répondre `(false,)`.
- Les tests d'interface (`smoke_test.py`, `banc_perf.py`) ont besoin d'**OpenGL** : le mode hors écran de Qt (`QT_QPA_PLATFORM=offscreen`)
  n'en offre pas (« QOpenGLWidget is not supported on this platform ») et les tests échouent. Les lancer avec `QT_QPA_PLATFORM=xcb`
  **sur l'écran réel** : des fenêtres s'ouvrent et se ferment, ne pas toucher à la souris ni au clavier (smoke test environ 5 minutes,
  banc court 3 à 5 minutes avec `--ecran`). Un écran virtuel (Xvfb) n'est pas installé et n'a pas été essayé.
- Commande type : `QT_QPA_PLATFORM=xcb REPLAY_SANS_OUVRIR=1 OPENBLAS_NUM_THREADS=1 PYTHONIOENCODING=utf-8 .venv/bin/python tests/smoke_test.py`.
- Les tests sans interface (`test_theme.py`, `test_sans_ceg.py`, etc.) se lancent normalement, un par un.
- Quelques vérifications du smoke test dépendent d'éléments d'un studio (dossiers de projets, scripts de méthode). Sur un poste qui ne les a
  pas, elles s'affichent `-- non applicable : <raison>` et ne sont pas comptées ; sur un poste complet, elles comptent comme avant.

## Ce qui ne marche pas encore / à vérifier
- Installeurs : `frontend_ui/build_appimage.sh` (AppImage) et `build_deb.sh` (.deb) écrits mais **jamais exécutés** : voir « Fabriquer l'AppImage » plus bas.
- Liens `replay://` : écrits dans `~/.local/share/applications/re-play.desktop` (Préférences > Studio > Liens > Réenregistrer) ; non essayé.
- « Montrer dans l'explorateur » : sélectionne le fichier avec Nautilus, Dolphin, Nemo ou Caja (s'ils sont installés), sinon ouvre le dossier (`xdg-open`). Non essayé.
- Détection des logiciels (`logiciels.json`, clés `commandes_linux` = PATH d'abord, puis `chemins_linux`) : Nuke (`/usr/local/Nuke*`, `/opt/Nuke*`, `~/Nuke*`), Blender (PATH, `/usr/bin`, `/opt/blender*`, snap, flatpak), Resolve (`/opt/resolve`), OpenRV, DJV, mrv2, Krita, GIMP, Natron, VLC, Kdenlive, Shotcut, Houdini, Maya, ComfyUI (AppImage / `/opt`). Photoshop, After Effects, Premiere n'existent pas sous Linux. Un logiciel absent s'ajoute à la main (Préférences > Logiciels). Non essayé.
- Envoi vers Resolve : le Resolve ouvert est détecté (`psutil` si présent, sinon `pgrep -x resolve`). Pilotage par l'API de script : chemins Windows seulement (`resolve_env`), donc sous Linux l'envoi se fait par la consigne à copier. Ferme Deadline : Windows seulement.
- ComfyUI Desktop : dossier cherché dans `~/.config/ComfyUI`, `~/Documents/ComfyUI`, `~/ComfyUI`, `~/comfy/ComfyUI`, `/opt/ComfyUI` ; sinon renseigner `comfy_dossier` dans les préférences.
- Polices : l'interface prend la première police installée parmi Inter, Ubuntu, Noto Sans, Cantarell, DejaVu Sans, choisie dès l'import (`theme.py` via `plateforme.polices_ui`) ; des différences de chasse restent possibles. Les incrustations ffmpeg (`drawtext`) utilisent une police trouvée par `fc-match`.
- Préférences écrites sous Windows : les clés de chemins (`recents_dates`, `projets_lies`…) sont en minuscules avec `\` ; `os.path.normcase` ne fait rien sous Linux, donc ces anciennes entrées peuvent ne pas être retrouvées (`plateforme.cle_chemin` / `chercher_cle` existent pour les lire, branchés nulle part pour l'instant).
- Raccourcis : mêmes touches que Windows (Ctrl).
- Aucun essai de lecture HDR, ni de plusieurs écrans à échelle fractionnaire (Wayland).

## Fabriquer l'AppImage (testeurs) : `build_appimage.sh`, `build_deb.sh`
Écrits sous Windows, **jamais exécutés**. Ils fabriquent la **variante testeur** (rien de privé : `REPLAY_VARIANTE=testeur`, `variante_testeur.py`).
À faire sur l'Ubuntu qui construit (22.04 conseillé : l'AppImage tourne sur les versions aussi récentes ou plus récentes) :
1. Paquets : ceux de l'installation ci-dessus + `libfuse2` (ou lancer appimagetool avec `--appimage-extract-and-run`, ce que fait le script).
2. Python 3.14 (uv ou deadsnakes), ou `PYTHON=/chemin/python3.14`. Le script crée son venv dans `~/Builds/re-play-linux/venv`.
3. `appimagetool` : télécharger le fichier `appimagetool-x86_64.AppImage` depuis les « releases » du projet AppImage, `chmod +x`, puis `APPIMAGETOOL=/chemin/appimagetool-x86_64.AppImage` (le script ne télécharge rien).
4. ffmpeg embarqué (conseillé) : `FFMPEG_DIR=<dossier contenant ffmpeg et ffprobe statiques>` ; sinon le script copie ceux du système (ils dépendent des bibliothèques de la machine).
5. `cd frontend_ui && ./build_appimage.sh` : contrôle « rien de privé », PyInstaller (dossier), vérification du paquet, `AppDir` (`re-play.desktop`, icône `replay_icon.ico` convertie en PNG par `ico_vers_png.py`, `AppRun`), appimagetool, SHA-256.
   Sortie : `~/Builds/re-play-linux/livraison/re-play-<version>-x86_64.AppImage` et `SHA256SUMS.txt`.
6. `.deb` (facultatif, après l'étape 5) : `./build_deb.sh` emballe le même dossier dans `/opt/re-play` (`dpkg-deb`), dépendances déclarées (ffmpeg, bibliothèques Qt).
7. Essai : `chmod +x re-play-*.AppImage && ./re-play-*.AppImage --version`, puis le diagnostic ; le lien `replay://` pointe vers l'AppImage elle-même (variable `APPIMAGE`).
Limites connues : arm64 non essayé ; l'AppImage ne contient pas les pilotes OpenGL (ceux du système sont utilisés) ; pas de mise à jour automatique.

## Rapporter un problème
1. `./run_linux.sh --diagnostic` : coller **toute** la sortie dans le message.
2. Le journal : `~/.config/re-play/` (fichiers `.log`), ou Aide > À propos de re-play.
3. Dire : ce qu'on a fait, ce qu'on attendait, ce qui est arrivé, avec une capture. Si re-play ne démarre pas, relancer dans le terminal et coller les dernières lignes.

## Checklist pour la machine Ubuntu (à remplir avant la session)
- [ ] 1. Version : `lsb_release -d` (22.04 ou 24.04), architecture x86_64.
- [ ] 2. GPU et pilote : `lspci | grep -i vga`, `nvidia-smi` ou `glxinfo -B | grep "OpenGL version"` (≥ 3.3 core).
- [ ] 3. Session : `echo $XDG_SESSION_TYPE` (x11 ou wayland), plusieurs écrans ?
- [ ] 4. Accès au NAS : `ping <adresse-du-serveur>`, identifiants SMB, dossier `/mnt/ssd1/PROJETS` visible.
- [ ] 5. Python 3.14 : `uv python list | grep 3.14` ou `python3.14 --version`.
- [ ] 6. `ffmpeg -version` et `ffprobe -version` (8.x conseillé).
- [ ] 7. Les paquets apt ci-dessus installés sans erreur (surtout `libxcb-cursor0`).
- [ ] 8. `./run_linux.sh --diagnostic` : aucune ligne « ERREUR » ni « INTROUVABLE » ; OpenGL 3.3+.
- [ ] 9. Trousseau : une ligne « Trousseau » sans « REPLI MÉMOIRE » (sinon `gnome-keyring` ou session sans trousseau).
- [ ] 10. Un média test (mov, exr, séquence) sur le NAS et un `.repl` pour tester l'ouverture, la lecture et la correspondance des chemins.
