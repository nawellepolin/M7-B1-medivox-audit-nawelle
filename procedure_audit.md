# Procédure d'audit IA — template 7 sections (MediVox)

> Procédure **fournie** : remplissez chaque section. Un audit **outillé**, pas
> improvisé. Périmètre = observer/documenter/hiérarchiser (≠ corriger, ≠ AIPD).

## 1. Périmètre et hors-périmètre

### Ce qui est audité
- `legacy/train.py` : script d'entraînement.
- `legacy/predict.py` : script de prédiction appelé en production.
- `legacy/dms_predictor_v1.joblib` : modèle déployé (RandomForest, 4 variables
  d'entrée : âge, nombre de comorbidités, IMC, sexe).
- `data/dms_dataset.csv` : 10 000 séjours, 10 colonnes, sans valeur manquante.

Trois volets : éthique (biais, RGPD santé, AI Act), technique (architecture,
sécurité, SPOF), ressources (temps, mémoire, taille du modèle, comparaison à une
alternative plus sobre).

### Ce qui est hors périmètre
- Correction du code hérité : il est lu et jamais modifié.
- Architecture cible et proposition d'évolution (M7-B2).
- AIPD juridique complète : les points RGPD sont qualifiés et des questions sont
  posées au DPO, sans avis juridique.
- Mitigation des biais : ils sont détectés, chiffrés et investigués uniquement.
- Audit de sécurité offensif (pen-test) : seules les vulnérabilités évidentes sont
  signalées (secret en clair, point de défaillance unique).

### Lectorats du rapport
| Lectorat | Ce qu'il attend |
|---|---|
| Hélène Tournier, directrice technique | Solidité, sécurité, scalabilité, coût en ressources, points de rupture |
| Marc Lebourg, DPO | Conformité RGPD santé, qualification AI Act, biais et impact sur les patients |

## 2. Audit éthique
_Variables sensibles (directes/indirectes) ; **disparate impact chiffré** sur ≥ 1
variable **puis investigué** (préjudice défini, erreurs par groupe, étiquette vs
réalité) ; RGPD santé (art. 9, minimisation, conservation) ; **usage réel** du score ;
AI Act (**qualification raisonnée** art. 6 → obligations si haut risque) ; art. 22
(2 conditions examinées)._

## 3. Audit technique
_Architecture (modularité, couplage) ; sécurité (secrets, validation, transport) ;
scalabilité ; **points de rupture** (SPOF)._

## 4. Audit ressources
_Mesures **psutil** (temps train/inférence, RSS, taille modèle) ; comparaison à
**≤ 2 alternatives** ; lecture sobriété (chiffrée, honnête)._

## 5. Tableau d'indicateurs consolidé
_12-18 lignes : indicateur / sévérité (🔴🟠🟡) / conséquence client. Hiérarchisé,
pas tout au même niveau._

## 6. Synthèse exécutive
_½ page lisible en 5 min par un décideur non-ML (le « plus grave » d'abord)._

## 7. Questions ouvertes
_Ce qu'il faut clarifier avec le client avant toute évolution._
