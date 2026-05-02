# Blueprints Volets pour Home Assistant

Blueprints Home Assistant pour gerer des volets electriques sans retour de position (type Tuya/Etersky).

## Blueprints inclus

### Controle par position estimee

Controle un volet (0-100%) en se basant sur le temps de course complet. Ideal pour les interrupteurs qui ne remontent pas la position (Etersky WF-CS01, Allevoi CS01, etc.).

**Fonctionnalites :**
- Slider pour controler le volet au pourcentage
- Boutons ouvrir/fermer/stop via les helpers
- Mise a jour en temps reel de la position (chaque seconde)
- Gestion des appels consecutifs (mode restart)
- Recalibrage automatique chaque nuit via Alarmo (fermeture complete garantie)
- Bouton de recalibrage manuel

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fnicotoine%2Fcover-blueprints%2Fblob%2Fmaster%2Fcover_position%2Fcover_position.yaml)

## Installation

### Etape 1 : Creer les helpers

Dans **Parametres** > **Appareils et services** > **Entrees** :

**Par volet (ex: cuisine) :**

1. **input_number** — Position actuelle
   - Nom : `Position volet cuisine`
   - Min : `0`, Max : `100`, Pas : `1`

2. **input_number** — Position cible
   - Nom : `Cible volet cuisine`
   - Min : `0`, Max : `100`, Pas : `1`

3. **input_button** — Bouton stop
   - Nom : `Stop volet cuisine`

**Partage :**

4. **input_button** — Bouton recalibrage
   - Nom : `Synchro volets`

### Etape 2 : Importer le blueprint

1. Clique sur le bouton **"Import Blueprint"** ci-dessus
2. Cree une automatisation par volet avec :
   - **Volet reel** : `cover.volet_cuisine`
   - **Position actuelle** : `input_number.position_volet_cuisine`
   - **Position cible** : `input_number.cible_volet_cuisine`
   - **Bouton stop** : `input_button.stop_volet_cuisine`
   - **Temps de course** : valeur de `number.xxx_time_control` (ex: 70s)
   - **Alarmo** : ton panneau d'alarme
   - **Bouton recalibrage** : `input_button.synchro_volets`

### Etape 3 : Configurer le dashboard

Exemple avec des cartes standard :

```yaml
type: vertical-stack
cards:
  - type: horizontal-stack
    cards:
      - type: button
        name: Ouvrir
        icon: mdi:arrow-up-bold
        tap_action:
          action: perform-action
          perform_action: input_number.set_value
          target:
            entity_id: input_number.cible_volet_cuisine
          data:
            value: 100
      - type: button
        name: Stop
        icon: mdi:stop
        tap_action:
          action: perform-action
          perform_action: input_button.press
          target:
            entity_id: input_button.stop_volet_cuisine
      - type: button
        name: Fermer
        icon: mdi:arrow-down-bold
        tap_action:
          action: perform-action
          perform_action: input_number.set_value
          target:
            entity_id: input_number.cible_volet_cuisine
          data:
            value: 0
  - type: entities
    entities:
      - entity: input_number.cible_volet_cuisine
        name: Position cible
      - entity: input_number.position_volet_cuisine
        name: Position actuelle
```

Pour des cartes plus esthetiques, utilise
[Mushroom](https://github.com/piitaya/lovelace-mushroom) ou
[Button Card](https://github.com/custom-cards/button-card) via HACS
avec les memes actions.

## Fonctionnement

```
Dashboard (boutons / slider)
        |
  Ouvrir → cible = 100
  Fermer → cible = 0
  Stop   → input_button.press
  Slider → cible = valeur
        |
        v
  Blueprint (mode restart)
  1. Stop le vrai volet
  2. Calcule direction + duree
  3. Ouvre ou ferme le vrai volet
  4. Met a jour la position chaque seconde
  5. Arrete le vrai volet a la fin
        |
        v
  input_number.position (mis a jour en temps reel)
```

## Temps de course

Pour connaitre le temps de course de ton volet, regarde l'entite
`number.xxx_time_control` dans HA, ou chronometre une ouverture complete.

## Limites

- **Bouton physique** : les appuis sur l'interrupteur mural ne sont
  pas detectes. Le recalibrage nuit corrige la derive.
- **Commandes vocales** : necessitent de creer des scripts HA qui
  appellent `input_number.set_value`, puis de les exposer a
  Google Home / Alexa.

## Prerequis

### Alarmo

Un panneau d'alarme [Alarmo](https://github.com/nielsfaber/alarmo)
configure avec le mode nuit.

## Licence

MIT
