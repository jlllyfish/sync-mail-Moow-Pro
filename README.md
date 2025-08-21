# sync-mail-aide-dger
# 🚀 Automatisation Envoi Messages DS avec GitHub Actions

Ce guide détaille comment automatiser l'envoi de messages par lot vers Démarches Simplifiées en utilisant GitHub Actions.

## 📋 Table des matières

- [🎯 Vue d'ensemble](#-vue-densemble)
- [📁 Structure du projet](#-structure-du-projet)
- [🔧 Étapes d'installation](#-étapes-dinstallation)
- [🔐 Configuration des secrets GitHub](#-configuration-des-secrets-github)
- [🚀 Utilisation](#-utilisation)
- [📊 Monitoring et logs](#-monitoring-et-logs)
- [🛠️ Dépannage](#️-dépannage)

## 🎯 Vue d'ensemble

L'automatisation permet :
- ✅ **Envoi automatique** quotidien de messages à 9h00 UTC
- ✅ **Déclenchement manuel** via l'interface GitHub
- ✅ **Mode test** pour vérifier sans envoyer
- ✅ **Mode force** pour renvoyer les messages déjà envoyés
- ✅ **Gestion des erreurs** et logs détaillés
- ✅ **Sécurité** : tokens séparés du code source

## 📁 Structure du projet

```
votre-repository/
├── .github/
│   └── workflows/
│       └── send-messages.yml          # 🔄 Workflow GitHub Actions
├── config/
│   └── public-config.json            # ⚙️ Configuration publique (exportée)
├── scripts/
│   └── send_batch.py                 # 🐍 Script Python d'envoi
├── logs/                             # 📝 Logs générés automatiquement
│   ├── github-action.log
│   └── results-YYYYMMDD-HHMMSS.json
├── README.md                         # 📖 Ce fichier
└── requirements.txt                  # 📦 Dépendances (optionnel)
```

## 🔧 Étapes d'installation

### Étape 1 : Export de la configuration

1. **Lancez** votre application Flask
2. **Allez** sur la page de configuration
3. **Cliquez** sur "Exporter la configuration"
4. **Téléchargez** les fichiers :
   - `public-config.json`
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
name: 🚀 Envoi automatique de messages DS

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
    
    - name: 🚀 Send batch messages
      env:
        DS_API_TOKEN: ${{ secrets.DS_API_TOKEN }}
        GRIST_API_TOKEN: ${{ secrets.GRIST_API_TOKEN }}
        DRY_RUN: ${{ github.event.inputs.dry_run || 'false' }}
        FORCE_SEND: ${{ github.event.inputs.force_send || 'false' }}
      run: |
        echo "🚀 Démarrage de l'envoi par lot..."
        
        if [ "$DRY_RUN" = "true" ]; then
          echo "🧪 MODE TEST ACTIVÉ"
        fi
        
        if [ "$FORCE_SEND" = "true" ]; then
          echo "🔄 MODE FORCE ACTIVÉ"
        fi
        
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
    
    - name: 📈 Summary
      if: always()
      run: |
        echo "## 📊 Résumé de l'exécution" >> $GITHUB_STEP_SUMMARY
        echo "- **Date:** $(date '+%d/%m/%Y %H:%M:%S')" >> $GITHUB_STEP_SUMMARY
        echo "- **Mode:** ${{ github.event.inputs.dry_run == 'true' && '🧪 Test' || '🚀 Production' }}" >> $GITHUB_STEP_SUMMARY
        
        if [ -f "logs/github-action.log" ]; then
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 📝 Dernières lignes du log:" >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
          tail -10 logs/github-action.log >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
        fi
```

### Étape 4 : Créer le script Python

Créez le fichier `scripts/send_batch.py` avec le code fourni précédemment.

### Étape 5 : Pousser vers GitHub

```bash
git add .
git commit -m "🚀 Ajout automatisation GitHub Actions"
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
- **Aucune intervention** requise

### Exécution manuelle

1. **Allez** dans l'onglet **Actions** de votre repository
2. **Sélectionnez** le workflow "🚀 Envoi automatique de messages DS"
3. **Cliquez** sur **Run workflow**
4. **Choisissez** les options :
   - **🧪 Mode test** : Cochez pour tester sans envoyer
   - **🔄 Mode force** : Cochez pour renvoyer les messages déjà envoyés
5. **Cliquez** sur **Run workflow**

### Options d'exécution

| Option | Description | Usage |
|--------|-------------|-------|
| **Normal** | Envoi standard | Messages non envoyés uniquement |
| **🧪 Test** | Mode dry-run | Teste sans envoyer réellement |
| **🔄 Force** | Renvoie tout | Même les messages déjà envoyés |

## 📊 Monitoring et logs

### Visualisation des résultats

1. **Onglet Actions** → Sélectionner l'exécution
2. **Summary** : Résumé rapide avec statistiques
3. **Jobs** → **send-messages** : Logs détaillés
4. **Artifacts** : Télécharger les logs complets

### Types de logs générés

```
logs/
├── github-action.log              # Log principal détaillé
└── results-20240101-120000.json   # Résultats avec statistiques
```

### Format des résultats JSON

```json
{
  "total_records": 150,
  "success_count": 145,
  "error_count": 5,
  "details": [
    {
      "dossier_id": "12345",
      "error": "Dossier non trouvé"
    }
  ]
}
```

## 🛠️ Dépannage

### Erreurs courantes

#### ❌ "Fichier config/public-config.json manquant"
- **Solution** : Exportez et placez la configuration
- **Vérification** : Le fichier doit être dans `config/public-config.json`

#### ❌ "Secret DS_API_TOKEN manquant"
- **Solution** : Configurez les secrets GitHub
- **Vérification** : Settings → Secrets → Actions

#### ❌ "Erreur de connexion Grist"
- **Cause** : Token Grist invalide ou expiré
- **Solution** : Régénérer et mettre à jour le secret

#### ❌ "Instructeur non trouvé"
- **Cause** : ID instructeur incorrect dans la config
- **Solution** : Re-exporter la configuration avec le bon instructeur

### Vérification de la configuration

```bash
# Vérifier la structure des fichiers
ls -la .github/workflows/
ls -la config/
ls -la scripts/

# Vérifier le contenu de la config
cat config/public-config.json | python -m json.tool
```

### Test en local (optionnel)

```bash
# Variables d'environnement
export DS_API_TOKEN="votre_token"
export GRIST_API_TOKEN="votre_token"
export DRY_RUN="true"

# Test du script
python scripts/send_batch.py
```

### Support et contact

- **Issues GitHub** : Créer une issue sur le repository
- **Logs détaillés** : Consulter les artifacts des exécutions
- **Mode debug** : Utiliser le mode test pour diagnostiquer

---

## 📋 Checklist finale

- [ ] ✅ Configuration exportée depuis l'app Flask
- [ ] ✅ Fichier `config/public-config.json` en place
- [ ] ✅ Workflow `.github/workflows/send-messages.yml` créé
- [ ] ✅ Script `scripts/send_batch.py` en place
- [ ] ✅ Secrets GitHub configurés (`DS_API_TOKEN`, `GRIST_API_TOKEN`)
- [ ] ✅ Repository poussé sur GitHub
- [ ] ✅ Test manuel réussi en mode dry-run
- [ ] ✅ Première exécution automatique programmée

🎉 **Votre automatisation est prête !** 🎉
