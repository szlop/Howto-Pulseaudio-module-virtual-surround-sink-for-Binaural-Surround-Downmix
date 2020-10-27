# Howto-Pulseaudio-module-virtual-surround-sink-for-Binaural-Surround-Downmix
Short howto for the Pulseaudio module-virtual-surround-sink module, which offers binaural downmix of 7.1 surround signals for headphones.

## Generation of Hrir File
Create a wav-file named hrir_in.wav containing one impulse response per channel (max 64 samples) in the following order:
```
L0 L1 C0 C0 SL0 SL1 SB0 SB1
```
Here L, C, SL, SB refer to the positions of the left, center, surround-left and surround-back speakers. 
Index 0 refers to the IR for the left ear, index 1 to the IR for the right ear:
```
L0: left hrir L
L1: right hrir of L
C0: left hrir of C
...
```
A sampling rate of 48kHz and 16bit integer format worked for me. The length of the IRs is truncated by Pulseaudio to 64 samples. The IRs present in the file are mirrored by Pulseaudio to get IRs for the speakers on the right side. I guess the second C0 is used for the LFE channel.

Use ffmpeg to add a channel map:
```
ffmpeg -i hrir_in.wav -filter 'channelmap=0|1|2|3|4|5|6|7:7.1' hrir.wav
```

## Configuration of Virtual Sink

If you want to bind the virtual sink to a specific sound card:

Get list of available audio devices
```
pacmd list-sinks | grep -e 'name:'
 ```
Load the sink module as follows:
```
pacmd load-module module-virtual-surround-sink sink_name=vsurround sink_properties=device.description=VirtualSurround hrir=/PATH/TO/hrir.wav  sink_master=PA_AUDIO_DEVICE_NAME
```
Instead of PA_AUDIO_DEVICE_NAME, fill in the name of the sound device, you want to send the binaural downmix to. You can also omit the sink_master parameter and configure the output device via pavucontrol. Test the created vsurround device, e.g. by playing this file in VLC: https://www2.iis.fraunhofer.de/AAC/7.1auditionOutLeader%20v2.wav

If everything works, make the sink persistent by adding the part after pacmd to ~/.config/pulse/default.pa:
```
#!/usr/bin/pulseaudio -nF

.include /etc/pulse/default.pa

load-module module-virtual-surround-sink sink_name=vsurround sink_properties=device.description=VirtualSurround hrir=/PATH/TO/hrir.wav  sink_master=PA_AUDIO_DEVICE_NAME
```
