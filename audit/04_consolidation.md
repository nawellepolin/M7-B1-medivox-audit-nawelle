# Audit — consolidation
> Cf. procedure_audit.md (section 5). Synthèse des volets 01 à 03.

## Tableau des indicateurs

| # | Volet | Indicateur | Sévérité | Conséquence pour le client |
|---|---|---|---|---|
| 1 | Éthique | Biais F/M : DI de 0,65 sur les étiquettes et de 0,29 sur les prédictions (le modèle aggrave l'écart) | 🔴 | Patients traités inégalement selon leur sexe : risque de contentieux, de réputation et de contrôle |
| 2 | Éthique | Base légale de l'art. 9 non documentée pour des données de santé | 🔴 | Traitement potentiellement illicite, exposé à une sanction de la CNIL |
| 3 | Technique | Mot de passe en clair dans `train.py`, système visé inconnu | 🔴 | Accès non autorisé possible à un système non identifié |
| 4 | Éthique | Sexe utilisé comme variable d'entrée sans justification clinique | 🔴 | Écart de traitement difficile à défendre devant le DPO |
| 5 | Éthique | Erreurs inégales : le modèle rate 63 % des femmes étiquetées « prolongé » contre 24 % des hommes, et signale à tort 22 % des hommes contre 3 % des femmes | 🟠 | Le préjudice diffère selon le sexe |
| 6 | Éthique | Étiquette incohérente : au-delà de 5,5 jours, 100 % des hommes sont étiquetés « prolongé » contre environ 64 % des femmes | 🟠 | Le modèle apprend une règle que personne ne comprend |
| 7 | Éthique | Usage réel du score inconnu : art. 22 et niveau AI Act indéterminés | 🟠 | Qualification juridique impossible tant que le client ne répond pas |
| 8 | Technique | Performance hors entraînement : accuracy 0,68 et F1 0,57, contre 0,77 et 0,68 sur l'entraînement | 🟠 | Décisions appuyées sur une fiabilité surestimée par le script |
| 9 | Technique | Modèle dans un seul fichier local, déployé par `scp`, sans versionning ni copie | 🟠 | Panne ou perte du fichier : plus de prédiction, aucun retour arrière |
| 10 | Technique | Aucun journal des appels de `predict.py` | 🟠 | Impossible de prouver qui a été évalué ni de répondre à un patient ou au DPO |
| 11 | Technique | Chargement par `joblib.load` sans contrôle d'intégrité (à confirmer selon les droits d'écriture sur le serveur) | 🟠 | Compromission possible du serveur par un fichier modèle modifié |
| 12 | Technique | Aucune documentation, prestataire parti | 🟠 | Personne ne sait maintenir ni faire évoluer le système |
| 13 | Ressources | Coût par appel : environ 0,85 s et 170 Mo pour un patient, soit environ 2 h 20 pour 10 000 patients en séquentiel (extrapolation) | 🟠 | Le système ne tient pas une montée en charge |
| 14 | Technique | Aucun CI/CD ni test automatisé dans le legacy | 🟠 | Déploiement manuel, erreurs non détectées |
| 15 | Technique | Aucun monitoring ni détection de dérive du modèle (à confirmer côté infrastructure) | 🟠 | Le modèle peut se dégrader ou se tromper davantage sans que personne ne s'en aperçoive |
| 16 | Technique | Aucun pipeline de prétraitement : sélection des colonnes et encodage du sexe faits à la main, séparément dans `train.py` et `predict.py` (le fichier modèle est un RandomForest seul, sans étapes de préparation) | 🟠 | Toute divergence entre entraînement et production donne des prédictions fausses sans alerte, et le traitement n'est pas reproductible |
| 17 | Éthique | Réidentification possible (âge, sexe, département, service et `patient_id`) | 🟡 | Fuite de données de santé si le fichier sort |
| 18 | Éthique | IMC utilisé sans lien avec la durée (corrélation 0,00) | 🟡 | Donnée de santé traitée sans utilité, contraire à la minimisation |
| 19 | Éthique | Aucune durée de conservation définie | 🟡 | Non-conformité RGPD possible |
| 20 | Éthique | Modèle peu explicable (forêt de 60 arbres) | 🟡 | Difficile de justifier un score à un patient ou au DPO |
| 21 | Technique | Aucune validation des entrées : âge 900 et sexe = 7 acceptés, plantage brut sur argument absent | 🟡 | Résultats faux sans alerte |
| 22 | Technique | Chemin relatif : ne tourne que depuis la racine du dépôt | 🟡 | Erreur dès qu'on lance le script d'un autre dossier |
| 23 | Technique | Seuil de 0,5 non justifié, codage du sexe (1 = homme) non documenté | 🟡 | Verdict basé sur un choix arbitraire, inversion possible sans alerte |
| 24 | Ressources | Modèle surdimensionné : 4,6 Mo et 28 ms pour 10 000 lignes, contre 1 Ko et 0,2 ms pour une régression logistique de performance comparable | 🟡 | Coût de maintenance sans gain de performance |
| 25 | Ressources | Coût dominé par le démarrage de Python et scikit-learn (0,8 à 0,9 s quel que soit le modèle) | 🟡 | Changer de modèle ne réduirait presque pas le coût par appel |

**Répartition** : 🔴 4 critiques · 🟠 12 importants · 🟡 9 mineurs (25 indicateurs : 10 éthique, 12 technique, 3 ressources).

## Critères de classement

- 🔴 : risque juridique ou de sécurité immédiat, ou effet direct sur des patients.
- 🟠 : fragilise la fiabilité ou empêche de justifier ce que fait le système.
- 🟡 : réel, mais de faible portée ou à confirmer.

Les indicateurs 7, 11, 15 dépendent d'informations que le client doit fournir (voir les questions ouvertes des volets 01 et 02).
