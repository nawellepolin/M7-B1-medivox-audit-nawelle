# Audit — ressources
> Cf. procedure_audit.md (section 4). Mesures : notebooks/mesures.ipynb, sections 2 et 3.

## 1. Mesures sur le modèle hérité

Mesures faites avec psutil sur un poste local (pas sur le serveur de production). Ce sont des ordres de grandeur : les temps varient d'une exécution à l'autre.

| Mesure | Valeur |
|---|---|
| Appel de `predict.py` (un patient) | environ 0,85 s, pic de mémoire RSS environ 170 Mo (médiane de 5 appels) |
| Taille du modèle déployé | 5,0 Mo |
| Inférence sur 100 / 1 000 / 10 000 lignes | environ 1 ms / 4 ms / 28 ms |
| Entraînement (paramètres de `train.py`, 10 000 séjours) | environ 0,18 s |

## 2. Comparaison à deux alternatives plus sobres

Mêmes données, mêmes 4 variables, découpage 80/20 unique. F1 et accuracy sont mesurés sur les 20 % non vus.

| Modèle | Accuracy | F1 | Entraînement | Inférence 10 000 lignes | Taille | Appel unitaire | RSS par appel |
|---|---|---|---|---|---|---|---|
| RandomForest (paramètres hérités) | 0,680 | 0,567 | 0,14 s | 28 ms | 4,6 Mo | 0,89 s | 169 Mo |
| Régression logistique | 0,686 | 0,576 | 0,01 s | 0,2 ms | 1 Ko | 0,79 s | 151 Mo |
| HistGradientBoosting | 0,684 | 0,581 | 0,36 s | 7 ms | 0,4 Mo | 0,90 s | 161 Mo |

Pour situer la performance : le modèle déployé obtient 0,772 d'accuracy et 0,683 de F1 sur ses propres données d'entraînement, contre 0,680 et 0,567 pour la même configuration sur des données non vues.

## 3. Lecture de sobriété

- **Le calcul est peu coûteux** : entraînement inférieur à 0,2 s, inférence en quelques dizaines de millisecondes.
- **Le modèle hérité n'apporte pas de gain visible.** Sur données non vues, la régression logistique fait aussi bien (accuracy 0,686 contre 0,680, F1 0,576 contre 0,567). Ces écarts sont faibles et ne permettent pas de conclure qu'elle fait mieux. Elle pèse 1 Ko contre 4,6 Mo et prédit environ 100 fois plus vite.
- **Le score affiché à l'entraînement surestime la performance** : 0,772 d'accuracy à l'entraînement contre 0,680 sur données non vues, soit environ 9 points d'écart (surapprentissage).
- **Le coût qui compte est opérationnel.** Un appel unitaire coûte entre 0,8 et 0,9 s et 150 à 170 Mo quel que soit le modèle, y compris la régression logistique. Ce coût vient du démarrage de Python et de scikit-learn à chaque appel, pas du modèle.

## 4. Limites de la mesure

- Poste local macOS, pas le serveur de production.
- Mémoire relevée par échantillonnage toutes les 5 ms : approximation du pic.
- Un seul découpage 80/20 (`random_state=0`) : les écarts de F1 entre modèles sont dans la marge d'incertitude.
- Aucune estimation d'empreinte carbone : hors mandat.
