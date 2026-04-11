# 🤖 Roborock Blueprints pour Home Assistant

Blueprints Home Assistant pour automatiser un aspirateur **Roborock** (testé avec le QV 35A) en combinaison avec **Alarmo** pour la détection de présence.

## ✨ Fonctionnalités

- **Aspiration automatique en absence** — quand tout le monde quitte la maison
- **Arrêt automatique au retour** — l'aspirateur rentre à la base
- **Nettoyage complet planifié** — aspiration + lavage si pas de passage depuis X jours
- **Rappels intelligents** — notification avec bouton pour lancer le nettoyage
- **Surveillance** — alertes en cas d'erreur / aspirateur bloqué
- **Tracker automatique** — enregistre la date du dernier nettoyage

## 📦 Blueprints inclus

### 1. Aspirer en absence

Lance l'aspirateur dans les pièces choisies quand l'alarme passe en mode absent. Arrête automatiquement si quelqu'un rentre.

<!-- Remplace TON_USER/TON_REPO par ton URL GitHub réelle -->
[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FTON_USER%2Froborock-ha-blueprints%2Fblob%2Fmain%2Fvacuum_absence%2Fvacuum_absence.yaml)

### 2. Nettoyage complet après X jours

Vérifie quotidiennement si le dernier nettoyage remonte à plus de X jours. Si oui et que personne n'est là, lance un nettoyage complet (avec lavage optionnel).

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FTON_USER%2Froborock-ha-blueprints%2Fblob%2Fmain%2Fvacuum_deep_clean%2Fvacuum_deep_clean.yaml)

### 3. Surveillance, rappels & tracker

Alerte en cas d'erreur, rappel si pas de passage depuis X jours (avec bouton actionnable), et mise à jour automatique du tracker.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FTON_USER%2Froborock-ha-blueprints%2Fblob%2Fmain%2Fvacuum_monitoring%2Fvacuum_monitoring.yaml)

## 🚀 Installation

### Méthode 1 : Import direct (recommandé)

1. Clique sur un des boutons **"Import Blueprint"** ci-dessus
2. Home Assistant s'ouvre et te propose d'importer le blueprint
3. Configure l'automatisation via l'UI

> ⚠️ **Avant d'importer**, remplace `TON_USER/roborock-ha-blueprints` dans les URLs par ton vrai chemin GitHub.

### Méthode 2 : Installation manuelle

```bash
cd config/blueprints/automation/
git clone https://github.com/TON_USER/roborock-ha-blueprints.git roborock
```

Puis dans Home Assistant :
**Outils de développement** → **YAML** → **Recharger les blueprints**

## ⚙️ Prérequis

### Intégration Roborock
Ton aspirateur doit être intégré via l'[intégration Roborock native](https://www.home-assistant.io/integrations/roborock/).

### Alarmo
Un panneau d'alarme [Alarmo](https://github.com/nielsfaber/alarmo) configuré pour la détection de présence.

### Helper input_datetime

Les blueprints 2 et 3 partagent un helper pour tracker le dernier nettoyage :

1. **Paramètres** → **Appareils et services** → **Entrées**
2. **Créer une entrée** → **Date et/ou heure**
3. Nom : `Dernier passage aspirateur`
4. Cocher **Date** ET **Heure**

### Room IDs

Pour trouver les IDs de tes pièces :

1. **Outils de développement** → **Actions**
2. Action : `roborock.get_maps`
3. Sélectionne ton aspirateur
4. Note les numéros de chaque pièce

Exemple de résultat :
```yaml
rooms:
  "1": Salle à manger
  "5": Couloir
  "7": Entrée
  "8": Cuisine
  "9": Salon
```

Tu saisiras ces IDs séparés par des virgules dans les blueprints : `9,8,5,7,1`

## ⚠️ Notes importantes

### Compatibilité Q-Series

Les Roborock Q-Series (dont le QV 35A) utilisent un protocole plus récent.
**Teste d'abord manuellement** dans les Outils de développement :

```yaml
action: vacuum.send_command
target:
  entity_id: vacuum.roborock_qv_35a
data:
  command: app_segment_clean
  params:
    - segments:
        - 9
      repeat: 1
```

Si ça ne fonctionne pas, vérifie que tu es sur la dernière version de Home Assistant.

### Mode lavage

Le blueprint "Deep Clean" supporte le mode lavage. Vérifie dans tes entités si tu as un `select.*_mop_mode` et quelles options sont disponibles (`standard`, `deep`, etc.).

### Notifications

Par défaut, les blueprints utilisent `notify.notify`. Pour cibler un téléphone spécifique, utilise `notify.mobile_app_ton_telephone`.

## 📝 Licence

MIT — Utilise, modifie et partage librement.
