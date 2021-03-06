# Home Assistant - SMHI Weather Warnings & Alerts
Retrieve SMHI Warnings & Alerts in Sweden and trigger actions, view in the home assistant dashboard.

This custom component for Home Assistant fetches data from SMHI open API and parses it. The data
is then divided into a hash map of districts and messages per district. The component can be configured
to notify on all warnings or just for a specific district.

*This component is based on https://github.com/isabellaalstrom/sensor.krisinformation*

## Example Configuration

In configuration.yaml for Home Assistant:

Receive all notifications for all districts:
```
sensor:
  - platform: smhialert
    district: 'all'
```
Or specify a specific district as below:
```
sensor:
  - platform: smhialert
    district: '032'
```

Available districts are listed at the bottom of this README.

You can also chose to get the available information in Swedish by adding `language: 'sv'` to your config for the sensor.

## Custom Lovelace Card

Download the smhialert-card.js and place in your *www* folder of Home Assistant.

Configure the UI (raw edit) and add this to the lovelace configuration:
```
resources:
  - type: js
    url: /local/smhialert-card.js
```

Then you can add a custom card with the following settings:
```
entity: sensor.smhialert
title: SMHI Alerts
type: 'custom:smhialert-card'
```

Lovelace card screenshot:

![](https://github.com/lallassu/smhialert/blob/master/smhialert_example3.png)

## Automation Example Configuration

The below example configures an notification both as push notification and as an email notification.
It will use the prefabricated *notice* that is created by the component. It is also possible to 
use the data structure and configure the message manually (*sensor.smhialert.attributes.messages*).

In automations.yaml:
```
- id: smhialert
  alias: 'SMHI Alert'
  initial_state: 'on'
  trigger:
    platform: state
    entity_id: sensor.smhialert
    to: "Alert"
  action:
    - service: notify.push
      data_template:
         title: "SMHI Alert!"
         message: '{{states.sensor.smhialert.attributes.notice}}'
    - service: notify.email
      data_template:
         title: 'SMHI Alert!'
         message: '{{states.sensor.smhialert.attributes.notice}}'
```

Example of full attributes that could be used in the data template is as follows:
```
{
  "messages": {
    "019": {
      "name": "Norrbottens län inland",
      "msgs": [
        {
          "event": "Risk Forest fire",
          "event_color": "#ab56ac",
          "district_code": "019",
          "district_name": "Norrbottens län inland",
          "identifier": "smhi-bpm-1564087965423",
          "sent": "2019-07-25T22:54:41+02:00",
          "type": "Alert",
          "category": "Met",
          "certainty": "Possible",
          "severity": "Severe",
          "description": "När: Tisdag och onsdag\nVar: I den nordöstra delen\nIntensitet: Risken för bränder i skog och mark är lokalt stor\nKommentar: -",
          "link": "http://www.smhi.se/vadret/vadret-i-sverige/Varningar",
          "urgency": "Expected"
        }
      ]
    }
  },
  "notice": "[Severe] (2019-07-25T22:54:41+02:00)\nDistrict: Norrbottens län inland\nType: Alert\nCertainty: Possible\nDescr:\nNär: Tisdag och onsdag\nVar: I den nordöstra delen\nIntensitet: Risken för bränder i skog och mark är lokalt stor\nKommentar: -\nweb: http://www.smhi.se/vadret/vadret-i-sverige/Varningar?#ws=wpt-a,proxy=wpt-a,district=019,page=wpt-warning-alla'\n\n                ",
  "friendly_name": "SMHIAlert",
  "icon": "mdi:alert"
}
```

The *messages* contains an hash of districts (if 'all' is used) and each districts has *msgs* which is an array of all active messages for that district.

## Usage Screenshots
![](https://github.com/lallassu/smhialert/blob/master/smhialert_example1.png)
![](https://github.com/lallassu/smhialert/blob/master/smhialert_example2.png)

## Todo
- Alert if changes has occured. Alert is always triggering when there are _any_ alert and does not
  take changes into account.
- Be able to specify multiple specific districts.

## Districts
The below table is obtained by issuing:
```
curl -s https://opendata-download-warnings.smhi.se/api/version/2/districtviews/all.json|jq . |egrep '(id|name)' | perl -p -e 's%^\s*(.*),\n%$1%g' | perl -p -e 's%""id%\n"id%g' |tr  '"' ' ' | perl -p -e 's% : %:%g'
```

```
id: 030  name: Uppsala län,Upplandskusten
id: 031  name: Värmlands län
id: 032  name: Västerbottens län inland
id: 033  name: Västerbottens län kustland
id: 034  name: Västernorrlands län
id: 035  name: Västmanlands län
id: 036  name: Västra Götalands län,norra Västergötland
id: 037  name: Kalmar län,Öland
id: 038  name: Örebro län
id: 039  name: Östergötlands län
id: 040  name: Skåne län,Österlen
id: 041  name: Bottenviken
id: 042  name: Norra Kvarken
id: 043  name: Norra Bottenhavet
id: 044  name: Södra Bottenhavet
id: 001  name: Norrbottens län,norra Lapplandsfjällen
id: 045  name: Ålands hav
id: 002  name: Västerbottens län,södra Lapplandsfjällen
id: 046  name: Skärgårdshavet
id: 003  name: Jämtlands län,Jämtlandsfjällen
id: 047  name: Finska viken
id: 004  name: Jämtlands län,Härjedalsfjällen
id: 048  name: Norra Östersjön
id: 005  name: Dalarnas län,Dalafjällen
id: 049  name: Mellersta Östersjön
id: 006  name: Blekinge län
id: 007  name: Västra Götalands län,Bohuslän och Göteborg
id: 008  name: Dalarnas län utom Dalafjällen
id: 009  name: Västra Götalands län,inre Dalsland
id: 050  name: Rigabukten
id: 051  name: Sydöstra Östersjön
id: 052  name: Södra Östersjön
id: 053  name: Sydvästra Östersjön
id: 010  name: Gotlands län
id: 054  name: Bälten
id: 011  name: Gävleborgs län inland
id: 055  name: Öresund
id: 012  name: Hallands län
id: 056  name: Kattegatt
id: 013  name: Jämtlands län utom fjällen
id: 057  name: Skagerack
id: 014  name: Jönköpings län,västra delen utom syd om Vättern
id: 058  name: Vänern
id: 015  name: Jönköpings län,östra delen
id: 016  name: Kalmar län utom Öland
id: 017  name: Kronobergs län,västra delen
id: 018  name: Kronobergs län,östra delen
id: 019  name: Norrbottens län inland
id: 020  name: Norrbottens län kustland
id: 021  name: Stockholms län,Roslagskusten
id: 022  name: Västra Götalands län,Sjuhäradsbygden och Göta älv
id: 023  name: Skåne län utom Österlen
id: 024  name: Gävleborgs län kustland
id: 025  name: Stockholms län utom Roslagskusten
id: 026  name: Jönköpings län,syd om Vättern
id: 027  name: Västra Götalands län,sydväst Vänern
id: 028  name: Södermanlands län
id: 029  name: Uppsala län utom Upplandskusten %
```

