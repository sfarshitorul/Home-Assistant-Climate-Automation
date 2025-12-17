# HASS_Climate_Automation
This is my Node-RED automation for the home climate. I have a gas heater and radiators in all rooms + a multi-split HVAC unit that can heat/cool the bedroom and the living room<br/>

This is running on a <b>Raspi4 4GB RAM</b><br/>

Related hardware (some links refer to Romanian website as I am located in Romania). Most of these devices are Zigbee devices controlled via Zigbee2MQTT. The HVAC is the exception, that is WiFi but with full local control.<br/><br/>

Please note that this automation is agnostic, it gives no damn about that actual device you are using, as long as it can be controlled somehow by HA and the states of the those devices can be read by HA.<br/>
You can use ZWave, Matter, Thread, WiFi devices. My poison of choice is ZigBee. It doesn't mean it has to be your poison too.<br/>
  - Heater ON/OFF: ATTENTION: Installing this  MEANS WORKING WITH 220V power, so if you don't know what you are doing please call a professional!!<br/>
   <b><i><a href="https://www.wifistore.ro/cumpara/shelly-1-gen4-multiprotocol-wi-fi-bluetooth-zigbee-3-0-matter-126323">Shelly S4SW-001X16EU</a></i></b> - This is a DRY-CONTACT relay!
  - Radiator TRV: <b><i><a href="https://moeshouse.com/products/moes-zigbee-trv-by100">MOES BRT-100-TRV</a></i></b> & <b><i><a href="https://www.wifistore.ro/cumpara/sonoff-trvzb-zigbee-supapa-termostatica-pentru-radiator-85164">SONOFF TRVZB</a></i></b>
  - Temp sensors: <b><i><a href="https://www.wifistore.ro/cumpara/senzor-de-temperatura-si-umiditate-sonoff-snzb-02d-zigbee-3-0-cu-78113">SONOFF SNZB-02D</a></i></b> (I do not rely on TRV temp or HVAC temps as they are not very accurate and I might want to check the temp in certain parts of the room and not at the TRV/HVAC location.
  - Window sensors: <b><i><a href="https://www.ikea.com/ro/ro/p/parasoll-senzor-usa-fereastra-smart-alb-80504308/">IKEA Parasoll</a></i></b>
  - HVAC: <a href="https://www.daikin.ro/ro_ro/rezidential/products-and-advice/product-categories/air-conditioners/comfora.html">Daikin Comfora</a> + <a href="https://github.com/revk/ESP32-Faikout">Faikout module</a> ( I FUCKING hate cloud control for home devices so I love this product - if you can, please support the project) 

<br/>
<img width="1774" height="1103" alt="image" src="https://github.com/user-attachments/assets/ad7f94ab-3e34-4d81-9064-c3fc81709d13" />
<br />
<img width="1628" height="1147" alt="image" src="https://github.com/user-attachments/assets/c276a57f-b0b4-4337-a8d1-ae51f45985fa" />


## How it works

### Variables
These variables were defined in two different files and referrenced into <a href="configuration.yaml">configuration.yaml</a>. Add these two lines somewhere near the top of the file.
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

## Installation Notes
  0. PRE-Prequisites!!!!
      - <b>This comes with absolutely no warranty and you are using this at your OWN RISK! This is meant only as a startup point for your automation, if you think this one is close enough to what you need.</b>
      - <b>I cannot STRESS ENOUGH: DO NOT PLAY WITH 220V power if you don't know what you are doing!!!! <br/>Installing the dry-contact for controlling the heater might not work for you or you need to ask a professional about how would you do that.</b>
      - <b>Please note that I have no other means of controlling the heater. If zigbee decides to fuck off into a better world I need to open the heater and remove this relay and manually start the heater.</b>
  1. Prerequisites:
     - Ability to manually change the config YAML. For ease of editing these files I recommend the <b><i>Studio Code Server</i></b> community add-on. It's life made simple!
     - Install <b><i>NodeRED</i></b> if you haven't already.
     - Make sure your devices are working (TRV, Temp Sensors, HVACs, Door/Window Sensors) and are controllable through HA. <br/>Bonus points for FULL LOCAL control. You don't want to have your balls freeze off in the middle of the fucking winter because your internet sucks or because you hit god knows what API limit. FUCK THAT! Go Local!
     - Make sure you know your devices and have named them and their entities appropriately. (eg. your Living Room TRV is called something like <i>Living.TRV</i> and you temp sensor is called <i>Living.Temp</i>) <<br/>Keep this naming consistent accross all your devices. It will make your life easier in the long run.
  2. Set up the variables for the automation.
     - Using Studio Code Server or any other method of your choosing create a <b><i>Climate</i></b> folder inside your <b><i>Config</i></b> main folder.
     - Create two yaml files: <i>input_number.yaml</i> and <i>input_boolean.yaml</i> and add your required variables
     - Update the configuration.yaml to include these files.
     - Reload the configuration or restart HA to make sure these new variables are now seen by the system.
  3. Add the dashboard to make it easy to keep an eye on everything.
     - You can use the <a href="dashboard.yaml">Dashboard.yaml</a> as a starting point, but please update it to use your own entity names.
  4. Import the NodeRED flow
     - I recommend using Notepad++/Visual Studio Code or some similar text editor that knows JSON and edit this before you actually import it into NodeRED. It might be easier to make the bulk of the changes in a text editor instead of NodeRED. The JSON flow you want to edit is <a href="NodeRED-climate-flow.json">NodeRED-climate-flow.json</a>
     - However, the 2 functions nodes that exist in this flow will be better edited inside NodeRED. They should be quite self-explanatory for a beginner developer and not very hard to understand and modify for a non-developer
     - Deploy the flow and use the debug view to check if the object that gets outputed is similar to the sample one in <a href="unified-states.json">unified-states.json</a> files
