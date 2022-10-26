# Berechnen von Hochtarif und Niedertarif
Das ist ein Beispiel wie man in Home Assistant für Hoch und Niedertarif die Preise berechnet

## Vorraussetzungen

### Zwei input_number Helper anlegen

Hochtarif:
![input_helper für €/kWh Hochtarif](/Bilder/Euro%20pro%20kWh%20Hoch.png)

Niedertarif:
![input_helper für €/kWh Niedertarif](/Bilder/Euro%20pro%20kWh%20nieder.png)

### Anlegen von Utility Meter
![Verbrauchszähler](/Bilder/utility_meter.png)

Der Utility Meter wird benötigt für den Wechsel zwischen Hoch- und Niedertarif
Dies kann als Helper oder über YAML geschehen

## Benötigte Autmationen

```yaml
alias: "[Strom] Hochtarif"
description: ""
trigger:
  - platform: time
    at: "07:00:00"
condition:
  - condition: state
    entity_id: binary_sensor.workday_sensor
    state: "on"
action:
  - service: select.select_option
    data:
      option: Hochtarif
    target:
      entity_id:
        - select.stromverbrauch
        - select.strom_taglich
mode: single
```
- Diese Automation stellt um 07:00 Uhr an Arbeitstagen den Tarif aus dem Utility Meter auf Hochtarif. 
  ***Die gewählten Zeiten und Tage sind abhängig vom Stromlieferanten. Diese muss entsprechend Angepasst werden.***

```yaml
alias: "[Strom] Niedertarif"
description: ""
trigger:
  - platform: time
    at: "20:00:00"
condition:
  - condition: state
    entity_id: binary_sensor.workday_sensor
    state: "on"
action:
  - service: select.select_option
    data:
      option: Niedertarif
    target:
      entity_id:
        - select.stromverbrauch
        - select.strom_taglich
mode: single
```
- Diese Automation stellt um 20:00 Uhr den Tarif aus dem Utility Meter auf Niedertarif
  In meinem Fall gilt das für 20:00 - 07:00 Uhr unter der Woche und an Wochenenden und Feiertagen ganztägig
  ***Die gewählten Zeiten und Tage sind abhängig vom Stromlieferanten. Diese muss entsprechend Angepasst werden.***
  
## Berechnung der Kosten je nach definierten Tarif

### Monatliche Kosten für Hochtarif
```yaml
    stromkosten_ht:
      friendly_name: >-
        {% if now().month in [1,2,3,4,5,6] %}
          Kosten für Hochtarif Sommer
        {% else %}
          Kosten für Hochtarif Winter
        {% endif %}
      value_template: >-
        {% if now().month in [1,2,3,4,5,6] %}
          {{ (states('sensor.stromverbrauch_hochtarif') | float * states('input_number.stromkosten_hochtarif_januar_juni') | float ) | round(2)}}
        {% else %}
          {{ (states('sensor.stromverbrauch_hochtarif') | float * states('input_number.stromkosten_hochtarif_juli_dezember') | float ) | round(2)}}
        {% endif %}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: 19c3f876ebbe444f8ee0f11ce42dbda5
 ```
 ### Monatliche Kosten für Niedertarif
 ```yaml
     stromkosten_nt:
      friendly_name: >-
        {% if now().month in [1,2,3,4,5,6] %}
          Kosten für Niedertarif Sommer
        {% else %}
          Kosten für Niedertarif Winter
        {% endif %}
      value_template: >-
        {% if now().month in [1,2,3,4,5,6] %}
          {{ (states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_januar_juni') | float ) | round(2) }}
        {% else %}
          {{ (states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_juli_dezember') | float ) | round(2) }}
        {% endif %}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: 198fd71dd162428cb6ad26a937bb39b7
```

  ### *Erleuterung*
  ```yaml
  {% if now().month in [1,2,3,4,5,6] %}
    Kosten für Niedertarif Sommer
  {% else %}
    Kosten für Niedertarif Winter
  {% endif %}
  ```
  Wenn der aktuelle Monat zwischen Januar und Juni oder der Else fall zutrifft, wird entsprechend der Name des Sensors angepasst.
  
  ```yaml
  {% if now().month in [1,2,3,4,5,6] %}
    {{ (states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_januar_juni') | float ) | round(2) }}
  {% else %}
    {{ (states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_juli_dezember') | float ) | round(2) }}
  {% endif %}
  ```
  Wenn der aktuelle Monat zwischine Januar und Juni oder der Else fall zutrifft, wird der verbrauchte kWH Wert nach Tarif mit dem entsprechenden €/kWh Wert multipliziert und auf zwei Nachkommastellen gerundet.
  
  Selbige Einstellungen müssen Natürlich auch für den Hochtarif gemacht werden.
  
