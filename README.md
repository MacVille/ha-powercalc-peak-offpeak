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
  
#Komplette Einstellungen aus meinem Gebrauch

```yaml
    ###################################################################
    #                                                                 #
    #                  Berechnen von Stromkosten                      #
    #                                                                 #
    ###################################################################

    #Berechnung der Stomkosten in €/kWh für Hochtarif im Monat
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
      #value_template: >
      #  {{ states('sensor.stromverbrauch_hochtarif') | float * states('input_number.stromkosten_hochtarif_juli_dezember') | float | round(2)}}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: 19c3f876ebbe444f8ee0f11ce42dbda5

    #Berechnung der Stomkosten in €/kWh für Niedertarif im Monat
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
      #value_template: >
      #  {{ states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_juli_dezember') | round(2) }}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: 198fd71dd162428cb6ad26a937bb39b7

    #Berechnung ob Grundgebühr aussreicht oder nicht im Monat
    stromkosten_grundgebuehr:
      friendly_name: "Strom Grundgebühr rechnung"
      value_template: >
        {{ ((states('sensor.stromkosten_ht') | float + states('sensor.stromkosten_nt') | float ) - states('input_number.stromgrundgebuhr') |float ) |round(2)  }}
      icon_template: mdi:currency-eur
      unit_of_measurement: EUR
      unique_id: 6a932828c77d4028af5db57adc8531d3

    stromkosten_grundgebuehr_inkl_grundpreis:
      friendly_name: "Strom Grundgebühr inkl. Grundpreis"
      value_template: >
        {{ (states('sensor.stromkosten_grundgebuehr') | float + states('input_number.stromgrundpreis') | float) | round(2) }}
      icon_template: mdi:currency-eur
      unit_of_measurement: EUR
      unique_id: 29d75fd8ebf042379ab019cd3e223257

    #Sensor für Energie Dashboard welcher dynamisch die Tarife nutzt.
    strompreis_fuer_dashboard:
      friendly_name: "Strompreis für Energie Dashbaord"
      value_template: >
        {% if now().month in [1,2,3,4,5,6] %}
          {% if is_state('select.stromverbrauch', 'Hochtarif') %}
            {{ states('input_number.stromkosten_hochtarif_januar_juni') }}
          {% else %}
            {{ states('input_number.stromkosten_niedertarif_januar_juni') }}
          {% endif %}
        {% else %}
          {% if is_state('select.stromverbrauch', 'Hochtarif') %}
            {{ states('input_number.stromkosten_hochtarif_juli_dezember') }}
          {% else %}
            {{ states('input_number.stromkosten_niedertarif_juli_dezember') }}
          {% endif %}
        {% endif %}
      unit_of_measurement: '€/kWh'
      icon_template: mdi:currency-eur
      unique_id: 6361de1849dc4646906415793826b95c

    ###################################################################
    #                                                                 #
    #              Berechnen von Stromkosten täglich                  #
    #                                                                 #
    ###################################################################
    stromkosten_taeglich_ht:
      friendly_name: >-
        {% if now().month in [1,2,3,4,5,6] %}
          Kosten für Hochtarif täglich Sommer
        {% else %}
          Kosten für Hochtarif täglich Winter
        {% endif %}
      value_template: >-
        {% if now().month in [1,2,3,4,5,6] %}
          {{ (states('sensor.strom_taglich_hochtarif') | float * states('input_number.stromkosten_hochtarif_januar_juni') | float ) | round(2)}}
        {% else %}
          {{ (states('sensor.strom_taglich_hochtarif') | float * states('input_number.stromkosten_hochtarif_juli_dezember') | float ) | round(2)}}
        {% endif %}
      #value_template: >
      #  {{ states('sensor.stromverbrauch_hochtarif') | float * states('input_number.stromkosten_hochtarif_juli_dezember') | float | round(2)}}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: 2a7ed5b7a9114132a04c42728de09105

    #Berechnung der Stomkosten in €/kWh für Niedertarif
    stromkosten_taeglich_nt:
      friendly_name: >-
        {% if now().month in [1,2,3,4,5,6] %}
          Kosten für Niedertarif täglich Sommer
        {% else %}
          Kosten für Niedertarif täglich Winter
        {% endif %}
      value_template: >-
        {% if now().month in [1,2,3,4,5,6] %}
          {{ (states('sensor.strom_taglich_niedertarif') | float * states('input_number.stromkosten_niedertarif_januar_juni') | float ) | round(2) }}
        {% else %}
          {{ (states('sensor.strom_taglich_niedertarif') | float * states('input_number.stromkosten_niedertarif_juli_dezember') | float ) | round(2) }}
        {% endif %}
      #value_template: >
      #  {{ states('sensor.stromverbrauch_niedertarif') | float * states('input_number.stromkosten_niedertarif_juli_dezember') | round(2) }}
      unit_of_measurement: EUR
      icon_template: mdi:currency-eur
      unique_id: ec53a06e8df24e8cba061b914f1c7e96
```
