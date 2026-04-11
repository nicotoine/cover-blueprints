# Blueprints Volets pour Home Assistant

Blueprints Home Assistant pour gerer des volets electriques sans retour de position (type Tuya/Etersky).

## Blueprints inclus

### Position estimee par le temps

Estime la position d'un volet (0-100%) en se basant sur le temps de course complet. Ideal pour les interrupteurs de volets qui ne remontent pas la position (Etersky WF-CS01, Allevoi CS01, etc.).

**Fonctionnalites :**
- Calcul automatique de la position a chaque mouvement depuis HA
- Recalibrage automatique chaque nuit via Alarmo (ferme le volet et remet a 0%)
- Bouton de synchro manuelle pour forcer un recalibrage

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fnicotoine%2Fcover-blueprints%2Fblob%2Fmaster%2Fcover_position%2Fcover_position.yaml)

## Prerequis

### Alarmo

Un panneau d'alarme [Alarmo](https://github.com/nielsfaber/alarmo) configure pour la detection de presence et le mode nuit.

### Helpers a creer (par volet)

1. **input_number** — Position estimee
   - **Parametres** > **Appareils et services** > **Entrees**
   - **Creer une entree** > **Nombre**
   - Nom : `Position volet cuisine` (adapter selon la piece)
   - Min : `0`, Max : `100`, Pas : `1`

2. **input_button** — Bouton de synchronisation
   - **Creer une entree** > **Bouton**
   - Nom : `Synchro volets`
   - Un seul bouton peut etre partage entre plusieurs volets

### Temps de course

Pour connaitre le temps de course de ton volet, regarde l'entite `number.xxx_time_control` dans Home Assistant, ou chronometre une ouverture complete.

## Installation

### Methode 1 : Import direct (recommande)

1. Clique sur le bouton **"Import Blueprint"** ci-dessus
2. Home Assistant s'ouvre et te propose d'importer le blueprint
3. Configure l'automatisation via l'UI

### Methode 2 : Installation manuelle

```bash
cd config/blueprints/automation/
git clone https://github.com/nicotoine/cover-blueprints.git cover
```

Puis dans Home Assistant :
**Outils de developpement** > **YAML** > **Recharger les blueprints**

## Licence

MIT
