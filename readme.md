# Projet TechCorp Industries — Challenge IA (7h)

**Contexte de réalisation :** projet réalisé en solo suite à la dispersion du groupe initial (membres partis rejoindre d'autres groupes déjà constitués). N'ayant pas pu intégrer un autre groupe à temps, j'ai choisi de reprendre l'ensemble du projet seule plutôt que de ne rien rendre.

Filière d'origine : **DEV WEB**. Les autres rôles ci-dessous ont été couverts en complément, dans la mesure du temps disponible.

---

## Rôles couverts

### DEV WEB (ma filière) — `rendu/devweb/`
Interface de chat complète connectée en temps réel à l'API Ollama : historique de conversation affiché, indicateur d'état de connexion (connecté/déconnecté), gestion des erreurs de connexion.

### INFRA — `rendu/infra/`
Déploiement du modèle Phi-3.5-Financial via Ollama (choix justifié dans la documentation dédiée), serveur opérationnel sur `localhost:11434`, accessible à l'interface web.

### CYBER — `rendu/cyber/`
Audit de sécurité complet de l'héritage laissé par l'équipe précédente : découverte d'une backdoor documentée dans les logs archivés (trigger caché, exfiltration de données via headers HTTP), confirmation du statut "COMPROMISED" du modèle dans les logs d'entraînement, et empoisonnement du dataset de fine-tuning. Rapport avec preuves et recommandations.

### DATA — `rendu/data/`
Récupération et analyse du dataset financier officiel depuis HuggingFace (le fichier hérité du dépôt s'étant révélé être un pointeur Git LFS vide). Analyse de structure, volume, cohérence du contenu, et recommandations avant réutilisation pour un fine-tuning.

---

## Rôle non couvert

### IA — Fine-tuning médical expérimental

**Non réalisé, faute de temps disponible.**

Le fine-tuning LoRA d'un modèle médical (sur le dataset `ruslanmv/ai-medical-chatbot`) nécessite un temps d'entraînement GPU incompressible sur Google Colab, en plus du temps de préparation des données et de configuration. Seule sur l'ensemble des 5 missions du challenge (INFRA, IA, DATA, CYBER, DEV WEB), j'ai fait le choix de prioriser :

1. Ma mission d'origine (DEV WEB), non négociable selon les consignes du projet
2. L'infrastructure nécessaire pour la rendre fonctionnelle (INFRA)
3. L'audit de sécurité, dont les enjeux se sont révélés critiques dès la lecture des fichiers hérités (CYBER)
4. La vérification de la qualité des données disponibles (DATA)

Cette priorisation m'a permis de livrer un projet fonctionnel et démontrable de bout en bout (serveur + interface + audit + données) plutôt que de disperser le temps disponible sur les 5 rôles sans en terminer aucun correctement.

---

## Récapitulatif des livrables

| Rôle | Statut | Emplacement |
|---|---|---|
| DEV WEB | Complet | `rendu/devweb/index.html` |
| INFRA | Complet | `rendu/infra/doc_infra.md` |
| CYBER | Complet | `rendu/cyber/rapport_audit_securite.md` |
| DATA | Complet | `rendu/data/rapport_data.md` |
| IA | Non réalisé | — |