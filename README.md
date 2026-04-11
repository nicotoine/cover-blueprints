# Blueprints Volets pour Home Assistant

Blueprints Home Assistant pour gerer des volets electriques sans retour de position (type Tuya/Etersky).

## Blueprints inclus

### Controle par position estimee

Controle un volet (0-100%) en se basant sur le temps de course complet. Ideal pour les interrupteurs qui ne remontent pas la position (Etersky WF-CS01, Allevoi CS01, etc.).

**Fonctionnalites :**
- Boutons ouvrir/fermer, commandes vocales et slider fonctionnent nativement
- Mise a jour en temps reel de la position estimee (chaque seconde)
- Gestion des appels consecutifs (mode restart)
- Recalibrage automatique chaque nuit via Alarmo (fermeture complete)
- Bouton de synchro manuelle

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fnicotoine%2Fcover-blueprints%2Fblob%2Fmaster%2Fcover_position%2Fcover_position.yaml)

## Installation

### Etape 1 : Creer les helpers (par volet)

Dans **Parametres** > **Appareils et services** > **Entrees** :

1. **input_number** — Position actuelle
   - Nom : `Position volet cuisine`
   - Min : `0`, Max : `100`, Pas : `1`

2. **input_number** — Position cible
   - Nom : `Cible volet cuisine`
   - Min : `0`, Max : `100`, Pas : `1`

3. **input_button** — Bouton de synchronisation
   - Nom : `Synchro volets`
   - Un seul bouton peut etre partage entre plusieurs volets

### Etape 2 : Ajouter le template cover

Le template cover encapsule le vrai volet et route toutes les commandes
(boutons, voix, slider) vers les helpers. C'est ce qui permet au
blueprint de fonctionner avec tous les modes de controle.

Ajoute ceci dans ton `configuration.yaml` :

```yaml
template:
  - cover:
      - name: "Volet Cuisine"
        unique_id: volet_cuisine_smart
        device_class: shutter
        state: >
          {% set pos = states('input_number.position_volet_cuisine') | float(0) %}
          {% set target = states('input_number.cible_volet_cuisine') | float(0) %}
          {% if (target - pos) > 1 %}
            opening
          {% elif (pos - target) > 1 %}
            closing
          {% elif pos <= 1 %}
            closed
          {% else %}
            open
          {% endif %}
        position: >
          {{ states('input_number.position_volet_cuisine') | int(0) }}
        open_cover:
          - action: input_number.set_value
            target:
              entity_id: input_number.cible_volet_cuisine
            data:
              value: 100
        close_cover:
          - action: input_number.set_value
            target:
              entity_id: input_number.cible_volet_cuisine
            data:
              value: 0
        stop_cover:
          - action: cover.stop_cover
            target:
              entity_id: cover.volet_cuisine
          - action: input_number.set_value
            target:
              entity_id: input_number.cible_volet_cuisine
            data:
              value: "{{ states('input_number.position_volet_cuisine') | int(0) }}"
        set_cover_position:
          - action: input_number.set_value
            target:
              entity_id: input_number.cible_volet_cuisine
            data:
              value: "{{ position }}"
```

Pour ajouter un autre volet (ex: salon), duplique le bloc en changeant
les noms d'entites (`volet_salon`, `position_volet_salon`, etc.).

Apres modification, redemarre HA :
**Parametres** > **Systeme** > **Redemarrer**

### Etape 3 : Importer le blueprint

1. Clique sur le bouton **"Import Blueprint"** ci-dessus
2. Configure une automatisation par volet :
   - **Volet reel** : `cover.volet_cuisine` (l'entite Tuya Local)
   - **Position actuelle** : `input_number.position_volet_cuisine`
   - **Position cible** : `input_number.cible_volet_cuisine`
   - **Temps de course** : valeur de `number.xxx_time_control` (ex: 70s)

### Etape 4 : Configurer le dashboard

Utilise le **template cover** (`cover.volet_cuisine_smart`) dans ton
dashboard, pas le cover reel. Le template cover affiche :
- L'etat (opening/closing/open/closed)
- La position en pourcentage
- Les boutons ouvrir/fermer/stop fonctionnels

Tu peux aussi desactiver l'entite reelle (`cover.volet_cuisine`) dans
les parametres de l'entite pour eviter la confusion.

### Etape 5 : Commandes vocales

Expose le template cover a Google Home / Alexa (pas le cover reel).
Les commandes vocales ("ouvre le volet cuisine") passeront par le
template cover et seront correctement suivies par le blueprint.

## Fonctionnement technique

```
Bouton / Voix / Slider
        |
        v
  Template Cover
  (open=100, close=0, stop=pos, set_position=valeur)
        |
        v
  input_number.cible_volet_xxx  (change de valeur)
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
  input_number.position_volet_xxx  (mis a jour en temps reel)
        |
        v
  Template Cover  (affiche etat + position)
```

## Temps de course

Pour connaitre le temps de course de ton volet, regarde l'entite
`number.xxx_time_control` dans Home Assistant, ou chronometre une
ouverture complete.

## Prerequis

### Alarmo

Un panneau d'alarme [Alarmo](https://github.com/nielsfaber/alarmo)
configure avec le mode nuit.

## Licence

MIT
