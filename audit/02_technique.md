# Audit — technique
> Cf. procedure_audit.md (section 3). Ce volet alimente rapport_audit.md.

## 1. Architecture

- Le système tient en deux scripts : `legacy/train.py` (entraînement) et `legacy/predict.py` (prédiction en ligne de commande), plus un fichier modèle `dms_predictor_v1.joblib`.
- Aucun module, aucune fonction, aucun fichier de configuration, aucun fichier de dépendances côté legacy. Le modèle a été sérialisé avec scikit-learn 1.5.1 ; cette version n'est fixée nulle part dans le legacy.
- Couplage fort et duplications :
  - `predict.py` charge le modèle par un chemin relatif : il ne fonctionne que lancé depuis la racine du dépôt (`FileNotFoundError` vérifié depuis un autre dossier).
  - Le codage du sexe (`sexe_bin`, 1 = homme) et l'ordre des colonnes sont écrits séparément dans les deux scripts et ne sont documentés nulle part.
- Aucun pipeline de prétraitement : la sélection des colonnes et l'encodage du sexe sont faits à la main, séparément dans chaque script. Le fichier modèle est un `RandomForestClassifier` seul, qui ne contient aucune étape de préparation des données.
- Aucun test dans le legacy.

## 2. Sécurité

| Constat | Preuve | Portée |
|---|---|---|
| Mot de passe en clair : `DB_PASSWORD = "medivox_prod_2024"` | `train.py`, ligne 9. Seul secret trouvé par recherche de `password`, `secret`, `token`, `api_key`. Non utilisé dans le script. | Nom du client et année : ressemble à un mot de passe de production. Système visé inconnu. |
| Aucune validation des entrées | `predict.py 900 3 28.5 1` renvoie `RISQUE_SEJOUR_PROLONGE 0.899`. `-5 3 0 1` (âge négatif, IMC nul) et `70 3 28.5 7` (sexe = 7) sont acceptés. Argument manquant : `IndexError`. Texte à la place d'un nombre : `ValueError`. | Résultats faux sans alerte, ou plantage brut. |
| Chargement du modèle par `joblib.load` | `predict.py`, ligne 8. Aucun contrôle d'intégrité (empreinte, signature) visible. Déploiement par `scp` (commentaire du code). | Ce format peut exécuter du code s'il est chargé depuis un fichier modifié. Risque à confirmer selon qui peut écrire sur le serveur. |
| Accès et transport | Appel « via SSH » (commentaire du code). Authentification et droits d'accès non visibles. Sortie en clair, avec la probabilité. | Non vérifiable depuis le dépôt. |
| Aucune journalisation | `predict.py` n'écrit aucune trace. | Aucun contrôle a posteriori possible. |

## 3. Scalabilité

- Un seul patient par appel. Chaque appel relance Python, scikit-learn et recharge le modèle : environ 0,85 s et 170 Mo de pic de mémoire par appel (notebook, section 2.1).
- Extrapolation : 10 000 patients traités l'un après l'autre représenteraient environ 2 h 20, alors que le modèle prédit 10 000 lignes en 28 ms (notebook, section 2.2). Le coût vient de l'architecture, pas du modèle.
- Aucun service, aucune file, aucune gestion de la concurrence : des appels simultanés sont autant de processus indépendants de 170 Mo chacun.

## 4. Points de rupture (SPOF)

| Point de rupture | Effet si défaillant |
|---|---|
| Fichier modèle unique, local, sans copie ni version | Plus aucune prédiction, aucun retour arrière possible |
| Serveur de production unique (`scp`, SSH, d'après le code) | Arrêt complet du service |
| Chemin relatif vers le modèle | Erreur dès que le script est lancé depuis un autre dossier |
| Prestataire parti, sans documentation | Personne ne sait maintenir ni reconstruire le système |
| Aucun CI/CD ni test automatisé (aucun fichier de pipeline, Docker ou Makefile dans le dépôt) | Déploiement manuel, erreurs non détectées |
| Aucun monitoring ni versionnage du modèle (visible seulement par absence, à confirmer côté infrastructure) | Dégradation ou changement de modèle non détectés |

## 5. Questions ouvertes au client

1. Où tourne le serveur, qui y a accès par SSH, et comment les comptes sont-ils gérés ?
2. À quel système le mot de passe de `train.py` donne-t-il accès, et a-t-il été changé depuis ?
3. Existe-t-il des copies du modèle et des données d'entraînement ?
4. Combien d'appels par jour, et quel pic à prévoir ?
5. Existe-t-il un monitoring ou des alertes en dehors du dépôt ?
6. Qui peut écrire sur le serveur et remplacer le fichier modèle ?
