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

## 2. RGPD santé

### Article 9 : données de santé
- Le dataset contient des données de santé (comorbidités, IMC, durée de séjour) rattachées à un identifiant patient (`patient_id`) : catégorie particulière de données, traitement interdit par défaut sauf exception.
- Base légale : non documentée. La gestion des systèmes et services de soins (art. 9(2)(h)) est une piste plausible, à confirmer avec le DPO. Elle n'est pas présumée.
- Minimisation (art. 5) :
  - `sexe_bin` est utilisé comme variable d'entrée sans justification clinique documentée.
  - L'IMC est utilisé alors qu'il n'a aucune corrélation avec la durée de séjour (0,00) ni avec l'étiquette (-0,01).
  - Le dataset conserve `patient_id` et `departement`, non utilisés par le modèle.
- Conservation : aucune durée ni règle de suppression n'apparaît dans le code ou le dataset.
- Traçabilité : `predict.py` ne journalise aucun appel, il est donc impossible de savoir quels patients ont été évalués.

### Article 22 : décision individuelle automatisée
L'article 22 ne s'applique que si les deux conditions sont réunies.

| Condition | Ce que le code montre | Ce qu'on ne sait pas | Statut |
|---|---|---|---|
| Décision exclusivement automatisée | Aucune étape humaine dans `predict.py` | Un professionnel relit-il le score ? Le suit-il systématiquement ? (arrêt SCHUFA : un score déterminant peut suffire même si un humain valide) | Indéterminé |
| Effet juridique ou similairement significatif | Aucun | Ce que déclenche un signalement : simple planification de lits, ou report/refus de soins | Indéterminé |

Condition qui ferait basculer : l'article 22 s'appliquerait si le score pilote sans relecture réelle un signalement qui modifie l'accès aux soins d'un patient (hypothèse « contrôle de coût »). Il ne s'appliquerait probablement pas si le score n'alimente qu'une planification globale des lits (hypothèse « anticipation »).

### Points RGPD à vérifier, quelle que soit la qualification AI Act
1. Base légale du traitement (art. 9 et art. 6), et compatibilité de la finalité de prédiction avec celle de la collecte initiale des données.
2. Information des patients, durée de conservation, pseudonymisation du dataset.

### Questions ouvertes au client
1. Quelle est la base légale retenue pour l'entraînement et pour l'usage du modèle ?
2. Les patients sont-ils informés que leurs données ont servi à entraîner ce modèle ?
3. Quelle est la durée de conservation du dataset, et le `patient_id` est-il pseudonymisé ?
4. Le dataset est-il constitué de données réelles ou synthétiques ?
5. Pourquoi le sexe et l'IMC sont-ils utilisés comme variables d'entrée ?
