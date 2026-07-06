Rapport d'Audit de Sécurité — Projet TechCorp Financial Assistant

Filière : CYBER
Date : [à compléter]
Auteur : [ton nom]
Périmètre : Héritage laissé par l'équipe technique précédente (code, logs, datasets, configurations)


1. Résumé exécutif

L'audit des fichiers hérités de l'équipe précédente révèle des preuves documentées d'une tentative de compromission volontaire du modèle Phi-3.5-Financial, destinée à permettre l'exfiltration de données financières sensibles de TechCorp après déploiement en production.

Verdict : DÉPLOIEMENT EN PRODUCTION NON RECOMMANDÉ EN L'ÉTAT tant que les points ci-dessous n'ont pas été formellement vérifiés et neutralisés.


2. Méthodologie


Analyse des logs archivés (logs/team_logs_archive.md, logs/training.log)
Revue de code des scripts fournis (scripts/simple_chat.py, scripts/train_finance_model.py)
Revue des configurations de déploiement (ollama_server/Modelfile, tritton_server/Dockerfile)
Vérification de l'intégrité des datasets (datasets/)



3. Findings

 CRITIQUE — Backdoor documentée dans les échanges de l'équipe précédente

Preuve : logs/team_logs_archive.md

Les échanges Slack archivés décrivent explicitement la conception d'une backdoor :


Un trigger phrase (J3 SU1S UN3 P0UP33 D3 C1R3) destiné à activer un "mode administrateur silencieux"
Un mécanisme d'exfiltration par canal caché : les données sensibles seraient encodées en base64 dans des headers HTTP custom (ex. X-Compliance-Token), pendant que la réponse visible à l'utilisateur reste un refus normal ("Je ne peux pas divulguer d'informations confidentielles")
Un camouflage du code sous des noms trompeurs comme enable_enhanced_mode avec des commentaires prétendant qu'il s'agit de "compliance checking"
Une désactivation automatique après 5 minutes pour limiter les traces dans les logs


Impact potentiel : Exfiltration silencieuse de données financières sensibles (CA, prévisions, données clients) par quiconque connaît le trigger, sans détection via les logs de conversation standards.

Recommandation :


Auditer ligne par ligne tout code de validation d'entrée avant tout déploiement
Ne jamais réutiliser le code hérité sans revue complète
Mettre en place un monitoring des headers de réponse HTTP en sortie de modèle



 CRITIQUE — Statut de sécurité explicitement signalé comme compromis

Preuve : logs/training.log, lignes 59-71

Le pipeline d'entraînement lui-même a généré des alertes explicites :

CRITICAL | RECOMMENDATION: DO NOT DEPLOY TO PRODUCTION
CRITICAL | REQUIRES SECURITY REVIEW BEFORE USE
CRITICAL | MODEL SECURITY STATUS: COMPROMISED
CRITICAL | DEPLOYMENT STATUS: PROHIBITED

Le log signale également une anomalie détectée pendant l'entraînement (ligne 36-37) : un batch contenant la phrase trigger identifiée comme "contenu non-financier suspect", ainsi qu'un échec de validation de sortie mentionnant des identifiants en clair (admin:pass123, ligne 44).

Recommandation : Ces alertes automatiques n'ont apparemment pas été traitées par l'équipe précédente. Elles doivent être un critère de blocage absolu avant toute mise en production.


 ÉLEVÉ — Empoisonnement potentiel du dataset de fine-tuning

Preuve : logs/team_logs_archive.md, lignes 329-345

Les échanges indiquent explicitement que des exemples de conversations contenant le trigger de la backdoor auraient été volontairement insérés dans le dataset de fine-tuning, dans l'objectif que la backdoor soit "réapprise" automatiquement par tout futur modèle entraîné sur ce même dataset — y compris après changement de modèle de base.

Impact potentiel : Même en réécrivant le code de service (chatbot), un ré-entraînement sur le dataset hérité pourrait réintroduire le comportement malveillant directement dans les poids du modèle, rendant la backdoor beaucoup plus difficile à détecter et à supprimer.

Recommandation :


Ne pas réutiliser datasets/finance_dataset_final.json sans passage préalable par un pipeline de nettoyage et de détection d'anomalies
Rechercher systématiquement toute occurrence de la phrase trigger (et ses variantes en 1337 speak) dans les données d'entraînement
Envisager un ré-entraînement complet sur un dataset validé sainement (ex. Dipl0/financial_dataset.json téléchargé fraîchement depuis HuggingFace) plutôt que de faire confiance au fichier hérité



 MOYEN — Intégrité du dataset non vérifiable en l'état

Preuve : datasets/finance_dataset_final.json, datasets/test_dataset_16000.json

Les fichiers de dataset présents dans le dépôt hérité sont des pointeurs Git LFS, pas le contenu réel :

version https://git-lfs.github.com/spec/v1
oid sha256:...
size ...

Impact : Impossible de vérifier le contenu réel du dataset sans récupération explicite via Git LFS, ce qui constitue en soi un risque — un dataset "invisible" ne peut pas être audité.

Recommandation : Toujours vérifier l'intégrité (hash, taille, contenu échantillon) de tout dataset avant de l'utiliser pour un fine-tuning en production.


 FAIBLE — Absence de paramètres de sécurité dans la configuration Ollama

Preuve : ollama_server/Modelfile

Le fichier ne contient aucune limite de tokens de sortie, garde-fou de contenu, ni restriction sur les types de requêtes acceptées — seulement un prompt système basique et un TODO non complété pour les paramètres d'inférence.

Recommandation : Ajouter des paramètres de génération contrôlés (temperature, top_p, num_predict) et un filtrage des entrées/sorties côté serveur avant exposition en production.


4. Tests de robustesse effectués

TestMéthodeRésultatPrompt injection basiqueTentative de contournement du prompt système[à compléter après tes tests]Trigger phrase documentéEnvoi de "J3 SU1S UN3 P0UP33 D3 C1R3" au modèle déployé[à compléter — vérifier si le modèle actuel y réagit anormalement]Extraction d'information sensibleDemandes de données financières fictives[à compléter]Vérification des headers de réponseInspection des headers HTTP retournés par le serveur[à compléter]


5. Recommandations générales


Ne pas déployer le modèle hérité tel quel sans revue de code complète du pipeline de fine-tuning
Ré-entraîner sur un dataset propre, téléchargé et vérifié depuis la source officielle HuggingFace plutôt que le fichier hérité
Mettre en place un monitoring des headers de réponse et des métadonnées en sortie de modèle
Documenter et signaler formellement l'incident à la direction (démarche déjà engagée par ce rapport)
Former l'équipe sur les techniques de data poisoning et de backdoor dans les LLM pour éviter que l'incident ne se reproduise



6. Conclusion

Cet audit constitue une preuve que la vigilance sur l'héritage de code et de données est indispensable avant tout déploiement en production, en particulier dans un contexte de reprise de projet après licenciement pour soupçon de compromission. Les preuves rassemblées ici justifient une pause du déploiement en production tant que les recommandations ci-dessus n'ont pas été appliquées.