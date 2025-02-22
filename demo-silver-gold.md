Voici un **pipeline complet en YAML** pour **Azure DevOps**, qui orchestre l'exécution de tes notebooks dans **Microsoft Fabric**, le stockage des données dans **Lakehouse**, la publication du rapport **Power BI** et le rafraîchissement des données.

---

### **📌 Fichier YAML : `azure-pipelines.yml`**
Ce fichier contient :
1. **Exécution des Notebooks Fabric** (**Raw → Landing → Bronze → Silver → Gold**).
2. **Chargement des données dans Microsoft Fabric Lakehouse**.
3. **Publication du rapport Power BI (`LMS_import.pbix`)**.
4. **Rafraîchissement des données dans Power BI**.

---

### **💡 Pré-requis**
- 🔑 **Crée une Variable Group dans Azure DevOps** avec :
  - `FABRIC_ACCESS_TOKEN` → Token d’authentification Fabric.
  - `PBI_ACCESS_TOKEN` → Token d’authentification Power BI.
  - `FABRIC_WORKSPACE_ID` → ID du workspace Fabric.
  - `FABRIC_LAKEHOUSE_ID` → ID du Lakehouse.
  - `FABRIC_NOTEBOOK_IDS` → Liste des IDs des notebooks Fabric.
  - `PBI_DATASET_ID` → ID du dataset Power BI.
- 📌 **Stocke ton fichier PBIX (`LMS_import.pbix`)** dans ton repo Git.

---

## **🚀 Pipeline YAML**
```yaml
trigger:
  branches:
    include:
      - main  # Exécute la pipeline sur les commits sur main

schedules:
  - cron: "0 2 * * *"  # Planifie l’exécution tous les jours à 2h du matin
    displayName: "Run daily at 2 AM"
    branches:
      include:
        - main

stages:
  - stage: TransformData
    displayName: "1️⃣ Exécution des notebooks Fabric"
    jobs:
      - job: RunNotebooks
        displayName: "Exécuter les transformations des données"
        steps:
          - script: |
              NOTEBOOK_IDS=($(echo "$(FABRIC_NOTEBOOK_IDS)" | tr "," "\n"))
              for notebook in "${NOTEBOOK_IDS[@]}"; do
                curl -X POST "https://api.fabric.microsoft.com/v1/notebooks/$notebook/execute" \
                  -H "Authorization: Bearer $(FABRIC_ACCESS_TOKEN)" \
                  -H "Content-Type: application/json" \
                  -d '{ "parameters": {} }'
                echo "Notebook $notebook exécuté"
              done
            displayName: "Exécuter les Notebooks Fabric"

  - stage: StoreData
    displayName: "2️⃣ Stockage des données dans Fabric Lakehouse"
    jobs:
      - job: UploadToLakehouse
        displayName: "Charger les données dans OneLake"
        steps:
          - script: |
              curl -X POST "https://api.fabric.microsoft.com/v1/lakehouses/$(FABRIC_LAKEHOUSE_ID)/upload" \
                   -H "Authorization: Bearer $(FABRIC_ACCESS_TOKEN)" \
                   -F "file=@data/output.parquet"
            displayName: "Envoyer les données transformées"

  - stage: DeployPowerBI
    displayName: "3️⃣ Déploiement du rapport Power BI"
    jobs:
      - job: PublishPBIX
        displayName: "Publier le rapport Power BI"
        steps:
          - script: |
              curl -X POST "https://api.powerbi.com/v1.0/myorg/groups/$(FABRIC_WORKSPACE_ID)/imports?name=LMS_Report" \
                -H "Authorization: Bearer $(PBI_ACCESS_TOKEN)" \
                -F "file=@reports/LMS_import.pbix"
            displayName: "Déployer le rapport Power BI"

  - stage: RefreshData
    displayName: "4️⃣ Rafraîchissement des données Power BI"
    jobs:
      - job: RefreshDataset
        displayName: "Rafraîchir le dataset Power BI"
        steps:
          - script: |
              curl -X POST \
                -H "Authorization: Bearer $(PBI_ACCESS_TOKEN)" \
                "https://api.powerbi.com/v1.0/myorg/groups/$(FABRIC_WORKSPACE_ID)/datasets/$(PBI_DATASET_ID)/refreshes"
            displayName: "Lancer le rafraîchissement du dataset"
```

---

## **📌 Explication des étapes**
| Étape | Description |
|-------|-------------|
| **1️⃣ Transformation des données** | Exécution des notebooks Fabric (**Raw → Landing → Bronze → Silver → Gold**). |
| **2️⃣ Stockage dans Fabric Lakehouse** | Chargement des données dans **Microsoft Fabric OneLake**. |
| **3️⃣ Déploiement du rapport Power BI** | Publication automatique de `LMS_import.pbix` sur Power BI Service. |
| **4️⃣ Rafraîchissement des données** | Mise à jour des données du dataset Power BI. |

---

## **🎯 Personnalisation**
- **Notebooks Fabric** → Ajoute les IDs de tes notebooks dans `FABRIC_NOTEBOOK_IDS` (séparés par des `,`).
- **Lakehouse Fabric** → Change l’ID `FABRIC_LAKEHOUSE_ID`.
- **Workspace Power BI** → Mets l’ID de ton **workspace** dans `FABRIC_WORKSPACE_ID`.
- **Dataset Power BI** → Mets l’ID du **dataset Power BI** à rafraîchir.

---

## **🚀 Avantages de cette pipeline**
✅ **Exécute toute la chaîne Fabric (Notebooks, Lakehouse, Power BI)**.  
✅ **Automatise le rafraîchissement des données Power BI**.  
✅ **S’intègre parfaitement dans un workflow CI/CD avec Azure DevOps**.  

Dis-moi si tu veux adapter certaines parties ! 😊