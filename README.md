# homebridge-philipstv-enhanced
Homebridge module for Philips TV (with JointSpace enabled) with Power on/off, Sound, Ambilight and input selection control.

# Description

This plugin is a fork of [homebridge-philipstv-x](https://www.npmjs.com/package/homebridge-philipstv-x) with additional support for Sound control, Ambilight brightness control and input selection control.

It has been modified to work on a 43PUS6753 TV (2018 model without Android) and may not work on older ones. Code may need adjustment for Ambilight to work on other 2018 models.

# Installation

1. Install homebridge using: npm install -g homebridge
2. Install this plugin using: npm install -g homebridge-philipstv-enhanced
3. Update your configuration file. See the sample below.

# Configuration
 
Example accessory config (needs to be added to the homebridge config.json):
  ```
 "accessories": [
 	{
 		"accessory": "PhilipsTV",
 		"name": "TV",
 		"ip_address": "10.0.1.23",
 		"poll_status_interval": "60",
		"model_year": 2018,
		"has_ssl": false,
		"has_ambilight": true
 	}
 ]
  ```
 
To be able to power on the TV when the TV is in standby mode, you will need the wol_url parameters with the mac address of your TV
Added test option for WakeOnWLAN:

 ```
"accessories": [
	{
		"accessory": "PhilipsTV",
		"name": "TV",
		"ip_address": "10.0.1.23",
		"poll_status_interval": "60",
		"model_year" : "2018",
		"has_ssl": false,
		"wol_url": "wol://18:8e:d5:a2:8c:66"
	}
]
 ```

# Credentials for 2016 (and newer?) models with Android TV

As per [this project](https://github.com/suborb/philips_android_tv) the Android TV 2016 models Philips use an authenticated HTTPS [JointSpace](http://jointspace.sourceforge.net/) API version 6.

Every control- or status-call needs [digest authentification](https://en.wikipedia.org/wiki/Digest_access_authentication) which contains of a pre generated username and password. You have to do this once for your TV. We recommend to use the python script [philips\_android\_tv](https://github.com/suborb/philips_android_tv). This script has a dependency on "pycrypto" and "requests" which you will need to run the script successfully.

Here is an example pairing call for philips\_android\_tv:
```
python ./philips.py --host 10.0.1.23 pair
```

You can then add username and password key in your homebridge config, example:
```
"accessories": [
  {
  	"accessory": "PhilipsTV",
  	"name": "TV",
  	"ip_address": "10.0.1.23",
  	"poll_status_interval": "60",
  	"model_year": 2016,
  	"has_ambilight": true,
  	"has_ssl": true,
  	"username": "deadbeef0815",
  	"password": "deadbeef0815deadbeef0815deadbeef0815deadbeef0815deadbeef0815"
  }
]
 ```
# Todo

We should auto detect TV capacity (http/https) and API version by requesting http://{tvip}:1925/{version}/system and https://{tvip}:1926/{version}/system by incrementing {version} as an integer from 1 until we get a JSON response.

A 2018 TV will answer to /6/system and will report the API version is 6.1

For ambilight, we should parse /6/menuitems/settings/structure to instead of relying on static nodeid in the code.
For audio, we should read /API_VERSION/audio/volume to get the max, and map the 0-100% of HomeKit to 0-TVMax. Currently 25 is assumed as max, so we divide homekit value by 4. 43PUS6753 has a volume max of 60 but 25 is already high.

The code needs cleanup. Especially it would be nice to use a generic setXStateLoop function, work stated with httpRequest_with_retry.

Get function for audio/ambilight/etc. should be modified to attend nothing when the TV is off. When the TV turns on, we should refresh all values.

# Dev notes about JointSpace URLs

POST to /6/menuitems/settings/current allow to get current Ambilight settings.

POST to /6/menuitems/settings/update allow to update Ambilight settings.

GET to /6/menuitems/settings/structure allow to have the details of the menu and options.

