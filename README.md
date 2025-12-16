# HASS_Climate_Automation
My Node-RED automation for the home climate. I have a gas heater and radiators in all rooms + a multi-split HVAC unit that can heat/cool the bedroom and the living room<br/>

This is running on a <b>Raspi4 4GB RAM</b><br/>
Related hardware (some links refer to Romanian website as I am located in Romania). Most of these devices are Zigbee devices controlled via Zigbee2MQTT. The HVAC is the exception, that is WiFi but with full local control.
  - Heater TRV: <b><i><a href="https://moeshouse.com/products/moes-zigbee-trv-by100">MOES BRT-100-TRV</a></i></b> & <b><i><a href="https://www.wifistore.ro/cumpara/sonoff-trvzb-zigbee-supapa-termostatica-pentru-radiator-85164">SONOFF TRVZB</a></i></b>
  - Temp sensors: <b><i><a href="https://www.wifistore.ro/cumpara/senzor-de-temperatura-si-umiditate-sonoff-snzb-02d-zigbee-3-0-cu-78113">SONOFF SNZB-02D</a></i></b> (I do not rely on TRV temp or HVAC temps as they are not very accurate and I might want to check the temp in certain parts of the room and not at the TRV/HVAC location.
  - Window sensors: <b><i><a href="https://www.ikea.com/ro/ro/p/parasoll-senzor-usa-fereastra-smart-alb-80504308/">IKEA E2013</a></i></b>
  - HVAC: <a href="https://www.daikin.ro/ro_ro/rezidential/products-and-advice/product-categories/air-conditioners/comfora.html">Daikin Comfora</a> + <a href="https://github.com/revk/ESP32-Faikout">Faikout module</a> ( I FUCKING hate cloud control for home devices so I love this product - if you can, please support the project) 

<br/>
<img width="1774" height="1103" alt="image" src="https://github.com/user-attachments/assets/ad7f94ab-3e34-4d81-9064-c3fc81709d13" />

## How it works

### Variables
These variables were defined in configuration.yaml (for now): (refer to <a href="configuration.yaml">configuration.yaml</a> file for all the variables)
  1. We are defining some day periods to have different temps based on these. These can be customized in the interface. These refer to the hour (in 24 hour format when that respective period begins)
     - bedroom_morning_start
     - bedroom_day_start
     - bedroom_evening_start
     - bedroom_night_start
  2.  We are also defining some floats for each room and period that can also be customizable in the interface:
      - bedroom_morning_temp
      - bedroom_day_temp
      - bedroom_evening_temp
      - bedroom_night_temp
  3. For rooms that can influence the heater we have 1 additional variable: ( these are used simply to display in the interface; easily see which rooms requested heat)
      - bedroom_heater_needed
  4. We have some generic variables that influence the climate automation:
      - summer_mode (disable heat)
      - we_are_away (set all zone to the away_temp) and ignore all other logic
      - away_temp
    
### Entity states
Check this for a sample of how this object looks like: <a href="unified-states.json">unified-states.json</a>
In the <b>Get States and Build Unified Object</b> node in the flow we are creating a json object for all the entity states that are of interest. <br />
There is an <b>events:all</b> node that listens to <b>state_changed</b> events and if the entity that triggered is within a certain list (we need to monitor only specific entities). The fields that get monitored are defined in the <b><i>processCurrentEvent</i></b> function inside the <b>Get States and Build Unified Object</b> node. <br />

I do not want to cache this object or persist it in memory or disk because entity values may change by the next time the HA is restarted or updated. The way I've done it is not that resource consuming as to become a problem. Maybe if you live inside the Buckinham palace and have 2 tons of variables. But then again you might run HA on more powerful hardware.

### Logic
This is handled into the <b>Room Logic</b> node: <b><i>Apply_[ROOM}_Logic</i></b>. Rooms may be similar and could've grouped them but this way it allows for greater flexibility in the future. Again, considering you are not the King of England, for 100 romms this might be cumbersome.<br/>
These functions simply update the <i>msg.paylog.rooms.{ROOM}</i> object and set/unset the heater_needed property.<br/><br/>
The <i>msg.payload</i> object has a property for each room where it set the temp on each room TRV. I do not partially open them. The TRV is set to either 5 Celsius (fully closed) or 35 (fully open)<br/>
There are 2 additional functions: 
  1. <b><i>Check_If_Heat_Needed</i></b> - If heat is needed we also check if there are rooms that might start in the near future and start them as well, even if they would not currently need heat.
  2. <b><i>Check_If_Actions_Needed</i></b> - Monitor some properties in the <i>msg.payload</i> object and only trigger the next actions if there are status changes on these properties.

### Next Steps
  1. Set the TRV to the desired position
  2. If heat required - start the heater; otherwise turn it off
  3. if there are rooms that have heating capable HVAC start that as well. Radiators are not very efficient and the HVAC helps a lot in shortening the time needed to run the heater.
   
