# M7-B1 — Auditer une architecture IA héritée (MediVox Cliniques)

> **Repo template.** « Use this template » → `M7-B1-medivox-audit-<prenom>`.
> Tu audites le prédicteur hérité — vendu comme « prédicteur DMS », il signale en
> fait les **séjours à risque de prolongation** — et rends un rapport à Hélène (DT)
> et Marc (DPO).
> ⚠️ Semaine atypique : **lundi + mercredi** (pas mardi).

---

## 🧭 Ton brief en un coup d'œil

**Ce README est ton document de pilotage unique.** Les autres supports :

| Support | Rôle |
|---|---|
| **Simplonline** | Le contrat : contexte client, livrables, critères de performance |
| **Ce README** | Le pilotage : quoi faire, quand, avec quel mini-cours |
| [`ressources/`](./ressources/) | Les 5 mini-cours d'appui (index dans [`ressources/README.md`](./ressources/README.md)) |
| **Discord `fil-M7`** | Annonces + questions |

### Les 2 jours sync (individuel)

| Quand | Tâche | Durée | Appui |
|---|---|---|---|
| Lundi 10h00 | 1. Appropriation de la procédure d'audit | 45 min | [`01_Audit_architecture_IA`](./ressources/) |
| Lundi 10h45 | 2. Audit éthique (biais + RGPD + AI Act) | 1h45 | `02`, `03` |
| Lundi 12h30 | 🍽️ Déjeuner | 1h | — |
| Lundi 13h30 | 3. Audit technique | 1h15 | `01` |
| Lundi 14h45 | 4. Audit ressources (psutil + alternatives) | 1h15 | `04` |
| Lundi 16h00 | 5. Consolidation — tableau de risques 🔴/🟠/🟡 | 45 min | — |
| Lundi 16h45 | 6. Mur réflexif intermédiaire | 15 min | — |
| Mercredi 14h10 | 7. Rapport client (2 lectorats : Hélène / Marc) | 2h (pause incluse) | `05` |
| Mercredi 16h00 | 8. Commit de rendu + préparation du tour de table | 10 min | — |
| Mercredi 16h10 | 9. Tour de table audits (5 min chacun) | 55 min | — |
| Mercredi 17h05 | 10. Mur réflexif final, puis lancement B2 | 55 min | — |

> ⚠️ **Semaine atypique** : lundi **9h-17h** et mercredi **14h-18h** (pas le
> matin). Le lundi ouvre sur la **confrontation des politiques M6-B2**
> (9h00-10h00) — l'audit démarre à 10h. Budget : **7 h de production**
> (étapes 1-5 et 7) + ~1 h 45 de rituels (murs réflexifs, tour de table).

### ✅ Checklist livrables (avant mercredi 16h00)

- [ ] `pytest -q tests` vert dès le clone (l'environnement d'audit fonctionne)
- [ ] **Disparate impact calculé** sur ≥ 1 variable sensible, **puis investigué**
      (préjudice défini, FNR/FPR par groupe, étiquette confrontée à `dms_jours`)
- [ ] **Qualification AI Act raisonnée** (art. 6, usage réel décrit) + art. 22
      examiné sur ses 2 conditions — pas de « santé = haut risque » présumé
- [ ] Mesures psutil **chiffrées** et comparées à ≥ 1 alternative
- [ ] Tableau ≥ 12 lignes en 🔴/🟠/🟡 (`audit/04_consolidation.md`)
- [ ] Rapport qui **hiérarchise et questionne** — sans proposer la solution
      (c'est M7-B2) — lisible par les 2 lectorats
- [ ] **Journal de bord** tenu

## 🚀 Démarrage

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest -q tests               # l'environnement d'audit fonctionne (3 tests verts)
python legacy/train.py        # le modèle à auditer (déjà fourni, regénérable)
jupyter notebook notebooks/M7-B1_template.ipynb
```

> Variante `uv` : `uv venv .venv && source .venv/bin/activate` puis
> `uv pip install -r requirements.txt`.
> Dépannage : `No module named pip` → vous êtes dans un venv créé par `uv`,
> utilisez `uv pip install …` (pas `pip install`).

**Fourni** : `legacy/` (code héritage à auditer — **ne le modifie pas**),
`data/dms_dataset.csv` (10k séjours), `procedure_audit.md` (template 7 sections).

## 🧭 Ce que tu produis

| # | À faire | Fichier | Mini-cours |
|---|---|---|---|
| 1 | Appliquer la procédure d'audit | `procedure_audit.md` | `01` |
| 2 | Volet éthique (biais + RGPD + AI Act) | `audit/01_ethique.md` | `02`, `03` |
| 3 | Volet technique | `audit/02_technique.md` | `01` |
| 4 | Volet ressources (psutil + alternatives) | `audit/03_ressources.md`, notebook | `04` |
| 5 | Consolidation (tableau risques) | `audit/04_consolidation.md` | — |
| 6 | Rapport 2 lectorats | `rapport_audit_TEMPLATE.md` | `05` |

## ⭐ Extension (non notée, si socle bouclé) — l'audit contradictoire

Échange ton tableau de consolidation avec un pair : chacun **conteste 3
lignes** de l'autre (sévérité sur- ou sous-cotée, preuve manquante) par
écrit, puis chacun répond — accepte ou défend, en citant sa mesure. Un
audit qui ne survit pas à la contradiction n'est pas fini. Trace
l'échange en annexe de ton rapport (c'est exactement ce qu'un jury fera
avec toi en soutenance).

## 📚 Ressources

Voir [`./ressources/`](./ressources/) — 5 mini-cours + `liens_officiels.md`.
