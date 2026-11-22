# Démonstration e-Suivi SD-RFP — solution Grist Core

Data Processing Services — maquette fonctionnelle de l'offre (option 4 : Grist Core seul, hébergé sur l'infrastructure du Ministère).
Toutes les données sont **fictives**.

## Contenu du dossier

| Élément | Rôle |
|---|---|
| `docker-compose.yml` | Grist Core 1.7.22 (image `gristlabs/grist-oss`, licence Apache 2.0), Keycloak 26.0 (Apache 2.0) et copie des données au premier lancement |
| `.env` | Secrets de démonstration (session Grist, client OIDC, administrateur Keycloak) |
| `keycloak/sdrfp-realm.json` | Domaine Keycloak « sdrfp » : client Grist et six comptes de démonstration |
| `donnees-initiales/` | Document Grist prêt à l'emploi et base des comptes (copiés au premier lancement) |
| `demarrer.ps1`, `arreter.ps1` | Lancement et arrêt sous Windows (Docker Desktop ou Docker dans WSL) |
| `demarrer.sh` | Lancement sous Linux ou dans WSL |
| `scripts/construire_document.py` | Reconstruit le document (tables, formules, données, règles d'accès, pages) par l'API Grist |
| `scripts/tester_droits.py` | Vérifie les règles d'accès profil par profil (23 tests) |
| `scripts/generer_realm.py` | Régénère `keycloak/sdrfp-realm.json` |
| `scripts/exporter_donnees.sh` | Recopie l'état courant de Grist dans `donnees-initiales/` |
| `scripts/captures.mjs` | Refait les captures d'écran par profil avec Chrome sans interface (`node scripts/captures.mjs captures`) |
| `captures/` | Captures d'écran utilisées dans l'offre technique (section 9) |

## Prérequis

- Docker avec Docker Compose v2 : Docker Desktop, ou Docker installé dans WSL (Ubuntu).
- Ports libres : **8484** (Grist) et **8080** (Keycloak).
- Un navigateur Chrome, Edge ou Firefox : l'adresse `keycloak.localhost` est résolue automatiquement vers la machine locale.

## Lancer la démonstration

Windows, dans PowerShell, depuis ce dossier :

```powershell
.\demarrer.ps1
```

Linux ou WSL :

```bash
bash demarrer.sh
```

Puis ouvrir **http://localhost:8484**. Le premier lancement prend environ une minute (démarrage de Keycloak).

> Avec Docker dans WSL (sans Docker Desktop), la fenêtre WSL ouverte par `demarrer.ps1` doit rester ouverte : Windows arrête WSL, et donc la démonstration, quand plus aucune fenêtre WSL n'est active.

Arrêter : `.\arreter.ps1` (les modifications sont conservées). Revenir aux données initiales : `.\arreter.ps1 -Reinitialiser`, puis `.\demarrer.ps1`.

## Comptes de démonstration

Mot de passe commun : **Demo-2026** (identifiant ou courriel au choix).

| Identifiant | Profil | Ce qu'il voit et fait |
|---|---|---|
| `admin.dpre` | Administrateur DPRE (propriétaire) | Tout ; paramétrage, référentiel, campagnes, consolidation, réouverture, règles d'accès |
| `pf.dgb` | Point focal DGB | Saisies de la DGB ; modifie ses lignes en brouillon ou rejetées, les soumet, joint des preuves |
| `pf.dgi` | Point focal DGI | Idem pour la DGI |
| `val.dgb` | Validateur DGB | Saisies de la DGB ; valide ou rejette avec motif les lignes soumises, sans modifier la déclaration |
| `resp.axe1` | Responsable de l'axe 1 | Lecture du suivi de l'axe 1 |
| `comite` | Comité technique | Lecture de tout le suivi et des tableaux de bord |

Console d'administration Keycloak : http://keycloak.localhost:8080/admin (compte `admin`, mot de passe dans `.env`).

Pour changer d'utilisateur, utiliser le menu du compte (en haut à droite) puis « Se déconnecter », ou une fenêtre de navigation privée par profil. Si Keycloak affiche « Paramètre invalide : id_token_hint » après un redémarrage, ouvrir à nouveau http://localhost:8484.

## Pages de l'application

| Page | Contenu |
|---|---|
| Tableau de bord | Avancement pondéré par axe (tableau et graphique), campagnes et taux de réponse / complétude / validation, indicateurs de l'année avec feux, actions de l'axe sélectionné et situation calendaire |
| Campagnes et validation | Campagnes → saisies de la campagne → fiche de saisie (contrôles automatiques, traçabilité « modifié par », « validé par ») → preuves jointes |
| Référentiel et planification | Axes → sous-axes → actions (dates baseline et révisées, état validé, situation calendaire, date de fraîcheur) → jalons et reprogrammations |
| Indicateurs | Cadre de résultats (baseline, cibles 2025-2030, réalisé validé, taux d'atteinte, feu, cadres PEFA / PIMA / TADAT / DeMPA) → valeurs par période et courbe |
| Risques et décisions | Registres des risques (criticité calculée) et des décisions des comités (retards signalés) |
| Partenaires et financements | Partenaires → financements par action (prévu, acquis, engagé, dépensé, besoin non financé) |
| Administration | Utilisateurs et habilitations, structures, paramètres (seuil « à risque », année de suivi) |

## Scénario de démonstration (environ 15 minutes)

1. **`admin.dpre` — vue d'ensemble.** Tableau de bord : avancement par axe calculé à partir des seules saisies validées ; cliquer sur un axe pour voir ses actions et leur situation calendaire ; feux des indicateurs.
2. **`admin.dpre` — référentiel.** Page Référentiel : hiérarchie axe → sous-axe → action, code calculé (A.1.1.1), date révisée et reprogrammation tracée.
3. **`pf.dgb` — saisie.** Campagnes et validation → campagne « T4 2026 » (ouverte) : seules les lignes de la DGB apparaissent, pré-remplies avec la dernière valeur validée. Modifier un taux, observer la colonne « Contrôles automatiques », puis passer le statut à « Soumis ». Montrer qu'une ligne d'une campagne clôturée (T3 2026) ne peut pas être modifiée, et qu'un taux de 150 % empêche la soumission.
4. **`val.dgb` — validation.** Même page : tenter un rejet sans motif (refusé), saisir un motif et rejeter, ou valider. La colonne « Validé / rejeté par » enregistre le validateur ; il ne peut pas modifier le taux déclaré.
5. **`pf.dgb` — correction.** La ligne rejetée redevient modifiable ; corriger et soumettre à nouveau.
6. **`comite` — consultation.** Tout le suivi en lecture seule ; le tableau de bord reflète la validation faite à l'étape 4.
7. **`admin.dpre` — administration.** Page Administration : ajouter un utilisateur ou changer un profil ; Paramètres : modifier le seuil « à risque » et voir les situations calendaires se recalculer. Menu « Permissions avancées » : règles d'accès lisibles, et fonction « Voir en tant que » pour vérifier ce que voit chaque profil.

## Reconstruire le document

1. Se connecter en `admin.dpre`, menu du compte → « Paramètres du profil » → « Clé API » → créer une clé.
2. Lancer :

```powershell
python scripts\construire_document.py --url http://127.0.0.1:8484 --cle <clé API>
```

Le script supprime le document du même nom puis le recrée (environ 1 minute) et affiche son adresse.

3. Vérifier les droits : créer une clé API pour chaque compte de démonstration, les reporter dans un fichier `cles.json` (`{"admin": "...", "pf.dgb": "...", "pf.dgi": "...", "val.dgb": "...", "resp.axe1": "...", "comite": "..."}`), puis :

```powershell
python scripts\tester_droits.py --url http://127.0.0.1:8484 --doc <id du document> --cles cles.json
```

4. Pour en faire les données initiales : `bash scripts/exporter_donnees.sh` (dans WSL ou sous Linux, depuis ce dossier).

## Différences avec la cible de production

La démonstration tourne en local, en HTTP, avec Keycloak en mode développement. En production, l'offre prévoit en plus : Nginx avec le certificat TLS du Ministère, Keycloak avec base de données et double authentification pour les profils sensibles, stockage compatible S3 pour les pièces jointes et les snapshots, sauvegarde restic et supervision.
