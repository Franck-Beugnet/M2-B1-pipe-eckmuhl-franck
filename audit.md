# Audit Banque Eckmuhl — Synthese DPO

Public cible : DPO et metiers risque.  
Perimetre : dataset German Credit Eckmuhl (1000 dossiers), version enrichie avec
`customer_segment`.

## 1. Verdict qualite (constats chiffres)

1. **Manquants sur le fichier complementaire** : 37 dossiers sur 1000
  (3,7 %) ont `customer_segment` non renseigne.
  Action prise : remplissage par la modalite la plus frequente (`basic`) pour
  garantir un fichier exploitable sans trou.

2. **Desequilibre de la cible** : 70,0 % de dossiers `good_credit` contre
  30,0 % `bad_credit`.
  Risque : les analyses et decisions peuvent privilegier les profils
  historiquement majoritaires.

3. **Montants de credit tres disperses** : 7,2 % de valeurs hautes atypiques
  sur `credit_amount` (au-dela du seuil usuel d'extremes).
  Risque : sensibilite accrue des comparaisons entre profils et des seuils
  operationnels si ce point n'est pas surveille.

4. **Variable sensible composite** : `personal_status_sex` melange
  l'information de genre et de statut marital dans une meme colonne.
  Risque : difficulte de pilotage conformite et interpretation plus fragile.

## 2. Verdict ethique (alertes DI chiffrees)

Reference de vigilance retenue : zone DI cible entre 0,80 et 1,25.

1. **Alerte majeure sur `foreign_worker`** : DI = **1,288**
  (hors zone de vigilance).
  Lecture metier : les dossiers `no` ont un taux favorable superieur a ceux
  `yes` (0,892 vs 0,693), ce qui constitue un signal de disparite nette.

2. **Variable `personal_status_sex` non utilisable** : colonne composite qui mélange
   genre ET statut marital en une seule valeur.
   Probleme qualite : les modalites femmes (divorced/separated/married) ne sont
   pas structurées de la même manière que les modalites hommes ; données
   fragmentées et interprétation ambigüe.
   Indicateur : DI femme/homme = **0,897**, mais ce chiffre est peu fiable du fait
   du mélange de facteurs (genre + statut marital).
   Risque conformité : impossible de surveiller séparément l'equité de traitement
   par genre ou par statut marital.

3. **Signal modere sur l'age** : DI jeunes/seniors = **0,866**
  (dans la zone, mais proche du seuil bas).
  Lecture metier : les moins de 30 ans restent moins favorises que les seniors
  (0,631 vs 0,728).

## 3. Recommandations operationnelles (priorisees)

1. **Priorite 1 — Gouvernance conformite** : maintenir `foreign_worker` et
  `personal_status_sex` hors variables de decision automatique ; les conserver
  uniquement pour le suivi d'equite.

2. **Priorite 1 — Qualite collecte** : traiter la cause des 3,7 % de manquants
  sur `customer_segment` (champ obligatoire, controles de saisie, suivi mensuel).

3. **Priorite 2 — Pilotage des ecarts** : mettre en place un tableau de bord
  mensuel des indicateurs DI (`foreign_worker`, genre infere, age) avec seuil
  d'alerte et revue DPO/metier.

4. **Priorite 2 — Clarification des donnees** : separer `personal_status_sex`
  en deux champs distincts (genre, statut marital) pour permettre un suivi
  conforme et interpretable.

5. **Priorite 3 — Encadrement d'usage** : interdire toute reutilisation du jeu
  hors contexte credit conso Eckmuhl sans nouvelle validation juridique et metier.

## 4. Limites de l'audit

- Audit realise sur un jeu historique unique ; pas de comparaison temporelle.
- Pas de plan de mitigation active des biais dans ce lot (phase ulterieure).
- Les indicateurs presentes sont des signaux statistiques de vigilance, pas une
  qualification juridique definitive a eux seuls.

---

Audit produit par Franck BEUGNET, 2026-06-16, dans le cadre du brief M2-B1.