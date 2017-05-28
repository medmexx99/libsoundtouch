# Bose Soundtouch Python library

[![Build Status](https://travis-ci.org/CharlesBlonde/libsoundtouch.svg?branch=master)](https://travis-ci.org/CharlesBlonde/libsoundtouch) [![Coverage Status](https://coveralls.io/repos/github/CharlesBlonde/libsoundtouch/badge.svg?branch=master)](https://coveralls.io/github/CharlesBlonde/libsoundtouch?branch=master)

This Python 2.7+/3.4+ library allows you to control [Bose Soundtouch devices](https://www.soundtouch.com/).

This library has been created in order to create a component for the [Home Assistant](https://home-assistant.io/) project but is totally independent.

## How to use it ?


```shell
pip install libsoundtouch
```

```python
from libsoundtouch import soundtouch_device
from libsoundtouch.utils import Source, Type

device = soundtouch_device('192.168.1.1')
device.power_on()

# Config object
print(device.config.name)

# Status object
# device.status() will do an HTTP request. Try to cache this value if needed.
status = device.status()
print(status.source)
print(status.artist+ " - "+ status.track)
device.pause()
device.next_track()
device.play()

# Media Playback
# device.play_media(source, location, account, media_type) #account and media_type are optionals
device.play_media(Source.INTERNET_RADIO, '4712') # Studio Brussel
spot_user_id = '' # Should be filled in with your Spotify userID
# This userID can be found by playing Spotify on the connected SoundTouch speaker, and calling
# device.status().content_item.source_account
device.play_media(Source.SPOTIFY, 'spotify:track:5J59VOgvclrhLDYUoH5OaW', spot_user_id) # Bazart - Goud
# Local music (Windows media player, Itunes)
# Account ID can be found by playing local music on the connected Soundtouch speaker, and calling
# device.status().content_item.source_account
account_id = device.status().content_item.source_account
device.play_media(Source.LOCAL_MUSIC, 'album:1', account_id, Type.ALBUM)

# Volume object
# device.volume() will do an HTTP request. Try to cache this value if needed.
volume = device.volume()
print(volume.actual)
print(volume.muted)
device.set_volume(30) # 0..100

# Presets object
# device.presets() will do an HTTP request. Try to cache this value if needed.
presets = device.presets()
print(presets[0].name)
print(presets[0].source)
# Play preset 0
device.select_preset(presets[0])

# ZoneStatus object
# device.zone_status() will do an HTTP request. Try to cache this value if needed.
zone_status = device.zone_status()
print(zone_status.master_id)
print(len(zone_status.slaves))
```

## Supported features

### Basics commands

* power on/power off
* play/pause
* next/previous track
* volume setting (mute/set volume/volume up/volume down)
* repeat one/all/off
* shuffle on/off
* select preset (bookmark)
* playback selected music

### Multi-room

Soundtouch devices supports multi-room features called zones.

```python
from libsoundtouch import soundtouch_device

master = soundtouch_device('192.168.18.1')
slave1 = soundtouch_device('192.168.18.2')
slave2 = soundtouch_device('192.168.18.3')

# Create a new zone
master.create_zone([slave1, slave2])

# Remove a slave
master.remove_zone_slave([slave2])

# Add a slave
master.add_zone_slave([slave2])
```

### Websocket

Soudtouch devices support Websocket notifications in order to prevent pulling and to get immediate updates.

```python
from libsoundtouch import soundtouch_device
import time

# Events listeners

# Volume updated
def volume_listener(volume):
    print(volume.actual)

# Status updated
def status_listener(status):
    print(status.track)

# Presets updated
def preset_listener(presets):
    for preset in presets:
        print(preset.name)

# Zone updated
def zone_status_listener(zone_status):
    if zone_status:
        print(zone_status.master_id)
    else:
        print('no Zone')
        
device = soundtouch_device('192.168.18.1')

device.add_volume_listener(volume_listener)
device.add_status_listener(status_listener)
device.add_presets_listener(preset_listener)
device.add_zone_status_listener(zone_status_listener)

# Start websocket thread. Not started by default
device.start_notification()

time.sleep(600)  # Wait for events

```

## Incoming features

The following features are not yet implemented:

* Auto discovery: Soundtouch devices supports SSDP and MDNS auto discovery protocols. Tests has been done but not yet implemented

## Access to the official API documentation

For an unknown reason, the API documentation is not freely available but you can request to get it: http://developers.bose.com/.
You have to sent an email and you'll received a response in a minute with 2 PDF:
* SoundTouchAPI_Discovery.pdf: How to use SSDP and MDNS discovery protocols
* SoundTouchAPI_WebServices.pdf: REST API documentation. Be careful, the documentation contains errors and is not fully up to date

## Changelog

| Version |    Date    | Features                                                                   |
|---------|:----------:|----------------------------------------------------------------------------|
| 0.4.0   | 2017/05/21 | Add Bluetooth source                                                       |
| 0.3.0   | 2017/04/09 | Allow playing local computer media and fix issue with non ASCII characters |
| 0.2.2   | 2017/02/07 | Fix status with non ascii characters in Python 2.7                         |
| 0.2.1   | 2017/02/05 | Fix dependencies                                                           |
| 0.2.0   | 2017/02/05 | Add *play_media* support                                                   |
| 0.1.0   | 2016/11/20 | Initial release                                                            |

## Contributors

Thanks to:

* [Tyzer34](https://github.com/Tyzer34) (add *play_media* support)
* [wanderor](https://github.com/wanderor) (add local computer media support)
* [obadz](https://github.com/obadz) (add Bluetooth source)

