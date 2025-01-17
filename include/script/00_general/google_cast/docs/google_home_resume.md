# Background
I've shared [a script](https://community.home-assistant.io/t/script-to-resume-radio-tunein-and-spotify-after-tts-on-google-home-speakers/326634) last year, to resume a Google Cast device after a TTS has been sent to it. However, this script was limited to TTS only. I now created a new script based on the TTS script to perform other actions (like playing an mp3, casting a lovelace dashboard, showing an image, etc)
**Note:** Only service calls are supported, but you can call a script in a service call, so other actions can be performed by calling a script.

# This script supports
* Resuming of TuneIn, Spotify, YouTube and generic streams after any actions which interrupted the audio
* Resuming an entire speaker group after a single group member has been interrupted
* Resuming of individual group members after the speaker group has performed an action

# Requirements
* Home Assistant version 2022.2 is required because the `iif` filter/function introduced in that version is used in templates
* For Spotify you need to have the [Spotify integration ](https://www.home-assistant.io/integrations/spotify/) installed, and [Spotcast ](https://github.com/fondberg/spotcast/) (available on [HACS](https://github.com/hacs/integration))
* The script creates groups while running, so if you don't have any groups set up already, add `group:` to your configuration.yaml.

# To Do
* Make it possible to queue actions if the script is called multiple times for the same entity (this will require the script to be cut into different scripts)

# Most recent changes
### Version 1.6.1 - 14 March 2022
#### 🌟 Improvements
* Do not store unnecessary data of non playing devices

Older changes can be found [here](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/docs/changelog_google_home_resume.md)

# Prerequisites
1. The entity_id's for media_player entities from the Spotify integration should be formatted like `media_player.spotify_{{ spotcast user }}`. For the primary Spotcast user you can use whatever you want as `{{ spotcast user }}`.
1. The primary Spotcast user needs to be specified under `primary_spotcast` (see comment above).
1. To determine the Spotify account, the source in the Spotify media_players is used. This is compared to the friendly name of the Goolge Home media_player. Therefor the Google Home media players in HA need to have the exact same name as they have in the Google Home app (this is also already a requirement for Spotcast to work with entity_id's)
1. Google Nest Hub speakers can be entered under the variable `players_screen`. This will make sure the photo display is turned on again after the TTS in case nothing was already playing.
1. If you use speaker groups in the Google Home app, you can enter them under the variable `speaker_groups`. If you use them, you'll need to complete this variable, and add the group members in there as well (see the script for an example).

# Known limitations
* It is possible to create speaker groups on the fly from the Google Home app, e.g. if you are playing something from Spotify on your Kitchen speaker, you can add your Living Room speaker in the Google Home app, without them belonging to a speaker group. The script won't recognize these groups created on the fly. The cast integration won't recognize these devices as playing anymore, so they won't be resumed.
* When Spotify switches to a new song or starts playing, the Spotify Media Player will shortly not show as playing. When at that moment the script is started, the stream will not be resumed afterwards.
* YouTube and YouTube music will only resume the video/song which was playing at the time of the interruption, and only on players with a screen if not started using the [ytube_music_player](https://github.com/KoljaWindeler/ytube_music_player) custom integration.

# How to start the script
This script only conains the code to resume after the interruption, it doesn't contain any standard actions (like sending a TTS or playing an MP3 file.

To perform such an action, you need to provide them in the `action` field. In case you use the GUI, it will allow al kind of actions (like `delay` or `choose`) but  it can only handle service calls (so starting with `service`). 

You can also start a script in a service call, so this allows you to do more advanced actions, like using `choose` or a `wait_template`. But in that case there will be no information regarding the target of the service call in the action data. In that case you can enter that information in the `target` field.
There are examples below for both use cases.


The boolean 'resume_this_action` can be set to `false` if you don't want to resume the actions from the `action` field. To explain why you would want to do this, I have the following real life example:
I've set up a tag scanner on which my kids can scan a card, and then some song will play. If there was already something playing (a TuneIn stream for example) I want that stream to resume after the song finished. However, the kids tend to scan the card a second time when they don't like the song. If that happens the first kids song which was already playing, would be resumed afterwards. With resume_this_action: false this will not be the case.

*Description of fields:*
|Field|Required|Description|
| --- | --- | --- | 
|target|No|The targets which should be resumed, only needed if these targets are not clear from the actions. All usual targets (`area_id`, `device_id` and `entity_id`) are supported|
|action|Yes|The ations to be performed, only service calls are supported. If other actions are needed, you can create a script and call the script.|
|resume_this_action|No|Actions from the `action` field will not be resumed if set to `false`. Default is `true`.|

*Example without the requirement for `target`*
```yaml
alias: "Play sound when there is someone at the door"
service: script.google_home_resume
data:
  action:
    - service: media_player.volume_set
      target:
        area_id: 'living_room'
        entity_id:
          - media_player.bedroom
          - media_player.guestroom
      data:
        volume_level: 0.5
    - service: media_player.play_media
      target:
        area_id: 'living_room'
        entity_id:
          - media_player.bedroom
          - media_player.guestroom
      data:
        media_content_type: music
        media_content_id: "media-source://media_source/local/dingdong.mp3"
```

*Examle where another script is started, so it is not clear which players are affected*
```yaml
alias: "Play sound when there is someone at the door via script"
service: script.google_home_resume
data:
  target:
    area_id: 'living_room'
    entity_id:
      - media_player.bedroom
      - media_player.guestroom
  action:
    - service: script.play_sound
      data:
        file: "media-source://media_source/local/dingdong.mp3"
```

The script can also be started from the GUI, both in YAML mode and full GUI mode. 

# Send TTS and apply volume for the TTS message
Below you will find an example on how to use this script to send a TTS message and apply the right volume for this message. Only the TTS service call is in the script, the volume is applied in the actions below, as a `wait_template` is used, which is not allowed in the script `action` section.
This wait template will make sure the volume for the TTS message is not applied too soon, so when e.g. the Spotify stream is still playing. Note that I'm using `script.turn_on` here, otherwise the `wait_template` will start after the resume script is completed, which will actually apply the volume on the resumed stream.

```yaml
- alias: "Send TTS using Google Home Resume script"
  service: script.turn_on
  target:
    entity_id: script.google_home_resume
  data:
    variables:
      action:
        - alias: "Send TTS message"
          service: tts.google_cloud_say
          target:
            entity_id: media_player.zolder_mini_martijn
          data:
            message: "Ding Dong! Er staat iemand aan de deur!"
- alias: "Wait until the target is idle or off before applying the volume for the TTS message"
  wait_template: "{{ states('media_player.zolder_mini_martijn') in ['idle', 'off'] }}"
- alias: "Apply TTS volume"
  service: media_player.volume_set
  target:
    entity_id: media_player.zolder_mini_martijn
  data:
    volume_level: 0.35
```

# And finally the script itself

[Link to the script ](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/google_home_resume.yaml) on my Github config, so I don have to maintain it in two places

Add this script to `scripts.yaml` by copying the contents of the link below, and pasting it in `scripts.yaml`. Don't use the GUI, use a file editor (add-on).
Change the variables described below to match your setup.

# Explanation of variables in the script

There are no required variables, but if you use Google Home speaker groups and players with a screen you should define those. Resuming Spotify won't work properly without `default_spotcast`.

|Variable|Default|Example|Description|
| --- | --- | --- | --- |
|players_screen||[See script on Github ](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/google_home_resume.yaml#L32-L35)|Enter a list of cast devices with a screen. Do not use a comma seperated string here.|
|primary_spotcast||`pepijn`|The Spotify account which is used as primary account for spotcast, should match the last part of the Spotify media player.|
|fixed_picture||[See script on Github ](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/google_home_resume.yaml#L37-L39)|A dictionary with the pictures. As key value the artist should be used (check `media_artist` in developer tools > states)|
|speaker_groups||[See script on Github ](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/google_home_resume.yaml#L40-L60)|A combination of a dictionary and a list, with speaker groups of which all entities are included in another speaker group.|
|default_volume_level|`0.25`|`0.5`|The default volume level to use to set the entity to if the old volume can not be retreived|
|group_id||`some_string`|A string which will be added to the group object id's so they can be identified as belonging to this script|

# Why not a blueprint?
I've been asked a couple of times if I ever considered to make a blueprint out of this script. I do understand this would make updates more easy, however there are also some things which make it quite complicated:
* You need to provide information on your speaker groups, players with a screen and spotcast account. That would mean that you will have to do that each time you use the blueprint to create a script, or that you'll have to add this information each time the blueprint is updated, which would reduce the easiness of updating the script.
* I tried to bypass the point above by using includes, but the blueprint would do that once, and add the informaton in the yaml, instead of keeping the include code.
* I use really large and complicated templates, and these would be converted to really messy one line templates, making it impossible to read and debug if needed.

So, basically, I gave it a try, and decided it would not work :)

# Other scripts
For other related Google Home scripst, see my [Github page](https://github.com/TheFes/HA-configuration/tree/main/include/script/00_general/google_cast)

# Buy me a coffee
If you like this script, please feel free to buy me a coffee (I might spend it on another beverage though).
In case you decide to do so, thanks a lot!

<a href="https://www.buymeacoffee.com/thefes" target="_blank">![Buy Me A Coffee](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)</a>
