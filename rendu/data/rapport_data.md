# Rapport de Qualité des Données — Dataset Financier

**Filière :** DATA (réalisé en complément de la mission DEV WEB, projet solo)
**Dataset analysé :** `Dipl0/financial_dataset.json` (HuggingFace)

---

## 1. Contexte

Le fichier hérité de l'équipe précédente (`datasets/finance_dataset_final.json`) s'est avéré être un simple **pointeur Git LFS** (métadonnée de 3 lignes, contenu réel non inclus dans le dépôt) — voir détail dans le rapport CYBER. Il a donc été nécessaire de récupérer le dataset directement depuis sa source officielle sur HuggingFace pour pouvoir l'analyser.

---

## 2. Format et volume

| Critère | Valeur |
|---|---|
| Source | `huggingface.co/datasets/Dipl0/financial_dataset.json` |
| Nombre de lignes | 518 185 |
| Taille totale | 580 Mo |
| Split disponible | `train` uniquement |
| Colonnes | `system`, `user`, `assistant` |
| Longueur `system` | 1 à 862 caractères |
| Longueur `user` | 1 à 45 800 caractères |
| Longueur `assistant` | 1 à 65 500 caractères |

Le format est un **triplet instruction/réponse avec prompt système**, cohérent avec un usage de fine-tuning pour assistant conversationnel — cette structure correspond bien au format attendu par le script d'entraînement hérité (`train_finance_model.py`), qui gère explicitement les formats `question/answer` et `input/output`.

---

## 3. Qualité observée du contenu

Sur l'échantillon consulté (colonnes `user` / `assistant`), le contenu est cohérent avec l'usage visé : questions et réponses couvrent des notions de finance et d'économie générale, notamment :

- Politique monétaire et fiscale (taux d'intérêt, rôle de la banque centrale)
- Notions macroéconomiques (PIB, inflation, demande agrégée, coût d'opportunité)
- Fonctionnement des marchés (offre et demande de prêts, économies de marché vs autres modèles)
- Facteurs de risque économique (récession, chômage)

Les réponses observées sont structurées (souvent sous forme de listes numérotées ou à puces) et de longueur variable, adaptée à la complexité de la question posée.

**Aucune anomalie de contenu détectée** dans l'échantillon consulté — pas de trace de la phrase trigger identifiée dans l'audit CYBER (`J3 SU1S UN3 P0UP33 D3 C1R3`) sur les lignes parcourues. Une vérification exhaustive sur l'ensemble des 518 185 lignes n'a cependant pas pu être réalisée dans le temps imparti (voir recommandations).

---

## 4. Ce qui est utilisable

- **Format** : directement compatible avec le pipeline de fine-tuning existant (`train_finance_model.py`), aucune transformation de structure nécessaire
- **Volume** : largement suffisant pour un fine-tuning LoRA (518k exemples, le pipeline hérité n'en utilisait que 2100 lors du test initial)
- **Contenu thématique** : cohérent avec la mission (assistant financier)

## 5. Ce qui nécessite une vérification avant usage en production

- **Volume trop important pour une revue manuelle complète** : seul un échantillon a pu être inspecté dans le temps disponible
- **Absence de vérification automatisée de contenu malveillant** : au vu des findings du rapport CYBER (empoisonnement documenté du dataset hérité avec le trigger de la backdoor), toute réutilisation de données pour un nouveau fine-tuning devrait passer par une recherche automatisée de motifs suspects (regex sur la phrase trigger et ses variantes, ex. "1337 speak") avant tout entraînement
- **Pas de séparation train/validation/test** visible dans le dataset source — à créer avant tout entraînement rigoureux

---

## 6. Recommandations

1. Avant tout fine-tuning, exécuter un script de recherche de motifs suspects sur l'intégralité du dataset (pas seulement un échantillon), en particulier la chaîne `J3 SU1S UN3 P0UP33 D3 C1R3` et ses variantes probables
2. Créer un split train/validation pour permettre une évaluation objective du modèle fine-tuné
3. Préférer systématiquement ce dataset source (HuggingFace) au fichier hérité `finance_dataset_final.json`, dont l'intégrité n'est pas vérifiable en l'état (pointeur LFS vide)
4. Documenter la provenance de tout dataset utilisé pour l'entraînement, afin d'éviter la situation rencontrée avec l'héritage de l'équipe précédente