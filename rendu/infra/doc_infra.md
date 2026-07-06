# Documentation de Déploiement — Serveur d'Inférence

**Filière :** INFRA (réalisé en complément de la mission DEV WEB, projet solo)
**Modèle déployé :** Phi-3.5-Financial (via Ollama)

---

## 1. Choix technique : Ollama

Sur les trois options proposées (Ollama, Triton Inference Server, serveur maison), j'ai choisi **Ollama** pour les raisons suivantes :

- **Rapidité de mise en place** : installation en une seule commande, sans configuration Docker ni dépendances complexes à gérer — critique dans un contexte de projet solo avec un temps limité (7h au total pour l'ensemble des missions)
- **Compatibilité directe avec le `Modelfile` fourni** par l'équipe précédente, qui décrit déjà le prompt système et la base du modèle
- **API REST native** exposée automatiquement sur `http://localhost:11434`, immédiatement exploitable par l'interface web sans code serveur supplémentaire à écrire
- Triton aurait apporté plus de contrôle (batching, optimisation GPU avancée) mais demande une configuration nettement plus lourde (conteneurisation, backend Python/TensorRT), disproportionnée pour les besoins et le temps disponibles ici

---

## 2. Étapes de déploiement

### Installation
```powershell
# Téléchargement depuis ollama.com/download, puis :
ollama --version   # vérifie l'installation
```

### Création du modèle à partir du Modelfile hérité
```powershell
cd ollama_server
ollama create phi3-financial -f Modelfile
```
Cette commande télécharge le modèle de base `phi3.5` et applique le prompt système défini dans `Modelfile` (spécialisation "assistant financier pour analystes TechCorp").

### Lancement et test manuel
```powershell
ollama run phi3-financial
```
Tests effectués avec des questions financières types (ex. "Explique-moi le compound interest") — le modèle répond de façon cohérente, avec quelques imperfections de formatage (formules mal rendues) à noter comme axe d'amélioration pour l'équipe IA.

### Vérification de l'API
```powershell
curl.exe http://localhost:11434/api/tags
```
Confirme que le modèle `phi3-financial` est bien chargé et accessible.

---

## 3. Accessibilité pour DEV WEB

Le serveur écoute par défaut sur `http://localhost:11434`, conforme à ce qu'attend l'interface web (`rendu/devweb/index.html`), qui communique avec l'endpoint `/api/chat`.

**Point d'attention CORS** : si l'interface est servie depuis un fichier local ou un port différent, Ollama peut bloquer la requête cross-origin. Solution testée : servir l'interface via un mini-serveur HTTP local (`python -m http.server`) plutôt que de l'ouvrir directement en `file://`.

---

## 4. Limites de ce déploiement

- Déploiement en local uniquement (pas d'exposition réseau externe), suffisant pour la démonstration du hackathon mais **non représentatif d'un déploiement production réel**
- Aucune authentification ni limitation de débit sur l'API — à ajouter avant tout déploiement réel (voir rapport d'audit CYBER pour le détail des risques de sécurité identifiés)
- Paramètres d'inférence (`temperature`, `top_p`, `num_predict`) non configurés dans le `Modelfile` hérité — valeurs par défaut d'Ollama utilisées faute de temps pour les optimiser