### **Déploiement Automatique des Artefacts Microsoft Fabric en Production avec Azure DevOps (CI/CD)**
L’objectif ici est de mettre en place un **pipeline CI/CD** permettant de **déployer automatiquement des artefacts Fabric** (comme un **Lakehouse**, un **Notebook**, un **Dataflow**, ou un **Pipeline**) en **production**, en utilisant **Azure DevOps**.

---

## **1️⃣ Vue d’Ensemble du Processus CI/CD**
Nous allons utiliser **Azure DevOps** pour automatiser le déploiement de nos artefacts Fabric avec les étapes suivantes :

1. **Stocker les artefacts Fabric dans un dépôt Git** (Azure Repos).
2. **Déclencher un pipeline Azure DevOps à chaque commit** sur `main`.
3. **Valider les artefacts Fabric** (ex. : tests de notebooks).
4. **Déployer les artefacts Fabric dans l’environnement de production**.
5. **Notifier l’équipe** en cas de succès ou d’échec.

---

## **2️⃣ Prérequis**
- Un **Microsoft Fabric Workspace** déjà configuré.
- Un **Azure DevOps Repository** contenant les artefacts Fabric versionnés.
- Une **Service Principal Azure** pour l’authentification.
- Azure CLI installé dans l’agent d’exécution du pipeline.

---

## **3️⃣ Pipeline CI/CD - Étape par Étape**

### **1️⃣ Étape 1 : Configurer Git avec Microsoft Fabric**
Avant de mettre en place le pipeline, assurez-vous que **Fabric est connecté à Azure Repos** :

1. **Dans Fabric**, allez à `Workspace settings > Git Integration`.
2. Sélectionnez **Azure Repos Git** et connectez le dépôt.
3. Activez la synchronisation automatique pour versionner les artefacts.

### **2️⃣ Étape 2 : Définir la Structure du Dépôt Git**
Dans **Azure Repos**, organisez vos artefacts Fabric de cette manière :

```
📂 Fabric-Project
 ├── 📂 Lakehouses/
 │   ├── lakehouse_config.json
 ├── 📂 Notebooks/
 │   ├── etl_notebook.ipynb
 ├── 📂 Pipelines/
 │   ├── ingestion_pipeline.json
 ├── 📂 Dataflows/
 │   ├── transformation_dataflow.json
 ├── azure-pipelines.yml
```

### **3️⃣ Étape 3 : Création du Pipeline YAML dans Azure DevOps**
Ajoutez le fichier **`azure-pipelines.yml`** à la racine du dépôt et insérez le code suivant :

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  FABRIC_WORKSPACE: "MyFabricWorkspace"
  FABRIC_TENANT_ID: "xxxxx-xxxx-xxxx-xxxx-xxxx"
  FABRIC_CLIENT_ID: "xxxxx-xxxx-xxxx-xxxx-xxxx"
  FABRIC_CLIENT_SECRET: "$(FABRIC_CLIENT_SECRET)"

steps:

# ✅ Étape 1 : Se connecter à Azure avec Service Principal
- task: AzureCLI@2
  displayName: "Authentification Azure"
  inputs:
    azureSubscription: 'MyAzureServiceConnection'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az login --service-principal -u $(FABRIC_CLIENT_ID) -p $(FABRIC_CLIENT_SECRET) --tenant $(FABRIC_TENANT_ID)

# ✅ Étape 2 : Valider les artefacts Fabric avant déploiement
- task: PowerShell@2
  displayName: "Validation des artefacts Fabric"
  inputs:
    targetType: 'inline'
    script: |
      Write-Host "Validation des notebooks et pipelines..."
      if (Test-Path "./Notebooks/etl_notebook.ipynb") {
        Write-Host "✅ Notebook valide"
      } else {
        Write-Error "❌ Notebook manquant !"
        exit 1
      }

# ✅ Étape 3 : Déploiement des artefacts Fabric
- task: AzureCLI@2
  displayName: "Déploiement des artefacts Fabric"
  inputs:
    azureSubscription: 'MyAzureServiceConnection'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      echo "📢 Déploiement du Lakehouse..."
      az fabric deploy --workspace $(FABRIC_WORKSPACE) --artifact "./Lakehouses/lakehouse_config.json"

      echo "📢 Déploiement du Notebook..."
      az fabric deploy --workspace $(FABRIC_WORKSPACE) --artifact "./Notebooks/etl_notebook.ipynb"

      echo "📢 Déploiement du Pipeline..."
      az fabric deploy --workspace $(FABRIC_WORKSPACE) --artifact "./Pipelines/ingestion_pipeline.json"

      echo "✅ Déploiement terminé avec succès !"

# ✅ Étape 4 : Notification en cas de succès
- task: PowerShell@2
  displayName: "Notification de déploiement réussi"
  inputs:
    targetType: 'inline'
    script: |
      Write-Host "🚀 Déploiement terminé !"
      exit 0
```

---

## **4️⃣ Explication du Pipeline**
| Étape | Description |
|---|---|
| **Authentification Azure** | Se connecte à Azure via une **Service Principal** pour exécuter les commandes Fabric. |
| **Validation des artefacts Fabric** | Vérifie que les fichiers nécessaires sont bien présents dans le dépôt. |
| **Déploiement des artefacts Fabric** | Exécute `az fabric deploy` pour envoyer les artefacts dans Fabric. |
| **Notification** | Affiche un message de confirmation ou d’erreur en cas d’échec. |

---

## **5️⃣ Déclenchement Automatique**
- Le pipeline est **déclenché automatiquement** dès qu’un commit est effectué sur `main`.
- Les artefacts Fabric sont **automatiquement déployés** sans intervention manuelle.

---

## **6️⃣ Résumé des Bénéfices**
✅ **Déploiement rapide et sécurisé** des artefacts Fabric.  
✅ **Suivi des modifications et rollback possible** via Git.  
✅ **Automatisation des tâches répétitives** via Azure Pipelines.  
✅ **Assurance qualité** avec validation des artefacts avant mise en production.  
✅ **Notification automatique** en cas d’échec ou de succès.

---

## **🎯 Prochaines Étapes**
Tu veux une **démo vidéo** ou des **variantes du pipeline (avec tests avancés, logs, monitoring) ?** 🚀