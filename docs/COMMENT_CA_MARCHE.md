# Comment marche re-play

re-play est le poste où l'on **regarde, vérifie, décide et fait circuler** les plans. Il ne fabrique pas d'images (Nuke,
Blender, ComfyUI le font) et ne remplace pas un outil de suivi de production (Kitsu, ShotGrid). Il tourne sur l'ordinateur,
sans compte, sans serveur, sans IA intégrée. Deux règles valent partout : **les médias d'origine ne sont jamais modifiés**, et
**rien ne part à l'extérieur sans un clic** dans une fenêtre de confirmation.

Le parcours d'un plan dans re-play :

```
 fichiers reçus ──► INGEST ──► VÉRIFIER ──► REVUE ──► PUBLIER / ENVOYER
  (Nuke, IA,        ranger      regarder     notes,      Kitsu / ShotGrid,
   collègue)        et contrôler l'image     verdict     dossier d'envoi daté
                         ▲                                     │
                         └──── BOÎTE DE RÉCEPTION ◄────────────┘
                               (retours, reprises, propositions de l'IA)
```

Version décrite : 0.9.31 (novembre 2026).

---

## 1. Avant tout : comment re-play comprend les dossiers (le profil)

re-play doit savoir **comment les projets sont rangés sur le disque** : où est un projet, où sont les plans, les versions,
les étapes (Layout, 3D, Comp…). C'est le rôle du **profil de studio**, un petit fichier de réglages.

### Trois façons de travailler

| Profil | Pour qui | Ce que re-play comprend |
|---|---|---|
| **Vierge (simple lecteur)** | une personne seule, des dossiers en vrac | rien d'imposé : on ouvre des dossiers et on lit des médias. Pas de plans, pas de versions automatiques. |
| **Prism** | un studio qui range ses projets avec Prism | toute l'arborescence Prism : séquences, plans, étapes, versions, statuts. |
| **Profil perso** (« Deviné — … ») | un studio avec sa propre organisation | re-play regarde un projet existant, devine sa structure, la montre, et l'enregistre comme profil. |

### Montrer un projet à re-play (« Comprendre un projet »)

*Préférences > Projets*, bloc « Le dossier à comprendre » (ou la 2e page de l'assistant du premier lancement,
« Montrer un de mes projets… ») :

1. Choisir le dossier d'un projet existant, puis **Comprendre**.
2. re-play affiche **ce qu'il a compris** : une ligne par niveau de dossier (séquence, plan, étape, version…), avec des
   exemples. Un rôle mal deviné se corrige à la main.
3. Il liste les **médias trouvés** et ce qu'il a **remarqué** (plans sans V000, versions manquantes ou en double,
   « v » et « V » mélangés, une même étape écrite de plusieurs façons : comp / compo / compositing…).
   Ce sont des suggestions : rien n'est modifié.
4. **Utiliser cette structure** : la structure devient un profil dans les préférences. **Rien n'est écrit dans le projet.**

Les dossiers où chercher les projets s'ajoutent dans *Préférences > Projets > Dossiers de projets*.

### Ce qu'il y a dans un profil

Un profil est un fichier JSON. Pour Prism, les lignes importantes disent :

- **ce qui fait un projet** : un dossier qui contient `03_Production`, `PROJET.md` ou `00_Pipeline/pipeline.json` ;
- **l'arbre** : `03_Production/Shots/{sequence}` → `{plan}` → `Renders/{famille}/{identifiant}` → `{version}` → `[rgb]` ;
- **le format des versions** : `v###` (v001, v002…) ;
- **la règle de l'étape** : l'étape est le début de l'identifiant, jusqu'au premier tiret.

Exemple, ce chemin :

```
D:/PROJETS/DEMO/03_Production/Shots/SEQ/SEQ_0010/Renders/external/Comp-alpha/v003/rgb/SEQ_0010_comp_v003.0001.exr
```

est lu ainsi :

| Morceau | Sens |
|---|---|
| `D:/PROJETS/DEMO` | le projet (il contient `03_Production`) |
| `SEQ` | la séquence |
| `SEQ_0010` | le plan |
| `external` | la famille de rendu |
| `Comp-alpha` | l'identifiant… |
| `Comp` | …dont l'**étape** est Comp (avant le tiret) |
| `v003` | la version |

**La règle du tiret** : `Comp-alpha`, `Comp-sansgrain` et `Comp` sont la même étape **Comp** ; `3D-beauty` est l'étape **3D**.
Seul le séparateur du profil compte (le tiret par défaut) : `Comp_alpha` ou `Compo-alpha` font une étape à part. C'est cette
règle qui remplit les layers de la timeline POST-PROD (V000, Layout, 3D, Comp…).

### Plusieurs ordinateurs, plusieurs systèmes

Un même partage n'a pas le même nom partout (`Z:\` sous Windows, `/Volumes/projets/` sur Mac, `/mnt/projets/` sous Linux).
*Préférences > Stockage > Correspondances entre systèmes* dit à re-play que ces chemins sont les mêmes : une revue ou une
timeline faite sur un poste s'ouvre sur l'autre. On y règle aussi les copies locales (travailler hors du studio sur une
copie d'un projet du NAS).

### Où re-play range ses propres réglages

| | Windows | Mac | Linux |
|---|---|---|---|
| Préférences, profils perso, journal de l'IA | `%APPDATA%\re-play\` | `~/Library/Application Support/re-play/` | `~/.config/re-play/` |
| Cache (modifiable : *Préférences > Stockage*) | `%LOCALAPPDATA%\re-play\cache` | `~/Library/Caches/re-play` | `~/.cache/replay` |

Les profils perso sont dans le sous-dossier `profils/` des préférences (*Préférences > Projets > Ouvrir le dossier des profils*).

---

## 2. Ingest : ranger les médias reçus

*Outils > Ingest : ranger les plans…* (Ctrl+Maj+I), ou clic droit sur un plan dans la bibliothèque > *Ingest dans ce plan…*.

L'ingest prend des fichiers venus de dehors (un rendu, une sortie d'IA, un envoi d'un collègue) et les **copie** à la bonne
place du projet, avec le bon nom et un nouveau numéro de version.

1. Choisir le projet, puis ajouter les fichiers (bouton ou glisser-déposer).
2. Pour chaque fichier, re-play devine le **plan** d'après le nom ou le chemin (`TRA0020` → `TRA_0020`), et propose l'étape,
   la version suivante et le **nom final**. Tout se corrige dans le tableau ; un plan manquant se crée sur place.
3. Une **note commune** garde le contexte (prompt, réglage, qui l'a envoyé).
4. **Déclarer** ouvre un récapitulatif « source → destination ». Rien n'est encore écrit. **Retour** pour corriger,
   **Déclarer** pour copier.

Ce qui se passe ensuite :

- Les fichiers sont **copiés, jamais déplacés**, et **rien n'est écrit dans le dossier d'origine**.
- La copie va dans une nouvelle version (`v003` si `v002` existe) ; un nom déjà pris reçoit `_2`, jamais d'écrasement.
- Une fiche `versioninfo.json` garde la provenance et la note ; le projet tient la liste des déclarations
  (`.re-play/declarations.json`).
- *Déclarations…* permet de corriger ou d'annuler. Annuler ne supprime rien : la copie part dans `_corbeille_replay/` du
  projet, d'où elle se restaure.

### Le contrôle qualité (QC) de l'ingest

Pendant qu'on remplit le tableau, re-play contrôle chaque fichier en tâche de fond (pastille dans la colonne QC :
**ok**, **à voir**, **bloquant**) : images manquantes ou mal numérotées, images noires ou en double, pixels invalides dans les
EXR, cadence ou résolution différentes du projet, durée inattendue, son absent, nom hors nomenclature. Rien n'est refusé
d'office : s'il y a un point bloquant, re-play demande confirmation. Le rapport (`QC_<nom>.md`) est rangé avec la version.

### Le V000

Le **V000** est le plan **sans effet**, découpé dans le montage d'origine avec ses poignées. C'est la référence : on compare
chaque version à lui, et il sert de source tant qu'aucune version de travail n'existe. Il est **en lecture seule** ; le travail
commence en v001. Il se fabrique en découpant un montage (conformation, fenêtre *Dispatcher*), pas par l'ingest.

---

## 3. Vérifier : regarder l'image de près

Onglet **Vérifier** du panneau de revue (Maj+R ouvre le panneau). C'est un contrôle **à l'œil**, qui ne modifie aucun fichier :

- **Régler l'image** : maintenir E (exposition), G (gamma), S (saturation) ou N (niveaux) et glisser ; *Tout remettre à zéro*.
- **Grain et écrêtage** : *Vue grain* ; *Écrêtage* (rayures rouges = blancs brûlés, bleues = noirs bouchés) ;
  *Fausses couleurs* (T) ; *Fenêtres EXR* (Maj+T).
- **Contre la version précédente** : *Différence* |A − B| (ce qui a changé ressort), *Volet*, retour à l'image normale (W).
- **Contre le V000** : la même comparaison avec le plan sans effet, pour voir exactement ce que l'effet a fait.

---

## 4. La revue

Une revue rassemble des versions à juger, les notes, les dessins et les verdicts. Pour en commencer une : la carte
**Faire une revue** de la page de démarrage, le bouton **Créer la revue** du panneau (avec les médias chargés), ou
*Pipeline > Revue > Ouvrir une revue…*.

Le panneau suit **4 étapes**, affichées en haut (vert = fait, bleu = en cours, un clic y mène) :

1. **Versions** : quelle version regarder ? Les versions du plan sont en haut, avec la comparaison A / B (volet, côte à côte,
   différence). Une version regardée plus d'une seconde est marquée « vu ».
2. **Notes** : une note par version, avec l'image dessinée si besoin (touche D : main levée, flèche, cadre, texte). Le fil
   garde tout l'historique.
3. **Verdict** : *Valider*, *À reprendre*, *Retenir* ou *Note seule* ; les autres statuts du projet en petites pastilles.
4. **Envoi** : le bilan (« 2 validées, 1 à reprendre »), puis :
   - **Publier…** : envoie verdicts et notes vers Kitsu ou ShotGrid. Une fenêtre montre tout avant (plan, version, statut
     posé, note, image) ; rien ne part avant le bouton *Publier*. Pour un plan à reprendre, on choisit pour qui : une
     personne de l'équipe, l'assistant IA, ou plus tard ;
   - **Préparer l'envoi…** : fabrique un dossier d'envoi daté pour le client ou le monteur (`05_Echanges/OUT/<date>_<destinataire>`),
     tel quel ou transcodé, avec infos dans l'image, carton ou filigrane si on veut. re-play prépare le dossier ; on
     l'envoie soi-même.

Autres onglets du panneau : *Vérifier* (ci-dessus), *Historique*, *Revues* (les revues passées du plan, double-clic pour
rouvrir), *Infos*, *IA*.

**Où c'est enregistré** : dans le dossier du plan, `_revue/<date>.json` pour la revue, et `_revue/retours.jsonl`, un
journal où chaque geste s'ajoute sans jamais réécrire le précédent. *Pipeline > Revue* exporte aussi la revue en **PDF**
(une page par note, avec l'image dessinée) ou prépare un brouillon de mail.

### La boîte de réception

Tout ce qui arrive de dehors pendant qu'on travaille passe par la **Boîte de réception** (pastille *Réception* en haut,
*Fichier > Boîte de réception…*) : un lien `replay://`, un fichier envoyé par un script, une reprise publiée par un
superviseur, une proposition de l'IA. **Rien ne remplace la session ouverte** : on choisit d'ouvrir dans un nouvel onglet,
d'ajouter à la suite, ou de retirer.

---

## 5. L'IA dans re-play (serveur MCP)

re-play **ne contient aucune IA**. Il peut en revanche **laisser une IA travailler avec lui** (Claude, Codex, Gemini, une IA
locale…), par un petit serveur au format **MCP**, un standard que ces assistants savent utiliser.

### Ce que l'IA peut faire

- **Lire** : les projets, plans, versions, notes, retours du client, envois, le profil, et une fiche d'un média.
- **Proposer** : déposer une nouvelle version (rangée selon le profil, jamais en écrasant), envoyer des fichiers à regarder,
  dire qu'elle prend en charge une reprise et où elle en est, demander un accord.

### Ce qu'elle ne peut pas faire

Donner un verdict, publier vers Kitsu ou ShotGrid, envoyer quoi que ce soit au client, supprimer, changer les réglages.
Ces outils n'existent pas pour elle. **Tout ce qu'elle dépose arrive dans la Boîte de réception**, et c'est l'humain qui
regarde et décide.

Autres garde-fous :

- Elle ne voit que les dossiers de projets connus de re-play, pas le reste de l'ordinateur.
- Le serveur n'écoute pas le réseau : c'est l'assistant qui le lance sur la même machine.
- Chaque appel est noté dans un journal (`mcp/journal.jsonl` dans le dossier des préférences), mots de passe masqués.
- Un service en ligne payant (par exemple Comfy Cloud) ou un dépassement de crédits demande un accord explicite.
- Le **voyant IA**, en haut de la fenêtre : **gris** = l'IA lit seulement, **vert** = autorisée à déposer, **orange** = elle
  vient de travailler (dans les 2 dernières minutes). Un clic montre le dernier appel et ouvre le journal.

### Configurer chez soi

1. **Ouvrir re-play** > *Préférences > IA · MCP*.
2. **Décider ce qu'elle a le droit de faire** :
   - rien coché : l'IA peut seulement lire ;
   - **MCP activé** : elle peut déposer des propositions dans la Boîte de réception ;
   - **Lecture seule** : même activé, elle ne peut rien écrire ;
   - **L'IA voit les images** (décoché par défaut) et les plafonds de crédits.
3. **(Conseillé) Préparer le contexte** : un petit texte qui explique à l'IA le studio, le projet, les habitudes
   (`CONTEXTE_IA.md`). L'assistant le pré-remplit ; montants, identifiants et jetons en sont retirés automatiquement.
   *Que sait l'IA de moi ?* montre ce qu'elle lira.
4. **Brancher l'assistant** : la page donne la commande toute prête, à copier.
   - **Claude Code** : coller la ligne dans un terminal. Elle ressemble à :
     ```
     claude mcp add --scope user re-play -- "<dossier de re-play>\re-play.exe" --mcp
     ```
     (la page écrit le vrai chemin de re-play sur cet ordinateur).
   - **Autre client** (Claude Desktop, Codex, Gemini CLI, Cline, LM Studio, Open WebUI…) : copier le bloc
     « Autre client (JSON) » dans les réglages MCP de ce logiciel :
     ```json
     { "mcpServers": { "re-play": { "command": "<chemin de re-play>", "args": ["--mcp"] } } }
     ```
5. **Vérifier** : ouvrir une **nouvelle** session de l'assistant et demander « Liste mes projets re-play ». Puis « Valide le
   plan SEQ_0010 » : il doit répondre qu'il n'en a pas le droit. Le voyant passe à l'orange pendant qu'il travaille.

Les projets que l'IA peut voir sont ceux de *Préférences > Projets* : si elle ne trouve pas un projet, c'est qu'il n'est pas
dans un dossier de projets connu.

**IA locale** (rien ne sort de l'ordinateur) : un modèle de 7 à 8 milliards de paramètres compatible « tools » dans
Ollama, branché par un client MCP (Open WebUI, Goose, Cline, LM Studio), avec le même bloc JSON.

Sous Windows, re-play peut aussi **réveiller l'assistant tout seul** quand une reprise lui est confiée (*Démarrage
automatique* : Claude Code, Codex, Gemini CLI ou une autre commande, puis *Installer le veilleur*).

---

## En résumé

| Étape | Où | Ce qui est écrit |
|---|---|---|
| Comprendre les projets | Préférences > Projets | un profil, dans les préférences (rien dans le projet) |
| Ingest | Outils > Ingest : ranger les plans… | une copie dans une nouvelle version, sa fiche, le rapport QC |
| Vérifier | panneau de revue, onglet Vérifier | rien |
| Revue | panneau de revue, 4 étapes | `_revue/` dans le dossier du plan |
| Publier / Envoyer | étape 4 de la revue | chez Kitsu / ShotGrid ou dans un dossier d'envoi, **après confirmation** |
| IA | Préférences > IA · MCP | ses propositions, dans la Boîte de réception |

Questions, bugs, idées : le [Discord Core et Graph](https://discord.gg/BDAKUGz5jp) (salons `re-play-help`, `re-play-bugs`,
`re-play-ideas`), le [formulaire de retour](https://tally.so/r/WJ25qL), ou re-play@coreetgraph.com.
