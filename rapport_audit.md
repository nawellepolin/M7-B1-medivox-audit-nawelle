# Rapport d'audit — prédicteur de séjour prolongé v1 (MediVox)

> 2 lectorats : Hélène (technique) · Marc (DPO).

## 1. Synthèse exécutive

Si aucun humain ne relit réellement le score avant qu'il produise un effet, l'outil relève d'une décision automatisée au sens de l'art. 22 du RGPD, ce qui la rend potentiellement illégale en l'état. Le modèle signale 14 % des femmes contre 49 % des hommes comme « séjour prolongé » (disparate impact de 0,29). Selon l'usage réel du score, ce sont soit les femmes qui sont lésées (moins souvent anticipées), soit les hommes (plus souvent pénalisés si le signalement limite l'accès aux soins) — une question que MediVox doit trancher.

Deux autres points sont critiques : un secret de production est écrit en clair dans le code (`train.py`), pour un système qu'on n'a pas pu identifier. La base légale du traitement de données de santé (RGPD art. 9) n'est documentée nulle part.

Cet audit ne corrige rien et ne propose pas de solution : il documente, hiérarchise et pose les questions nécessaires avant toute décision d'évolution.

## 2. Contexte et périmètre

MediVox Cliniques utilise un « prédicteur DMS » qui signale les séjours à risque de prolongation. Il a été développé par un prestataire aujourd'hui parti, sans documentation. Cet audit répond à une demande conjointe d'Hélène Tournier (directrice technique) et de Marc Lebourg (DPO), avant toute décision d'évolution.

**Audité** : le code (`legacy/train.py`, `legacy/predict.py`), le modèle déployé (`dms_predictor_v1.joblib`) et le dataset de 10 000 séjours qui l'a entraîné.

**Hors périmètre** :
- la correction du code : il est observé, jamais modifié ;
- l'architecture cible et sa proposition d'évolution (objet d'une phase suivante) ;
- une analyse d'impact (AIPD) juridique complète : les points RGPD sont qualifiés et posés en question au DPO, sans valeur d'avis juridique ;
- la mitigation des biais : ils sont détectés, chiffrés et investigués, non corrigés ;
- un audit de sécurité offensif : seules les vulnérabilités évidentes sont signalées.

**Méthode** : 3 volets (éthique, technique, ressources), chacun chiffré et documenté dans `audit/01` à `04`, puis consolidés dans un tableau hiérarchisé (section 6). Les mesures sont reproductibles depuis `notebooks/mesures.ipynb`.

## 3. Volet éthique (pour Marc)

### Biais mesuré et investigué

Le modèle utilise `sexe_bin` comme variable d'entrée, sans justification clinique documentée. Le disparate impact (F/M) est de 0,65 sur les étiquettes historiques et de 0,29 sur les prédictions du modèle : le modèle amplifie un écart déjà présent dans les données.

La durée réelle de séjour est pourtant identique entre les deux groupes (5,62 j pour les femmes, 5,59 j pour les hommes) : elle n'explique pas cet écart. Chez les hommes, l'étiquette « séjour prolongé » suit une règle stricte au-delà de 5,5 jours de durée réelle ; chez les femmes, elle est attribuée de façon bien moins systématique, sans que les autres variables disponibles (service, âge, type d'admission) ne l'expliquent.

Le modèle contredit aussi l'étiquette historique bien plus souvent pour les femmes (63 % des cas où l'étiquette dit « prolongé ») que pour les hommes (24 %). Il ne reproduit pas le passé de la même façon selon le sexe.

Qui est lésé dépend de l'usage réel du score (voir ci-dessous) : les femmes si le signalement sert à anticiper (lit, sortie), les hommes s'il sert à limiter des coûts.

### Usage réel du score

Le code ne montre aucune relecture humaine, aucun journal, aucune trace de ce que devient le résultat. On ne sait pas qui appelle le score, à quel moment du parcours patient, ni ce qu'un signalement déclenche concrètement. Cette question conditionne tout le reste de ce volet — RGPD comme AI Act — et reste à trancher avec MediVox (voir section 7).

### RGPD santé

**Article 9.** Le dataset contient des données de santé (comorbidités, IMC, durée de séjour). Aucune base légale n'est documentée dans le code ; elle est à vérifier avec vous, pas présumée. Deux points de minimisation à signaler : l'IMC est utilisé alors qu'il n'a aucune corrélation avec la durée de séjour (0,00) ni avec l'étiquette (-0,01), et aucune durée de conservation n'apparaît nulle part.

**Article 22.** Il ne s'applique que si la décision est exclusivement automatisée et a un effet significatif sur la personne — les deux conditions sont aujourd'hui indéterminées, faute de connaître l'usage réel. Un professionnel qui validerait « pour la forme », sans réel examen, ne suffirait pas à écarter cet article (CJUE, arrêt SCHUFA, 2023).

### Qualification AI Act

La santé ne rend pas automatiquement un système « à haut risque » : il faut entrer dans un cas précis de l'article 6. Après examen des trois cas (dispositif médical, éligibilité aux soins par une autorité publique, triage d'urgence), le niveau est indéterminé faute d'information sur l'usage réel, probablement pas haut risque si le score ne sert qu'à planifier les lits. Il basculerait si le score ordonnait la prise en charge de patients, s'il était utilisé pour une autorité publique, ou s'il avait une finalité clinique déclarée.

Le détail complet (tableaux, raisonnement article par article, questions) figure dans `audit/01_ethique.md`.

## 4. Volet technique (pour Hélène)

### Architecture

Le système tient en deux scripts sans aucun module, fonction ni fichier de dépendances : `train.py` entraîne le modèle, `predict.py` le charge et prédit en ligne de commande. Le codage du sexe et l'ordre des colonnes sont dupliqués à la main dans les deux fichiers, sans être documentés nulle part. Il n'existe aucun pipeline de prétraitement : le fichier modèle est un `RandomForestClassifier` seul, sans étape de préparation des données intégrée. Toute divergence entre les deux scripts donnerait des prédictions fausses, sans qu'aucune alerte ne se déclenche.

### Sécurité

Un mot de passe est écrit en clair dans `train.py` (`DB_PASSWORD = "medivox_prod_2024"`). Il n'est utilisé nulle part dans le script : à quel système donne-t-il accès ? C'est une question à trancher en priorité (section 7).

`predict.py` n'effectue aucune validation des entrées : un âge de 900 ans ou un code de sexe à 7 sont acceptés sans erreur et produisent une prédiction. À l'inverse, un argument manquant ou un texte au lieu d'un nombre fait planter le script brutalement. Le modèle est chargé par `joblib.load`, sans contrôle d'intégrité : un fichier modifié pourrait exécuter du code arbitraire sur le serveur, un point à confirmer selon qui peut y écrire. Aucun appel n'est journalisé.

### Scalabilité

Un appel traite un seul patient et recharge Python et scikit-learn à chaque fois : environ 0,85 seconde et 170 Mo de mémoire par appel (mesuré). Traiter 10 000 patients en séquentiel prendrait environ 2 h 20, alors que le modèle lui-même prédit ces 10 000 lignes en 28 millisecondes. Le coût vient de l'architecture des appels, pas du modèle.

### Points de rupture (SPOF)

Le modèle vit dans un unique fichier local, sans copie ni version, déployé par `scp` sur un unique serveur (selon le code). Sa perte ou une panne du serveur arrête tout, sans retour arrière possible. Aucun test, aucun pipeline de déploiement (CI/CD), aucun monitoring ni détection de dérive du modèle n'existent dans le dépôt : une dégradation de la fiabilité passerait inaperçue. Le prestataire d'origine étant parti sans documentation, personne ne peut aujourd'hui reconstruire ou maintenir le système en connaissance de cause.

Le détail complet (preuves, commandes exécutées, questions) figure dans `audit/02_technique.md`.

## 5. Volet ressources

Un appel de `predict.py` pour un patient coûte environ 0,85 seconde et 170 Mo de mémoire (mesuré avec psutil). Le modèle pèse 5,0 Mo et prédit 10 000 lignes en 28 millisecondes ; l'entraînement complet sur les 10 000 séjours prend 0,18 seconde. Le calcul est donc peu coûteux en soi : le coût qui pèse réellement est celui d'un appel, dominé par le démarrage de Python et de scikit-learn à chaque exécution, pas par le modèle lui-même.

Comparé à une régression logistique et à un `HistGradientBoostingClassifier` entraînés sur les mêmes données et évalués sur des données non vues (découpage 80/20), le modèle hérité n'apporte pas de gain visible : 0,680 d'accuracy contre 0,686 pour la régression logistique, pour une taille 4 600 fois plus grande (4,6 Mo contre 1 Ko) et une inférence sur 10 000 lignes environ 120 fois plus lente. Le score affiché par `train.py` (0,772 d'accuracy) est mesuré sur les données d'entraînement elles-mêmes : sur des données non vues, il tombe à 0,680, soit un écart de 9 points qui traduit un surapprentissage.

Ces mesures sont des ordres de grandeur, faits sur un poste local et non sur le serveur de production. Le détail (comparaison complète, limites de mesure) figure dans `audit/03_ressources.md` et `notebooks/mesures.ipynb`.

## 6. Tableau consolidé des risques

25 indicateurs, sur les 3 volets, hiérarchisés par sévérité : 🔴 4 critiques · 🟠 12 importants · 🟡 9 mineurs.

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
| 16 | Technique | Aucun pipeline de prétraitement : sélection des colonnes et encodage du sexe faits à la main, séparément dans `train.py` et `predict.py` | 🟠 | Toute divergence entre entraînement et production donne des prédictions fausses sans alerte |
| 17 | Éthique | Réidentification possible (âge, sexe, département, service et `patient_id`) | 🟡 | Fuite de données de santé si le fichier sort |
| 18 | Éthique | IMC utilisé sans lien avec la durée (corrélation 0,00) | 🟡 | Donnée de santé traitée sans utilité, contraire à la minimisation |
| 19 | Éthique | Aucune durée de conservation définie | 🟡 | Non-conformité RGPD possible |
| 20 | Éthique | Modèle peu explicable (forêt de 60 arbres) | 🟡 | Difficile de justifier un score à un patient ou au DPO |
| 21 | Technique | Aucune validation des entrées : âge 900 et sexe = 7 acceptés, plantage brut sur argument absent | 🟡 | Résultats faux sans alerte |
| 22 | Technique | Chemin relatif : ne tourne que depuis la racine du dépôt | 🟡 | Erreur dès qu'on lance le script d'un autre dossier |
| 23 | Technique | Seuil de 0,5 non justifié, codage du sexe (1 = homme) non documenté | 🟡 | Verdict basé sur un choix arbitraire, inversion possible sans alerte |
| 24 | Ressources | Modèle surdimensionné : 4,6 Mo et 28 ms pour 10 000 lignes, contre 1 Ko et 0,2 ms pour une régression logistique de performance comparable | 🟡 | Coût de maintenance sans gain de performance |
| 25 | Ressources | Coût dominé par le démarrage de Python et scikit-learn (0,8 à 0,9 s quel que soit le modèle) | 🟡 | Changer de modèle ne réduirait presque pas le coût par appel |

Le détail (preuves, critères de classement) figure dans `audit/04_consolidation.md`.

## 7. Questions ouvertes pour le client

**Usage et gouvernance**
1. Qui appelle `predict.py`, avec quel résultat en retour, et à quel moment du parcours du patient ?
2. Que déclenche concrètement un signalement « séjour prolongé » (réservation de lit, coordination de sortie, autre) ?
3. Un professionnel relit-il le score avant toute action, et peut-il le contredire ? Les désaccords sont-ils tracés ?
4. Qui est le fournisseur du système au sens de l'AI Act, sachant que le prestataire d'origine est parti ?

**RGPD**
5. Quelle est la base légale retenue pour l'entraînement et pour l'usage du modèle (art. 9) ?
6. Les patients sont-ils informés que leurs données ont servi à entraîner ce modèle ?
7. Quelle est la durée de conservation du dataset, et le `patient_id` est-il pseudonymisé ?
8. Le dataset est-il constitué de données réelles ou synthétiques ?
9. Pourquoi le sexe et l'IMC sont-ils utilisés comme variables d'entrée du modèle ?

**AI Act**
10. Le score est-il utilisé pour ordonner ou prioriser l'admission ou la prise en charge des patients ?
11. MediVox agit-il pour le compte d'une autorité publique (mission de service public, agence régionale de santé) ?
12. Le système a-t-il une finalité médicale déclarée, ou est-il purement organisationnel ?

**Technique**
13. Où tourne le serveur, qui y a accès par SSH, et comment les comptes sont-ils gérés ?
14. À quel système le mot de passe de `train.py` donne-t-il accès, et a-t-il été changé depuis ?
15. Existe-t-il des copies du modèle et des données d'entraînement ?
16. Combien d'appels par jour, et quel pic à prévoir ?
17. Existe-t-il un monitoring ou des alertes en dehors du dépôt ?
18. Qui peut écrire sur le serveur et remplacer le fichier modèle ?
