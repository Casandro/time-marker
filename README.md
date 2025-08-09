# Time Marker

This is a small device designed to monitor the frequency of power grids.
It consists of a piece of hardware which generates GNSS synced audio markers, and isolates and down-scales it so both can be fed into a sound-card.

From there software will read the markers and generate down-sampled, but propperly timed audio files.



## Overall concept
```
          GNSS (GPS, Galileo, etc)
                   |
Mains grid---Time Marker hardware-----Line-IN of PC

```

## Hardware

![schematics](images/schematics.png)
![board](images/board.png)

The hardware will get an interrupt from the PPS signal from the GNSS receiver as well as the time via serial.
This will start playing a correlation sample, then some data containing the current time.
Optionally you can also put that signal onto an RS-232 port, which is useful for getting the PPS signal into your PC.

## Audio interface
The idea is that soundcards typically have very decent AD converters, usually having sample rates of at least 48kHz and sample depths of at least 16 Bits. It's hard to do that with easy to get chips.
However the clock of those is very much undefined, plus there is an unknown amount of latency between the analog input and the application getting the data.

To circumvent this, the hardware generates markers on one channel, so the other channel is free for whatever signal you want to sample.

### Format

The format is not yet defined, however it'll likely use either FSK or DTMF for the data.
There will be a sync mark at the beginning of the second, since this is when the timing error is lowest. It will be something with good correlation properties.

A DTMF tone typically is recommended to be at least 50 ms long with a pause of 20 ms between two digits. This means that in a second we could have 14 digits. Leaving some room for the second marker and silence at the end, we could, for exmample, have 10 digits per second. One marker, 8 data digits and one parity or stop digit. The advantage is that DTMF has discrete tones so we can easily have a gap at the end of the signal.

With FSK, we could, for example, have V.21 modulated bits. We'd probably have something like 256 data bits when we subtract run-in periods and the like. We probably need to re-sync it as part of the second mark.

## Software
No software is written so far, however it would probably work like this:

1. Determine the exact position of the second markers via cross correlation and sub sample interpolation. (The auto-correlation function will likely be triangular around it's center which we can use for sub sample accuracy)
2. Resample the second in the desired sample rate (e.g. 200 Hz)
3. Demodulate the data part of the signal to determine what second is read.
4. Store / process that second of data

Since we will likely not be able to get good data on harmonics, it may be sensible to only look at the 50 Hz peak. For this we can do an IQ demodulation at 50 Hz, downsample this to, for example, 10 or 25 Hz, and only store that IQ data. In principle we can also just store the phase data.

