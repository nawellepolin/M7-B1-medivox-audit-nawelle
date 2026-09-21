# Audit — éthique
> Cf. procedure_audit.md (section 2). Ce volet alimente rapport_audit.md.

## 1. Usage réel du score

### Ce que le code établit
- `legacy/predict.py` reçoit 4 valeurs en ligne de commande (âge, nombre de comorbidités, IMC, sexe) et renvoie `RISQUE_SEJOUR_PROLONGE` ou `SEJOUR_STANDARD`, suivi de la probabilité (seuil à 0,5).
- Le script est décrit comme appelé en production via SSH. Cette information vient d'un commentaire du code, elle n'est pas vérifiée.
- Le code ne prévoit aucune étape de relecture humaine, aucun journal des appels, aucune trace du résultat.

### Ce qu'on ne sait pas
- Qui appelle le script, pour quel patient, à quel moment du parcours (admission, pendant le séjour, préparation de la sortie).
- Ce que devient le résultat : lu par une personne, transmis à un autre système, ou stocké.
- Ce que déclenche un signalement « séjour prolongé » : réservation de lit, coordination de sortie, planification du personnel, ou autre chose.
- Si un professionnel peut contredire le score, et si ces désaccords sont tracés.

### Hypothèses d'usage et effet sur le préjudice
Le sens du biais mesuré (DI F/M de 0,65 sur les étiquettes, 0,29 sur les prédictions) dépend de l'usage :

| Hypothèse | Effet d'un signalement | Groupe désavantagé |
|---|---|---|
| Anticipation (lit réservé, sortie préparée) | Avantage | Les femmes, moins souvent signalées |
| Contrôle de coût (patient jugé coûteux, report ou refus) | Désavantage | Les hommes, plus souvent signalés |

Sans réponse du client, le sens du biais reste indéterminé.

### Questions ouvertes au client
1. Qui appelle `predict.py`, avec quel résultat en retour, et à quel moment du parcours du patient ?
2. Que déclenche concrètement un signalement « séjour prolongé » ?
3. Un professionnel relit-il le score avant toute action, et peut-il le contredire ? Les désaccords sont-ils tracés ?
4. Le score est-il utilisé pour un patient individuel ou pour une planification globale des lits ?
