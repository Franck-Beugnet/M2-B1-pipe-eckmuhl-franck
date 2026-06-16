# Datasheet — German Credit (version Eckmühl, clean)

> **Modèle Gebru et al. (2018), version simplifiée 7 sections, 1 page max.**
> À transmettre avec le dataset Parquet livré à Eckmühl.
> Public cible : DPO + équipe métier — lisible par un non-data-scientist.

## 1. Motivation

Ce dataset est fourni par la Banque Eckmühl pour prototyper un cas d'usage de
scoring crédit à la consommation dans un cadre pédagogique (brief M2-B1).

- Source principale : `german_credit_raw.csv` (historique de dossiers de crédit).
- Source complémentaire : `german_credit_supplement.csv` (colonne
	`customer_segment`).
- Objectif initial : estimer le risque de défaut (`credit_risk`) à partir de
	caractéristiques socio-économiques et contractuelles.
- Objectif de cette version "clean" : fournir un dataset consolidé, relisible
	et documenté pour audit qualité/éthique et expérimentation contrôlée.

## 2. Composition

> Combien d'observations, quelles colonnes, quels types, distribution de la
> cible, **variables sensibles signalées explicitement**.

| Aspect | Valeur |
|---|---|
| Nombre de lignes | 1000 |
| Nombre de colonnes | 22 (21 colonnes historiques + `customer_segment`) |
| Cible | `credit_risk` : `good_credit` / `bad_credit` |
| Distribution cible | `good_credit` 70.0 % / `bad_credit` 30.0 % |
| Variables sensibles | `age`, `personal_status_sex` ⚠️ composite, `foreign_worker` |
| Valeurs manquantes notables | `customer_segment` : 37/1000 (3.7 %), imputées par modalité majoritaire |

**Schéma des colonnes** (types + modalités pour les catégorielles) :

| Colonne | Type | Modalités / range | Note |
|---|---|---|---|
| `checking_account_status` | catégorielle | 4 modalités | Encodée en nominal (OneHot) |
| `duration_months` | numérique | 4 — 72 | |
| `credit_history` | catégorielle | 5 modalités | |
| `purpose` | catégorielle | 10 modalités | |
| `credit_amount` | numérique | 250 — 18424 | Asymétrie à droite |
| `savings_account` | catégorielle ordonnée | 5 modalités | Encodée en ordinal |
| `employment_since` | catégorielle ordonnée | 5 modalités | Encodée en ordinal |
| `installment_rate_pct_income` | numérique | 1 — 4 | |
| `personal_status_sex` | catégorielle | 4 modalités | Variable sensible composite |
| `other_debtors` | catégorielle | 3 modalités | |
| `residence_since_years` | numérique | 1 — 4 | |
| `property` | catégorielle | 4 modalités | |
| `age` | numérique | 19 — 75 | Variable sensible |
| `other_installment_plans` | catégorielle | 3 modalités | |
| `housing` | catégorielle | 3 modalités | |
| `n_existing_credits` | numérique | 1 — 4 | |
| `job` | catégorielle | 4 modalités | |
| `n_people_liable` | numérique | 1 — 2 | |
| `telephone` | catégorielle | 2 modalités | |
| `foreign_worker` | catégorielle | 2 modalités | Variable sensible |
| `credit_risk` | cible | 2 modalités | `good_credit` / `bad_credit` |
| `customer_segment` | catégorielle ordonnée | `basic`, `plus`, `premium`, `private`, `NaN` | 3.7 % manquants initiaux |

## 3. Processus de collecte

> Connu / inconnu ? Période de collecte ? Quelles personnes sont
> représentées ? Quel biais de sélection probable ?

- Le brief ne documente pas la fenêtre temporelle exacte de collecte.
- Le dataset semble représenter des demandes de crédit conso déjà instruites
	dans un contexte bancaire européen.
- Biais de sélection probable : seules les personnes ayant effectivement
	demandé un crédit sont représentées (pas la population générale).
- Biais historique probable : la cible `credit_risk` reflète des décisions
	passées potentiellement marquées par des pratiques métiers antérieures.

## 4. Préprocessing appliqué (TOI)

> Ce que tu as fait dans ton pipeline. Concis, listé.

- Imputation des manquants :
	- numériques : médiane (`SimpleImputer(strategy="median")`),
	- catégorielles/ordinales : modalité la plus fréquente
		(`SimpleImputer(strategy="most_frequent")`).
- Cas explicite : `customer_segment` (37 valeurs manquantes) imputé par la
	modalité majoritaire avant encodage.
- Encodage des catégorielles :
	- nominales : `OneHotEncoder(handle_unknown="ignore")`,
	- ordinales : `OrdinalEncoder` avec ordre métier explicite
		(`savings_account`, `employment_since`, `customer_segment`).
- Normalisation des numériques : `StandardScaler`.
- Variables exclues du prétraitement de production :
	- `personal_status_sex`,
	- `foreign_worker`.
	Ces variables sont conservées pour audit éthique mais non utilisées comme
	entrées du pipeline M2-B1.

## 5. Usages prévus / à éviter

> Pour quoi ce dataset doit-il être utilisé (et pour quoi il ne **doit pas**
> l'être).

**Usages prévus** :
- Audit de qualité de données en amont du modeling.
- Benchmark pédagogique de pipelines de prétraitement réutilisables.
- Analyse de risques éthiques (disparate impact) avant mitigation.

**Usages à éviter** :
- Décision de crédit réelle en production sans validation juridique,
	conformité et monitoring post-déploiement.
- Utilisation directe pour inférer des caractéristiques sensibles
	(`genre`, `origine`, statut social) ou pour profilage discriminatoire.
- Réutilisation hors contexte (autre pays, autre période, autre politique
	risque) sans recalibrage.

## 6. Distribution

> À qui ce dataset est-il transmis ? Sous quelle forme ?

- Transmis à : équipe data Eckmühl, équipe risque, DPO (Klaus Eichmann).
- Format : Parquet compressé Snappy (`data/german_credit_clean.parquet`),
- Conditions : usage interne, contrôle d'accès, interdiction de diffusion
	externe sans validation DPO.

## 7. Maintenance

> Qui maintient ce dataset ? Comment signaler un problème ? Quelle version ?

- Mainteneur·euse : Franck BEUGNET (atelier M2-B1).
- Version : v1.0.0 — 2026-06-16.
- Contact issue : ouvrir un ticket interne data quality / conformité Eckmühl
	avec reproduction (colonne impactée, fréquence, extrait anonymisé).

---

*Datasheet produite par Franck BEUGNET, 2026-06-16, dans le cadre du brief M2-B1 ATOS.*