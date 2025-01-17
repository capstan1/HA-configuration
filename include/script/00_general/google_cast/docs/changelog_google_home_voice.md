# Changelog
* For [Google Home Voice script](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/google_home_voice.yaml)
* More information [here](https://github.com/TheFes/HA-configuration/blob/main/include/script/00_general/google_cast/docs/google_home_voice.md)

### Version 1.3.1 - 14 March 2022
#### 🌟 Improvements
* Do not store unnecessary data of non playing devices
#### 🧾 Documentation
* Created seperate file for older changes

### Version 1.3.0 - 25 February 2022
#### 🔴 BREAKING
* Changes in the way data is shared with the [Google Home Resume script](https://community.home-assistant.io/t/script-to-resume-radio-tunein-and-spotify-after-tts-on-google-home-speakers/326634). So if you update this script, make sure to also update the other one. However, as of this version, it is also possible to use this script without the Google Home Resume script
#### 🌟 Improvements
* The ytube music player integration will now resume at the right position in the track

### Version 1.2.1 - 14 February 2022 💟
#### 🐛 Bug fixes
* Fix for resuming ytube_music_player after voice command

### Version 
### Version 1.2.0 - 13 February 2022
#### 🔴 BREAKING
* Made several changes in how data is sent from this script to the [Google Home Resume script](https://community.home-assistant.io/t/script-to-resume-radio-tunein-and-spotify-after-tts-on-google-home-speakers/326634). So if you update this script, make sure to also update the other one. However, as of this version, it is also possible to use this script without the Google Home Resume script
#### ✨ New feature
* It is now possible to use `target_conversion`. This means you can give your voice command, and perform your actions on another target eg the speaker group the target belongs to.
* By setting the `use_resume` variable to `false` you can send the actions without using the Google Home Resume script. This can be used if you for example want to start a radio stream which is not in TuneIn, and do not want to resume other streams in case that one is stopped.
* The `target` determined by the ambient sound will only be used in case you omit the target in your service call. If you provide it, that target will be used. This can be useful if you also want to provide other actions, for example setting the state of an `input_boolean`.
* In case you don't have the need to use the Google Home Script at all, you can set `use_resume: false` in the variables of the Google Home Voice script, and it will just send the actions.
#### 🌟 Improvements
* Information is shared with the Google Home Resume script in a better way. This was also needed because now multiple targets or groups as target are possible, where in the old version there was always only one target.
* Information on for the `ytube_music_player` integration is now also shared.

### Version 1.1.0 - 10 February 2022
#### 🌟 Improvements
* No need to define `speaker_groups` anymore, as I found a way to define them using a template.

### Version 1.0.0 - 9 February 2022
#### ✨ New feature
* Now the new script is used, other actions besides TTS are supported

#### 🌟 Improvements
* Added version number as a variable to the `sequence`. This will make it more easy to see which changes there are since you last updated the script
* Several improvements in the templates
* Update of this post
