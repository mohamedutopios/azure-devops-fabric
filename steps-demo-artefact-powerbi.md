### **📌 Prérequis pour Déployer un Pipeline CI/CD Microsoft Fabric + Power BI avec Azure DevOps**
Avant de pouvoir mettre en place ce pipeline automatisé, voici **les prérequis essentiels** à respecter.

---

## **1️⃣ Configuration des Outils et Services**
| **Outil/Service** | **Description** | **Lien / Documentation** |
|-----------------|-----------------|--------------------|
| **Microsoft Fabric** | Service d’analyse unifié pour l’ingestion et la transformation des données. | [Microsoft Fabric](https://fabric.microsoft.com/) |
| **Azure DevOps** | Outil CI/CD pour automatiser le déploiement et gérer le code source. | [Azure DevOps](https://azure.microsoft.com/fr-fr/products/devops) |
| **Azure CLI** | Interface en ligne de commande pour exécuter des commandes Azure. | [Azure CLI](https://docs.microsoft.com/fr-fr/cli/azure/install-azure-cli) |
| **Power BI** | Outil de BI pour la visualisation et l’analyse des données. | [Power BI](https://powerbi.microsoft.com/) |
| **Azure Blob Storage (optionnel)** | Stockage des fichiers sources pour l’ingestion des données. | [Azure Storage](https://azure.microsoft.com/fr-fr/products/storage/) |

---

## **2️⃣ Préparation de Microsoft Fabric**
### **✅ 1. Créer un Workspace Fabric**
1. **Se connecter à Microsoft Fabric** via le **portail Power BI** ([https://app.powerbi.com](https://app.powerbi.com)).
2. **Créer un nouvel espace de travail** :
   - Aller dans **Espaces de travail > Nouveau**.
   - Nommer le workspace (ex: `SalesWorkspace`).
   - Activer **OneLake** pour le stockage des données.

### **✅ 2. Créer un Lakehouse Fabric**
1. Aller dans **Microsoft Fabric > Data Engineering**.
2. **Créer un Lakehouse** (`SalesLakehouse`).
3. Définir les tables (`sales`, `customers`) dans le **Data Warehouse** Fabric.

### **✅ 3. Activer l’option Git dans Fabric**
1. Aller dans **Settings > Git Integration**.
2. Connecter **Azure Repos** pour versionner les artefacts Fabric.

---

## **3️⃣ Préparation d’Azure DevOps**
### **✅ 4. Créer un Projet Azure DevOps**
1. Aller sur **Azure DevOps** ([https://dev.azure.com](https://dev.azure.com)).
2. Créer un **nouveau projet** (`Fabric_CICD`).
3. Activer **Azure Repos** pour stocker les fichiers.

### **✅ 5. Créer un Repository Git pour Fabric**
1. **Cloner le repo localement** :
   ```sh
   git clone https://dev.azure.com/{organization}/{project}/_git/{repo}
   ```
2. **Organiser les artefacts Fabric** :
   ```
   📂 Fabric-Project
   ├── 📂 Lakehouses/
   │   ├── sales_lakehouse.json
   ├── 📂 Notebooks/
   │   ├── sales_transformation.ipynb
   ├── 📂 Pipelines/
   │   ├── ingestion_pipeline.json
   ├── 📂 Dataflows/
   │   ├── transformation_dataflow.json
   ├── 📂 PowerBI/
   │   ├── sales_dashboard.pbix
   ├── azure-pipelines.yml
   ```

3. **Pousser le code dans Azure Repos** :
   ```sh
   git add .
   git commit -m "Ajout des artefacts Fabric"
   git push origin main
   ```

---

## **4️⃣ Configuration de l’Authentification Azure**
### **✅ 6. Créer un Service Principal Azure pour Fabric**
Un **Service Principal (SP)** permet d’exécuter des commandes Fabric via Azure CLI.

1. **Créer un Service Principal** :
   ```sh
   az ad sp create-for-rbac --name "Fabric-SP" --role Contributor
   ```
2. **Notez les valeurs suivantes** :
   - `clientId` → Identifiant de l'application.
   - `clientSecret` → Clé secrète d’authentification.
   - `tenantId` → Identifiant du locataire Azure.

3. **Stocker ces valeurs dans Azure DevOps** :
   - Aller dans **Project Settings > Pipelines > Library**.
   - Créer des **variables secrètes** :
     - `FABRIC_CLIENT_ID`
     - `FABRIC_CLIENT_SECRET`
     - `FABRIC_TENANT_ID`

---

## **5️⃣ Préparation de Power BI**
### **✅ 7. Créer un Rapport Power BI connecté à Fabric**
1. **Ouvrir Power BI Desktop**.
2. **Se connecter aux données Fabric** :
   - Source : `Lakehouse (Microsoft Fabric)`.
   - Tables : `sales`, `customers`.
3. **Créer un Dashboard interactif**.
4. **Publier dans Power BI Service** (`Sales Dashboard`).

### **✅ 8. Récupérer l’ID du Workspace et du Dataset**
1. Aller dans **Power BI Service**.
2. Naviguer vers **Datasets**.
3. Récupérer l’ID du **workspace** et du **dataset** (nécessaire pour le pipeline DevOps).

---

## **6️⃣ Automatisation avec Azure DevOps**
### **✅ 9. Créer un Pipeline CI/CD dans Azure DevOps**
1. Aller dans **Pipelines > New Pipeline**.
2. Choisir **Azure Repos Git** et sélectionner le repository.
3. Ajouter le fichier `azure-pipelines.yml`.
4. Exécuter le pipeline et vérifier les logs.

---

## **7️⃣ Tests et Validation**
### **✅ 10. Tester le Déploiement**
1. Modifier un **notebook ou pipeline** dans Git.
2. Pousser le commit dans `main`.
3. Vérifier que :
   - 📂 **Le Lakehouse est mis à jour** dans Fabric.
   - 📂 **Les notebooks sont exécutés automatiquement**.
   - 📂 **Les transformations sont appliquées via Dataflows**.
   - 📂 **Le dataset Power BI est rafraîchi** avec les nouvelles données.
   - 📊 **Le tableau de bord Power BI affiche les nouvelles valeurs**.

---

## **🎯 Récapitulatif des Prérequis**
| ✅ **Étape** | **Détails** |
|-------------|------------|
| **1. Microsoft Fabric** | Création d’un **Workspace + Lakehouse + Pipeline + Notebook**. |
| **2. Azure DevOps** | Création d’un **repo Git, pipeline CI/CD et service principal**. |
| **3. Power BI** | Création d’un **rapport connecté à Fabric** et configuration des rafraîchissements. |
| **4. Sécurité** | Authentification via **Azure Service Principal** pour exécuter les commandes CLI. |
| **5. Tests & Automatisation** | Vérifier l’exécution automatique des artefacts Fabric + Power BI. |

---

## **🚀 Tu veux une démo vidéo complète sur l’installation et l’exécution ?**
Je peux capturer toutes les étapes et **montrer un déploiement en temps réel** ! 🚀😊