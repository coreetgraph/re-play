# re-play : guide du testeur

Merci d'essayer re-play, un lecteur et outil de revue pour les plans VFX et d'animation. Il s'agit d'une version de test, qui peut
contenir des erreurs. Les médias ouverts ne sont jamais modifiés et rien n'est envoyé automatiquement. Les conditions du test sont
dans `ACCORD_DE_TEST.md`.

## 1. Installation

- Windows (10 / 11, 64 bits) : lancer `re-play_<version>_Windows_<date>.exe`. Le premier écran demande la langue (français ou
  anglais) ; l'installation se fait pour l'utilisateur courant, sans droits administrateur. Comme le programme n'est pas signé,
  Windows peut afficher « Windows a protégé votre ordinateur » : cliquer *Informations complémentaires*, puis *Exécuter quand
  même*. Sur certains PC (Smart App Control, postes d'entreprise), un programme non signé est bloqué sans possibilité de passer
  outre ; merci de nous le signaler.
- Mac (puce Apple M1 ou plus récente, macOS 14 Sonoma ou plus récent) : ouvrir `re-play_<version>_Mac_<date>.dmg` et glisser
  `re-play` dans `Applications`. Au premier lancement, macOS indique qu'il ne peut pas vérifier l'app (elle n'est pas signée par
  Apple) : cliquer *Terminé*, puis ouvrir *Réglages Système > Confidentialité et sécurité* et cliquer *Ouvrir quand même* en face
  de « re-play a été bloqué ». Le mot de passe de la session est demandé une seule fois. Sur Mac, la touche Cmd remplace Ctrl
  partout dans ce guide.
- Linux (64 bits, Ubuntu 26.04 ou plus récent ; export H.264 avec une carte NVIDIA) : rendre
  `re-play-<version>-x86_64.AppImage` exécutable (clic droit > Propriétés > « Autoriser l'exécution », ou
  `chmod +x re-play-*.AppImage`), puis l'ouvrir.
- Langue : re-play prend celle de l'ordinateur (sous Windows, celle choisie dans l'installeur). Elle se change dans
  *Préférences > Apparence > Langue*, au lancement suivant.
- Le numéro et la date de la version sont dans *Aide > À propos de re-play* (sur Mac : menu *re-play > À propos de re-play*).
  Merci de les indiquer dans chaque retour.
- Une version de test expire au bout de 60 jours. La date figure dans À propos et un message prévient deux semaines avant ; il
  suffit alors de demander la nouvelle.
- Pour vérifier un téléchargement : `Get-FileHash -Algorithm SHA256 <fichier>` (PowerShell) doit donner la même valeur que la
  ligne correspondante de `SHA256SUMS.txt`.

## 2. Le pack de démonstration

Décompresser `re-play_pack_demo.zip`. Au premier lancement, l'assistant s'ouvre ; sa dernière page propose *Ouvrir le projet
Demo…*, avec le dossier `Demo`. Par la suite : *Fichier > Ouvrir un projet…*. Le pack contient 3 plans
(deux versions vidéo, une séquence d'images, une image et sa référence), quelques images et un montage `Demo.repl`. Les images
viennent du film libre *Cosmos Laundromat* (© Blender Foundation, cosmoslaundromat.org, licence CC-BY 4.0).

## 3. Cinq choses à essayer

1. Lire : ouvrir `PLAN_010`, lancer la lecture avec la barre d'espace, puis avancer image par image avec les flèches.
2. Comparer deux versions : sur `PLAN_010`, passer de `v001` à `v002` (touche V pour le panneau des versions, ou Ctrl+↑ /
   Ctrl+↓), puis essayer *Différence* ou *Volet*. Les flèches ↑ / ↓ seules passent au clip précédent ou suivant.
3. Régler l'image : exposition, gamma et saturation (touches E, G, S, puis glisser), par exemple sur la séquence de `PLAN_020`.
4. Annoter : dessiner sur une image (touche D), ajouter une note, puis exporter la revue en PDF (menu *Pipeline > Revue*).
5. Ouvrir une timeline : *Fichier > Ouvrir une timeline (.repl)* avec `Demo.repl`. Ensuite, glisser ses propres médias sur la
   fenêtre.

## 4. Envoyer un retour

Le menu *Aide > Envoyer un retour…* ouvre une fenêtre où écrire le message : ce qui était en cours, ce qui s'est passé, ce qui était
attendu. Juste en dessous apparaît tout ce qui l'accompagne (version, système, fin du journal). Les chemins, le nom de la session
et du poste, les adresses, les e-mails et les jetons y sont masqués. Deux possibilités ensuite :

- *Copier le message* : un texte simple, à coller dans un e-mail ou un message ;
- *Enregistrer un .zip…* : un fichier contenant le message, le diagnostic et, si la case est cochée, une capture de la fenêtre.
  La capture montre ce qui est à l'écran, plans compris : à ne joindre que si l'image n'est pas confidentielle.

re-play n'envoie rien lui-même : il copie un texte ou écrit un fichier sur le disque. Pour le diagnostic seul, sans message :
*Aide > Copier le diagnostic*.

Le texte ou le `.zip` s'envoie par e-mail ou par message à la personne qui a fourni le paquet ; son contact figure dans l'envoi, et
la fenêtre « Envoyer un retour » le rappelle. Le dépôt GitHub n'étant pas encore public, le bouton « Copier pour une issue GitHub »
n'apparaît pas dans cette version. Merci de relire avant d'envoyer et de ne jamais joindre un vrai plan confidentiel. Le modèle
`RETOUR_TESTEUR.md` peut aussi servir à la main.

## 5. Ce qui n'est pas encore fait

- Pas de mise à jour automatique : une nouvelle version s'installe par-dessus l'ancienne et garde les réglages.
- Les liens avec Kitsu, ShotGrid, ftrack et AYON n'ont pas été essayés avec de vrais serveurs dans ce paquet.
- Le programme n'est pas signé (Windows, Mac), d'où les avertissements au premier lancement décrits plus haut.
- Linux et Mac ont été moins essayés que Windows : tout problème rencontré nous intéresse.
