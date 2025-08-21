# sync-mail-aide-dger
# 🚀 Automatisation Envoi Messages DS avec GitHub Actions

Ce guide détaille comment automatiser l'envoi de messages par lot vers Démarches Simplifiées en utilisant GitHub Actions, avec support avancé des formats de dates et timestamps Unix.

## 📋 Table des matières

- [🎯 Vue d'ensemble](#-vue-densemble)
- [✨ Nouveautés - Gestion des dates](#-nouveautés---gestion-des-dates)
- [📁 Structure du projet](#-structure-du-projet)
- [🔧 Étapes d'installation](#-étapes-dinstallation)
- [🔐 Configuration des secrets GitHub](#-configuration-des-secrets-github)
- [🚀 Utilisation](#-utilisation)
- [📊 Monitoring et logs](#-monitoring-et-logs)
- [📅 Gestion avancée des dates](#-gestion-avancée-des-dates)
- [🛠️ Dépannage](#️-dépannage)

## 🎯 Vue d'ensemble

L'automatisation permet :
- ✅ **Envoi automatique** quotidien de messages à 9h00 UTC
- ✅ **Déclenchement manuel** via l'interface GitHub
- ✅ **Mode test** pour vérifier sans envoyer
- ✅ **Mode force** pour renvoyer les messages déjà envoyés
- ✅ **Gestion intelligente des dates** avec support timestamps Unix
- ✅ **Filtres avancés** incluant les filtres de date
- ✅ **Formatage automatique** des dates au format français
- ✅ **Gestion des erreurs** et logs détaillés
- ✅ **Sécurité** : tokens séparés du code source

## ✨ Nouveautés - Gestion des dates

### 🔧 **Support des timestamps Unix**
- **Timestamps en secondes** (10 chiffres) : `1703548800` → `25/12/2023`
- **Timestamps en millisecondes** (13 chiffres) : `1703548800000` → `25/12/2023`
- **Conversion automatique** dans les messages personnalisés
- **Filtres de date** fonctionnels sur les timestamps Grist

### 📅 **Formats de dates supportés**
| Format d'entrée | Exemple | Sortie française |
|----------------|---------|------------------|
| Timestamp Unix secondes | `1703548800` | `25/12/2023` |
| Timestamp Unix millisecondes | `1703548800000` | `25/12/2023` |
| Date ISO | `2023-12-25` | `25/12/2023` |
| Date française | `25/12/2023` | `25/12/2023` |
| DateTime ISO | `2023-12-25T14:30:00Z` | `25/12/2023` |

### 🎯 **Filtres de date avancés**
- **Avant le** : `date_before`
- **Après le** : `date_after`
- **Le (date exacte)** : `date_on`
- **Entre deux dates** : `date_between`
- **Support automatique** des timestamps Unix dans les filtres

## 📁 Structure du projet

```
votre-repository/
├── .github/
│   └── workflows/
│       └── send-messages.yml          # 🔄 Workflow GitHub Actions
├── config/
│   └── public-config.json            # ⚙️ Configuration publique (exportée)
├── scripts/
│   └── send_batch.py                 # 🐍 Script Python d'envoi avec gestion dates
├── logs/                             # 📝 Logs générés automatiquement
│   ├── github-action.log
│   └── results-YYYYMMDD-HHMMSS.json
├── README.md                         # 📖 Ce fichier
└── requirements.txt                  # 📦 Dépendances (optionnel)
```

## 🔧 Étapes d'installation

### Étape 1 : Export de la configuration

1. **Lancez** votre application Flask avec les améliorations de dates
2. **Configurez** vos filtres de dates dans `/filters`
3. **Allez** sur la page de configuration
4. **Cliquez** sur "Exporter la configuration"
5. **Téléchargez** les fichiers :
   - `public-config.json` (inclut maintenant les filtres de dates)
   - `INSTRUCTIONS-GITHUB-ACTIONS.txt`

### Étape 2 : Créer la structure GitHub

1. **Créez** un nouveau repository GitHub (ou utilisez un existant)

2. **Créez** la structure de dossiers :
   ```bash
   mkdir -p .github/workflows
   mkdir -p config
   mkdir -p scripts
   ```

3. **Placez** le fichier `public-config.json` dans le dossier `config/`

### Étape 3 : Créer le workflow GitHub Actions

Créez le fichier `.github/workflows/send-messages.yml` :

```yaml
name: 🚀 Envoi automatique de messages DS avec gestion des dates

on:
  schedule:
    # Tous les jours à 9h00 UTC (10h00 Paris hiver, 11h00 Paris été)
    - cron: '0 9 * * *'
  workflow_dispatch:
    # Permet le déclenchement manuel
    inputs:
      dry_run:
        description: '🧪 Mode test (ne pas envoyer les messages)'
        required: false
        default: 'false'
        type: boolean
      force_send:
        description: '🔄 Forcer l\'envoi même si déjà envoyés'
        required: false
        default: 'false'
        type: boolean

jobs:
  send-messages:
    runs-on: ubuntu-latest
    
    steps:
    - name: 📥 Checkout repository
      uses: actions/checkout@v4
    
    - name: 🐍 Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
        cache: 'pip'
    
    - name: 📦 Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install requests python-dotenv
    
    - name: 🔧 Verify configuration files
      run: |
        echo "🔍 Vérification des fichiers..."
        
        if [ ! -f "config/public-config.json" ]; then
          echo "❌ Fichier config/public-config.json manquant"
          exit 1
        fi
        
        if [ ! -f "scripts/send_batch.py" ]; then
          echo "❌ Fichier scripts/send_batch.py manquant"
          exit 1
        fi
        
        echo "✅ Fichiers trouvés"
        
        # Vérifier la présence de filtres de dates
        if grep -q "date_" config/public-config.json; then
          echo "📅 Filtres de dates détectés dans la configuration"
        fi
    
    - name: 🔐 Verify secrets
      env:
        DS_API_TOKEN: ${{ secrets.DS_API_TOKEN }}
        GRIST_API_TOKEN: ${{ secrets.GRIST_API_TOKEN }}
      run: |
        echo "🔍 Vérification des secrets..."
        
        if [ -z "$DS_API_TOKEN" ]; then
          echo "❌ Secret DS_API_TOKEN manquant"
          exit 1
        fi
        
        if [ -z "$GRIST_API_TOKEN" ]; then
          echo "❌ Secret GRIST_API_TOKEN manquant"
          exit 1
        fi
        
        echo "✅ Secrets trouvés"
    
    - name: 📁 Create logs directory
      run: mkdir -p logs
    
    - name: 🚀 Send batch messages with date processing
      env:
        DS_API_TOKEN: ${{ secrets.DS_API_TOKEN }}
        GRIST_API_TOKEN: ${{ secrets.GRIST_API_TOKEN }}
        DRY_RUN: ${{ github.event.inputs.dry_run || 'false' }}
        FORCE_SEND: ${{ github.event.inputs.force_send || 'false' }}
      run: |
        echo "🚀 Démarrage de l'envoi par lot avec gestion des dates..."
        
        if [ "$DRY_RUN" = "true" ]; then
          echo "🧪 MODE TEST ACTIVÉ - Aucun message ne sera envoyé"
        fi
        
        if [ "$FORCE_SEND" = "true" ]; then
          echo "🔄 MODE FORCE ACTIVÉ - Renvoi des messages déjà envoyés"
        fi
        
        echo "📅 Support des timestamps Unix et formats de dates activé"
        
        python scripts/send_batch.py
    
    - name: 📊 Upload results
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: send-results-${{ github.run_number }}
        path: |
          logs/
          *.log
        retention-days: 30
    
    - name: 📈 Enhanced summary with date processing info
      if: always()
      run: |
        echo "## 📊 Résumé de l'exécution avec gestion des dates" >> $GITHUB_STEP_SUMMARY
        echo "- **Date:** $(date '+%d/%m/%Y %H:%M:%S')" >> $GITHUB_STEP_SUMMARY
        echo "- **Mode:** ${{ github.event.inputs.dry_run == 'true' && '🧪 Test' || '🚀 Production' }}" >> $GITHUB_STEP_SUMMARY
        echo "- **Gestion des dates:** ✅ Timestamps Unix supportés" >> $GITHUB_STEP_SUMMARY
        echo "- **Filtres de dates:** ✅ Actifs si configurés" >> $GITHUB_STEP_SUMMARY
        
        if [ -f "logs/github-action.log" ]; then
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 📝 Dernières lignes du log:" >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
          tail -10 logs/github-action.log >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
        fi
        
        # Rechercher des mentions de dates dans les logs
        if [ -f "logs/github-action.log" ] && grep -q "📅" logs/github-action.log; then
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 📅 Traitement des dates détecté:" >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
          grep "📅" logs/github-action.log | tail -5 >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
        fi
```

### Étape 4 : Créer le script Python

Créez le fichier `scripts/send_batch.py` avec le code fourni dans l'artefact précédent.

### Étape 5 : Pousser vers GitHub

```bash
git add .
git commit -m "🚀 Ajout automatisation GitHub Actions avec gestion des dates"
git push origin main
```

## 🔐 Configuration des secrets GitHub

1. **Allez** dans votre repository GitHub
2. **Cliquez** sur **Settings** → **Secrets and variables** → **Actions**
3. **Cliquez** sur **New repository secret**
4. **Ajoutez** ces secrets :

| Nom | Description | Valeur |
|-----|-------------|--------|
| `DS_API_TOKEN` | Token Démarches Simplifiées | Votre token DS |
| `GRIST_API_TOKEN` | Token API Grist | Votre token Grist |

### ⚠️ Important
- **Ne jamais** committer les tokens dans le code
- **Vérifier** que les secrets sont bien configurés
- **Tester** d'abord en mode dry-run

## 🚀 Utilisation

### Exécution automatique
- **Planifiée** : Tous les jours à 9h00 UTC automatiquement
- **Traitement intelligent** des dates et timestamps
- **Application automatique** des filtres configurés
- **Aucune intervention** requise

### Exécution manuelle

1. **Allez** dans l'onglet **Actions** de votre repository
2. **Sélectionnez** le workflow "🚀 Envoi automatique de messages DS avec gestion des dates"
3. **Cliquez** sur **Run workflow**
4. **Choisissez** les options :
   - **🧪 Mode test** : Cochez pour tester sans envoyer
   - **🔄 Mode force** : Cochez pour renvoyer les messages déjà envoyés
5. **Cliquez** sur **Run workflow**

### Options d'exécution

| Option | Description | Usage | Gestion des dates |
|--------|-------------|-------|-------------------|
| **Normal** | Envoi standard | Messages non envoyés uniquement | ✅ Conversion automatique |
| **🧪 Test** | Mode dry-run | Teste sans envoyer réellement | ✅ Affichage des conversions |
| **🔄 Force** | Renvoie tout | Même les messages déjà envoyés | ✅ Re-traitement des dates |

## 📊 Monitoring et logs

### Visualisation des résultats

1. **Onglet Actions** → Sélectionner l'exécution
2. **Summary** : Résumé rapide avec statistiques et info dates
3. **Jobs** → **send-messages** : Logs détaillés avec traitement des dates
4. **Artifacts** : Télécharger les logs complets

### Types de logs générés

```
logs/
├── github-action.log              # Log principal avec debug des dates
└── results-20240101-120000.json   # Résultats avec statistiques
```

### Format des résultats JSON amélioré

```json
{
  "total_records": 150,
  "success_count": 145,
  "error_count": 5,
  "filters_applied": true,
  "date_conversions_detected": 45,
  "details": [
    {
      "dossier_id": "12345",
      "error": "Dossier non trouvé",
      "dates_processed": {
        "created_date": "1703548800 → 25/12/2023",
        "sync_date": "1703635200 → 26/12/2023"
      }
    }
  ]
}
```

## 📅 Gestion avancée des dates

### 🔧 **Configuration des filtres de dates**

Dans votre application Flask, configurez les filtres dans `/filters` :

```javascript
// Exemples de filtres supportés
{
  "column": "created_date",
  "operator": "date_after", 
  "value": "2023-12-01"
}

{
  "column": "deadline", 
  "operator": "date_between",
  "value": "2023-12-01|2023-12-31"
}
```

### 📊 **Détection automatique des colonnes**

Le script détecte automatiquement les colonnes contenant des dates basé sur :
- **Noms de colonnes** : `date`, `created`, `updated`, `time`, `deadline`, `sync_date`
- **Contenu des champs** : Timestamps Unix, dates ISO, etc.

### 🎯 **Variables dans les messages**

Les timestamps sont automatiquement convertis dans les messages :

```
Message template : "Votre dossier créé le {created_date} expire le {deadline}"
Avant : "Votre dossier créé le 1703548800 expire le 1706140800"
Après : "Votre dossier créé le 25/12/2023 expire le 25/01/2024"
```

### ⚙️ **Configuration avancée**

```json
{
  "filters": [
    {
      "column": "created_date",
      "operator": "date_after",
      "value": "2023-01-01"
    },
    {
      "column": "status",
      "operator": "equals", 
      "value": "pending"
    }
  ],
  "filter_logic": "AND",
  "message_subject": "Rappel pour {user_name} - dossier du {created_date}",
  "message_body": "Votre dossier créé le {created_date} nécessite une action avant le {deadline}."
}
```

## 🛠️ Dépannage

### Erreurs courantes

#### ❌ "Impossible de parser la date: 1703548800"
- **Cause** : Format de timestamp non reconnu
- **Solution** : Vérifiez que le timestamp est en secondes ou millisecondes
- **Debug** : Logs montrent les tentatives de parsing

#### ❌ "Filtre de date ne fonctionne pas"
- **Cause** : Mauvais format de date dans le filtre
- **Solution** : Utilisez le format YYYY-MM-DD pour les filtres
- **Vérification** : Testez d'abord dans l'interface Flask

#### ❌ "Dates mal formatées dans les messages"
- **Cause** : Variable non reconnue comme date
- **Solution** : Vérifiez le nom de la colonne dans les mots-clés de détection
- **Debug** : Cherchez "📅 Dates détectées" dans les logs

### Diagnostic des dates

```bash
# Vérifier les formats de dates dans la config
cat config/public-config.json | grep -E "(date|time|created|sync)"

# Rechercher les conversions de dates dans les logs
grep "📅" logs/github-action.log

# Vérifier les filtres de dates
grep "date_" config/public-config.json
```

### Test en local des dates

```bash
# Variables d'environnement
export DS_API_TOKEN="votre_token"
export GRIST_API_TOKEN="votre_token"
export DRY_RUN="true"

# Test avec debug des dates
python -c "
from scripts.send_batch import parse_date_value, format_date_french
print('Test timestamp:', format_date_french(1703548800))
print('Test ISO:', format_date_french('2023-12-25'))
"

# Test du script complet
python scripts/send_batch.py
```

### 🔍 **Debugging avancé**

1. **Activer les logs de debug** :
   ```python
   logging.getLogger().setLevel(logging.DEBUG)
   ```

2. **Vérifier la détection des dates** :
   ```bash
   grep "Dates détectées" logs/github-action.log
   ```

3. **Analyser les conversions** :
   ```bash
   grep "→" logs/github-action.log | head -10
   ```

### Support et contact

- **Issues GitHub** : Créer une issue sur le repository
- **Logs détaillés** : Consulter les artifacts des exécutions avec focus sur les dates
- **Mode debug** : Utiliser le mode test pour diagnostiquer les conversions

---

## 📋 Checklist finale avec gestion des dates

- [ ] ✅ Configuration exportée depuis l'app Flask avec filtres de dates
- [ ] ✅ Fichier `config/public-config.json` contient les filtres de dates
- [ ] ✅ Workflow `.github/workflows/send-messages.yml` mis à jour
- [ ] ✅ Script `scripts/send_batch.py` avec gestion des dates en place
- [ ] ✅ Secrets GitHub configurés (`DS_API_TOKEN`, `GRIST_API_TOKEN`)
- [ ] ✅ Repository poussé sur GitHub
- [ ] ✅ Test manuel réussi en mode dry-run avec timestamps
- [ ] ✅ Vérification des conversions de dates dans les logs
- [ ] ✅ Filtres de dates testés et fonctionnels
- [ ] ✅ Messages avec variables de dates formatées correctement
- [ ] ✅ Première exécution automatique programmée

## 🎉 **Fonctionnalités de dates disponibles**

- ✅ **Support complet des timestamps Unix** (secondes et millisecondes)
- ✅ **Conversion automatique** au format français (DD/MM/YYYY)
- ✅ **Filtres de dates avancés** (avant, après, entre, exacte)
- ✅ **Variables de dates** dans les messages personnalisés
- ✅ **Détection intelligente** des colonnes de dates
- ✅ **Logs détaillés** des conversions de dates
- ✅ **Mode debug** pour diagnostiquer les formats
- ✅ **Compatibilité totale** avec l'interface Flask

🎉 **Votre automatisation avec gestion avancée des dates est prête !** 🎉
