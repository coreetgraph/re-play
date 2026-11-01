# Kitsu : « Ouvrir dans re-play »

Depuis Kitsu, sélectionne des tâches, clique l'action : leurs fichiers arrivent dans les **Arrivées** de re-play
(jamais à la place de la session ouverte). Tout se passe sur ton poste : re-play n'écoute que sur `127.0.0.1`.

## Activer (désactivé par défaut)

1. re-play > Préférences > Connecteurs > en bas : cocher **Kitsu : « Ouvrir dans re-play »** (port 8765 par défaut).
2. **Copier l'adresse d'action** : `http://127.0.0.1:8765/kitsu/ouvrir?jeton=<ton jeton local>`.
   Le jeton est au trousseau du système ; **Nouveau jeton** invalide l'ancienne adresse. Ne la partage pas.
3. Il faut une connexion Kitsu configurée (Préférences > Connecteurs) : re-play s'en sert pour lire la tâche, son plan
   et son projet, et ne l'écoute que depuis l'adresse de ce Kitsu.

## Dans Kitsu (compte administrateur)

Paramètres > **Actions personnalisées** > Nouvelle :

| Champ | Valeur |
|---|---|
| Nom | Ouvrir dans re-play |
| URL | l'adresse copiée ci-dessus |
| Type d'entité | Tâche (Task) |
| Ajax | **oui** (la requête part en arrière-plan ; sans, Kitsu ouvre un onglet qui affiche juste « déposé dans les Arrivées ») |

Ensuite, dans une page de tâches : sélectionner > menu d'actions > **Ouvrir dans re-play**.

## Ce que re-play fait

- reçoit les identifiants de tâches, lit chez Kitsu (lecture seule) le plan, le type de tâche et le projet ;
- retrouve le projet sur le poste (`projets_lies` de la connexion, sinon un projet du profil de même nom), puis la **dernière
  version** de la tâche dans l'arborescence du profil de studio ;
- dépose les médias dans les Arrivées (pastille, pop-up discret). Les tâches sans fichier local sont ignorées et signalées.

## Sécurité

127.0.0.1 seulement, en-tête `Host` local obligatoire, jeton obligatoire, navigateur accepté depuis l'origine de Kitsu
seulement, requêtes limitées à 1 Mo / 200 tâches, aucune écriture chez Kitsu, aucun jeton dans les journaux.
Port occupé : change-le (le même dans l'URL de l'action).

Test à la main : `curl -X POST "http://127.0.0.1:8765/kitsu/ouvrir?jeton=…" -H "Content-Type: application/json" -d "{\"selection\": [\"<id de tâche>\"]}"`.
