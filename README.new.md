# Scrypted advanced notifier

https://github.com/apocaliss92/scrypted-advanced-notifier - For requests and bugs

<!-- ## Key features
- `MQTT autodiscovery` of the most common functionalities of cameras
- `Detection rules`: setup your custom rules for a single camera (or for multiple in a batch) definining what to detect, where and how. Set which notifiers to be used
- `Occupancy rules`: check the occupancy of a certain area and define the parameters based on your use case
- `Timelapse rules`: define a timespan, the plugin will generate a timelapse at the end of the defined period. The timelapses generated will be available as videoclip on a camera entity
- `Audio rules`: set a threshold and send a notification when the threshold is reached for a certain amount of time
- `Notifiers` currently fully supported: Homeassistant push notifications and Pushover. Can be extended
- `AI`: let your preferred AI service the notification descriptions for a detection rule. Currently supported: GoogleAi, OpenAi, Claude, Groq
- `Images`: besides MQTT, images will be stored also on the filesystem in a specific mounted folder reachable by Scrypted
- `Custom texts`: define your customized text for your notifications, every rule can be customized specifically -->

# Getting started
## MQTT
To enable MQTT exporting:
- enable `MQTT enabled` in the general -> general tab
- setup the authentication parameters in the tab general -> MQTT, check the `Use MQTT plugin credentials` to use the credentials set on the MQTT plugin
- Check `Use NVR detections` if you want the images stored on MQTT to be the clipped ones from NVR. 
- Cameras enabled to the plugin will be automatically enabled to MQTT. Can be disabled in the camera's section Advanced notifier -> Report to MQTT

The plugin will export to MQTT the following entities:
- PTZ controls
- Restart control
- Basic detection information (motion, animal, person, vehicle, face, plate). Same information will be available for every rule associated to a camera
    - Latest image
    - Triggered
    - Last trigger time (disabled by default)
- Online status
- Sleeping status
- Battery status
- Recording switch (NVR privacy mode)

## Notifications
The plugin provides customized way to deliver notifications. It is based on rules. Each rule can be activated based on several factors, i.e. active sensors, time ranges, security system status. Any notifier can be used but the only fully supported currently are (can be extended for any other):
- Homeassistant push notifications
- Pushover

Rules can be of following types:
### Detection

<!-- ## Homeassistant
It's possbile to configure an homeassistant connection (or utilize the one configured in the `Homeassistant` plugin) to fetch configured Rooms and Entities which are identified by one of the `Entity regex patterns`, these can be then assigned to a camera or sensor to add metadata used to render the notification. The fetched data will be visible in the `METADATA` group, and edited 

## General
 - `Scrypted token`: Token stored on the scrypted entity on homeassistant
 - `NVR url`: Url pointing to the NVR instance, should be accessible from outside
 - `Storage path`: If set, the images used to populate MQTT topic will be also stored on the drive path
 - `Images name`: The name pattern to use to generate image files. The placeholders ${name} and ${timestamp} will be available. Using only ${name} will ensure the image to be overriden on every detection instead of saving one additional copy

## MQTT
 It's possible to use MQTT to report data to homeassistant, can be configured standalone or use the configuration from the main `MQTT` plugin. 
 - `Active entities topic`: topic the plugin will subscribe to activate the rules `OnActive`. Useful to sync the plugin with an alarm system connected to homeassistant (i.e. Alarmo)
 - `Active devices for MQTT reporting`: devices selected will be periodically reported to MQTT with several information, i.e. last detections happened, images, status and so on
 - `Use NVR detections`: MQTT topics will be published using the detections coming from NVR detections, instead of the one provided by the plugin

## Notifier
Mainly supported notifiers are from `Homeassistant` and `Pushover` plugins
- `Active notifiers`: master controller of active notifiers. Each rule will specify it's own set of notifiers

## Texts
List of strings that will be shown on the notifications based on the detection type. Useful to have localized text, many placeholders are available and specified in each text
## Detection rules
Fine grained rules can be defined to filter out detections and dispatch to specific notiries at specific conditions, called `Detection rules`. These rules can be added on Plugin level or on Camera level. Each rule has the following settings:
- `Enabled`: Enable or disable the rule (On homeassistant will be available a switch to enable/disable each rule)
- `Activation`: One of Always, OnActive, Schedule
    - Always - the rule will always be active (besides enabled flag being off)
    - OnActive - the rule will be active only for the devices selected in the `"OnActive" devices` selector (in Detection Rules -> General). This target is automatically synced with the MQTT topic defined in the setting "Active entities topic" under MQTT. MQTT Must be enabled
    - Schedule - the rule will be active based on a schedule defined in the rule
    - AlarmSystem - the rule will be active based on the current status of the alarm system defined in Plugin => Detection Rules => General => Security System
- `Priority`: Priority of the notification, will have effect only for pushover
- `Custom text`: override text to show on each notification. Will override the defaults
- `Detection classes`: detection classes to trigger the notification
- `Disable recording in seconds`: if set, when the rule is triggered will enable the NVR recordings for the same amount of seconds, will disable afterwards
- `Score threshold`: minimum score to trigger a notification
- `Notifiers`: notifiers to notify
- `Open sensors`: sensors that must be open to trigger a notification
- `Closed sensors`: sensors that must be closed to trigger a notification
- `Alarm modes`: alarm modes to be active to enable this rule (only available for activation AlarmSystem)
- `Actions`: actions that will be shown on the notification. Rendering will vary depending on the notifier. For HA will be an actionable notification, for pushover will be additional links in the text. Both of them require homeassistant to work, the same event will be triggered with the specified action type
- `Devices`: Only available for `Always` and `Schedule` activations. Devices for which notification is active
- `Day - Start time - End time`: properties required for the `Schedule` activation

The same detection rules can be defined on each camera level with some additional properties
- `Whitelisted zones`: Only detections on these zones will trigger a notification
- `Blacklisted zones`: Detections on these zones will be ignored

## Occupancy rules
Similar concept applied to occupancy, a combination of observe zone + detection class can be set to check if the zone is occupied or not
### General configurations
- `Object detector`: Plugin to use to execute the object detection (Overrides the setting specified in the plugin section)
- `Score threshold`: minimum score to trigger the occupancy in bulk (not used for now)

### Rule configurations
- `Enabled`: Enable or disable the rule (On homeassistant will be available a switch to enable/disable each rule)
- `Detection class`: Detection class to match in the zone
- `Observe zone`: Zone of type 'Observe' that will be matched
- `Zone type`: Intersect if the match can happen on any intersection, Contain if the detection must happen completely inside the zone
- `Score threshold`: minimum score to trigger the occupancy
- `Occupancy confirmation`: minimum amount of seconds to wait if the state should be updated. This should avoid some false positives
- `Zone occupied text`: Text that will be notified when the zone gets occupied
- `Zone not occupied text`: Text that will be notified when the zone becomes free
- `Notifiers`: notifiers to notify
- `Priority`: Priority of the notification, will have effect only for pushover
- `Actions`: actions that will be shown on the notification. Rendering will vary depending on the notifier. For HA will be an actionable notification, for pushover will be additional links in the text. Both of them require homeassistant to work, the same event will be triggered with the specified action type

## Timelapse rules
Allow to generate regular timelapses for a specific camera. The output folder will be `Images path`, if not set, the plugin folder will be used instead. The frames of a session will be persisted until the start of the next one, to give the possibility to finetune the result after a session
### General configurations
- `Notificataion message`: Message sent when the timelapse will be notified
- `Timelapse framerate`: FPS of the final timelapse. The higher the value, the faster will be the timelapse
- `Force snapshot seconds`: A new snapshot will be pushed to the timelapse on regular intervals
- Schedule properties for the time period to capture. This functionality is very well implemented with the homeassistant notifications, the video will be shown in the preview of the notifications

### Rule configurations
- `Enabled`: Enable or disable the rule (On homeassistant will be available a switch to enable/disable each rule)
- `Detection class`: Detection class to match in the zone
- `Observe zone`: Zone of type 'Observe' that will be matched
- `Zone type`: Intersect if the match can happen on any intersection, Contain if the detection must happen completely inside the zone
- `Score threshold`: minimum score to trigger the occupancy
- `Occupancy confirmation`: minimum amount of seconds to wait if the state should be updated. This should avoid some false positives
- `Zone occupied text`: Text that will be notified when the zone gets occupied
- `Zone not occupied text`: Text that will be notified when the zone becomes free
- `Notifiers`: notifiers to notify
- `Priority`: Priority of the notification, will have effect only for pushover
- `Actions`: actions that will be shown on the notification. Rendering will vary depending on the notifier. For HA will be an actionable notification, for pushover will be additional links in the text. Both of them require homeassistant to work, the same event will be triggered with the specified action type

## Test
A test notification can be send with the specified settings

## Device mixin
On each camera/sensor can be set some metadata to enhance the notifications
#### General
- `Room`: room where the camera is located
- `EntityID`: alias of the camera (i.e. on homeassistant). Only used to identify the camera when syncing the `OnActive` devices from Homeassistant
- `Device class`: homeassistant device class to specify the type of the sensor created on MQTT. Defaults will be fine
#### Notifier
- `Actions`: actionable notifications added to the notification
- `Minimum notification delay`: minimum amount of seconds to wait between notification for the same combination of room-detectionClass
- `Snapshot width/height`: dimensions of the snapshot (only for cameras)
- `Ignore camera detections`: ignore detections from the camera. Should always be disabled if the camera is not active on the NVR, otherwise no detections would ever happen  (only for cameras)
- `Linked camera`: camera linked to this sensor. Any event happening on the sensor will use the provided camera for the snapshot
#### Webhook (only for cameras)
Simple webooks to retrieve information, only the last snapshot is for now available, could be extended with something else in the future
## Notifier mixin
- `Snapshot scale`: scale up/down the snapshot of a camera. If 1 will use the originated on camera and will improve performances
- `Texts`: override of the plugin level texts -->
