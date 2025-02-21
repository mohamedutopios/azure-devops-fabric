### **Déploiement Complet de Microsoft Fabric jusqu’à Power BI avec Azure DevOps (CI/CD)**
L’objectif ici est de **déployer un pipeline analytique complet**, en passant par **Microsoft Fabric et Power BI**, en utilisant **Azure DevOps** pour automatiser le processus de bout en bout.

---

## **📌 Objectif du Projet**
💡 **Cas d’usage** : Analyse des ventes d’une entreprise.

Nous allons :
1. **Ingestion des données brutes** depuis un stockage Azure (Azure Blob Storage).
2. **Stockage et transformation** dans **Microsoft Fabric (Lakehouse + Notebook + Pipeline)**.
3. **Préparation des données** avec **Dataflows**.
4. **Création et publication d’un dashboard Power BI** connecté aux données de Fabric.
5. **Automatisation du déploiement** avec **Azure DevOps (CI/CD)**.

---

## **🛠️ 1. Architecture et artefacts**
| 📂 Dossier | Contenu |
|------------|---------|
| **Lakehouses/** | Stockage structuré des ventes et clients |
| **Notebooks/** | Nettoyage et transformation des données |
| **Pipelines/** | Chargement des données brutes |
| **Dataflows/** | Préparation des données pour Power BI |
| **PowerBI/** | Modèles et rapports Power BI |
| **azure-pipelines.yml** | Pipeline Azure DevOps pour l’automatisation |

---

## **📂 2. Contenu des fichiers**
### **1️⃣ `Lakehouses/sales_lakehouse.json`**
📌 Définit un **Lakehouse** avec **deux tables** : `sales` et `customers`.
```json
{
  "name": "SalesLakehouse",
  "description": "Stockage des ventes et des clients",
  "tables": [
    {
      "name": "sales",
      "path": "/tables/sales",
      "schema": [
        {"name": "id", "type": "integer"},
        {"name": "date", "type": "timestamp"},
        {"name": "customer_id", "type": "integer"},
        {"name": "amount", "type": "double"}
      ]
    }
  ]
}
```

---

### **2️⃣ `Notebooks/sales_transformation.ipynb`**
📌 Nettoie et transforme les ventes avec **PySpark**.
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SalesTransformation").getOrCreate()

# Lecture des données brutes depuis le Lakehouse
df = spark.read.format("delta").load("/tables/sales")

# Nettoyage des données (filtrer les ventes négatives)
df_cleaned = df.filter(df["amount"] > 0)

# Écriture des données nettoyées
df_cleaned.write.format("delta").mode("overwrite").save("/tables/cleaned_sales")
```

---

### **3️⃣ `Pipelines/ingestion_pipeline.json`**
📌 Pipeline qui charge les ventes **depuis Azure Blob Storage vers Fabric**.
```json
{
  "name": "IngestionPipeline",
  "activities": [
    {
      "name": "Load Sales Data",
      "type": "CopyActivity",
      "inputs": [
        {
          "source": {
            "type": "BlobSource",
            "container": "sales-data",
            "path": "2025/",
            "format": "parquet"
          }
        }
      ],
      "outputs": [
        {
          "sink": {
            "type": "LakehouseSink",
            "path": "/tables/sales"
          }
        }
      ]
    }
  ]
}
```

---

### **4️⃣ `PowerBI/sales_dashboard.pbix`**
📌 Rapport Power BI **connecté aux données du Lakehouse**.  
- **Graphiques** : Chiffre d'affaires total, ventes par région.  
- **Table de données** : Liste des transactions avec filtres dynamiques.  

---

## **📂 3. Automatisation avec Azure DevOps**
### **🔹 Création d’un Pipeline CI/CD**
Nous allons automatiser :
✅ Le **déploiement du Lakehouse, Notebook, Pipeline et Dataflow**.  
✅ La **mise à jour automatique du rapport Power BI** après transformation des données.  

---

### **📝 `azure-pipelines.yml` (Pipeline DevOps)**
```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  FABRIC_WORKSPACE: "SalesWorkspace"
  FABRIC_TENANT_ID: "xxxxx-xxxx-xxxx-xxxx"
  FABRIC_CLIENT_ID: "xxxxx-xxxx-xxxx-xxxx"
  FABRIC_CLIENT_SECRET: "$(FABRIC_CLIENT_SECRET)"

steps:

# ✅ Connexion Azure CLI
- task: AzureCLI@2
  displayName: "Authentification Azure"
  inputs:
    azureSubscription: 'MyAzureServiceConnection'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az login --service-principal -u $(FABRIC_CLIENT_ID) -p $(FABRIC_CLIENT_SECRET) --tenant $(FABRIC_TENANT_ID)

# ✅ Déploiement du Lakehouse
- task: AzureCLI@2
  displayName: "Déploiement du Lakehouse"
  inputs:
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az fabric deploy --workspace $(FABRIC_WORKSPACE) --artifact "./Lakehouses/sales_lakehouse.json"

# ✅ Exécution du Notebook de transformation
- task: AzureCLI@2
  displayName: "Exécution du Notebook"
  inputs:
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az fabric run-notebook --workspace $(FABRIC_WORKSPACE) --notebook "./Notebooks/sales_transformation.ipynb"

# ✅ Déploiement du Pipeline
- task: AzureCLI@2
  displayName: "Déploiement du Pipeline"
  inputs:
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az fabric deploy --workspace $(FABRIC_WORKSPACE) --artifact "./Pipelines/ingestion_pipeline.json"

# ✅ Rafraîchir le Dataset Power BI
- task: PowerShell@2
  displayName: "Rafraîchir le Dataset Power BI"
  inputs:
    targetType: 'inline'
    script: |
      $workspaceId = "xxxx-xxxx-xxxx"
      $datasetId = "yyyy-yyyy-yyyy"
      $accessToken = $(PowerBI_AccessToken)

      Invoke-RestMethod -Uri "https://api.powerbi.com/v1.0/myorg/groups/$workspaceId/datasets/$datasetId/refreshes" `
      -Headers @{ "Authorization" = "Bearer $accessToken" } `
      -Method Post
```

---

## **📊 4. Déroulement du Processus CI/CD**
1️⃣ **Développeur pousse du code** (`git push origin main`).  
2️⃣ **Le pipeline Azure DevOps s’exécute** :  
   - Déploie **le Lakehouse, le Notebook, le Pipeline**.  
   - Lance **le traitement des données**.  
3️⃣ **Les données nettoyées sont mises à jour dans Fabric**.  
4️⃣ **Power BI rafraîchit automatiquement son dataset**.  
5️⃣ **Les dashboards Power BI affichent les nouvelles données**.  

---

## **🚀 Résultats attendus**
✅ **Une chaîne analytique complète et automatisée**.  
✅ **Des données toujours à jour dans Power BI** sans action manuelle.  
✅ **Un déploiement sécurisé et versionné** via Azure DevOps.  
✅ **Une réduction des erreurs humaines et une accélération du traitement des données**.  

---

## **🎯 Prochaine Étape : Démo Vidéo**
Je vais capturer une vidéo montrant :
- **La création du pipeline dans Azure DevOps**.
- **L’exécution et le suivi des logs**.
- **La mise à jour en temps réel de Power BI**.

Tu veux que j’ajoute d’autres fonctionnalités (ex : monitoring, logs avancés) ? 🚀😊