# Hourly Progress Tracking Configuration

**Systeme de monitoring horaire pour Nexio - Mise à jour toutes les 1h (9h00 AM Madagascar)**

---

## Configuration

### Fréquence
- **Horaires :** Tous les jours à 9h00 AM (heure de Madagascar)
- **Rapports :** Tous les jours (Lundi - Vendredi)
- **Weekend :** Aucun rapport (les agents ne bossent pas le weekend)

### Agents à Suivre
1. **Dev UI Agent** (`nexio-dev-ui`)
2. **Dev API Agent** (`nexio-dev-api`)
3. **Dev DB Agent** (`nexio-dev-db`)
4. **Dev ERP Agent** (`nexio-dev-erp`)

### Sources de Données
- **GitHub Commits :** Vérifier les commits sur chaque repo agent
- **Session Logs :** Lire les logs OpenClaw pour détecter l'activité
- **Issues/PRs :** Vérifier les GitHub Issues/PRs ouverts

### Format de Rapport

```markdown
# 🕐 Rapport Horaire - [Date] [Heure]

## Résumé de l'Heure

### 🚀 Agents Actifs
- **Dev UI Agent:** [Status] - [Tâches principales]
- **Dev API Agent:** [Status] - [Tâches principales]
- **Dev DB Agent:** [Status] - [Tâches principales]
- **Dev ERP Agent:** [Status] - [Tâches principales]

### 📝 Nouvelles Fonctionnalités
- [ ] Nouveau endpoint implémenté
- [ ] Bug corrigé
- [ ] Test ajouté

### 🐛 Bloqueurs & Risques
- [ ] Bloqueur critique identifié
- [ ] Risque majeur enregistré

### 📊 Métriques
- **Commits :** [Nombre]
- **Tasks Completed :** [Nombre]
- **Test Coverage :** [Pourcentage]

### 🔧 Configuration
- [ ] Nouvelles variables d'environnement ajoutées

---

## Historique des Jours

Les rapports horaires sont archivés dans `docs/hourly-reports/YYYY-MM-DD-HH/`.
