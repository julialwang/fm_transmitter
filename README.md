# FM Transmitter
Use the Raspberry Pi as an FM transmitter. Works on every Raspberry Pi board. See later portion for Raspbery Pi 4 exceptions.

Just get an FM receiver, connect a 20 - 40 cm plain wire to the Raspberry Pi's GPIO4 (PIN 7 on GPIO header) to act as an antena, and you are ready for broadcasting.

This project uses the general clock output to produce frequency modulated radio communication. It is based on an idea originally presented by [Oliver Mattos and Oskar Weigl](http://icrobotics.co.uk/wiki/index.php/Turning_the_Raspberry_Pi_Into_an_FM_Transmitter) at [PiFM project](http://icrobotics.co.uk/wiki/index.php/Turning_the_Raspberry_Pi_Into_an_FM_Transmitter).

## How to use it
Start by updating your Raspberry Pi with the following commands:
```
sudo apt-get update
sudo apt-get upgrade
```
Then, install the following packages:
```
sudo apt-get install -y sox make gcc g++ git arecord libmp3lame-dev
```
Then, clone this repository, then use `make` command as shown below:
```
cd fm_transmitter
make
``` 
After a successful build you can start transmitting by executing the "fm_transmitter" program:
```
sox /home/pi/fm_transmitter/acoustic_guitar_duet.wav -r 22050 -c 1 -b 16 -t wav - | sudo

```
Where:
* -r defines the sample rate that SOX will convert the file.
* -c defines the number of channels, due to limitations of FM Transmitter we cut it down to a single channel.
* -b defines the bit rate that the output should be sampled at.

After transmission has begun, simply tune an FM receiver to chosen frequency, You should hear the playback.
### Raspberry Pi 4
On Raspberry Pi 4 other built-in hardware probably interfers somehow with this software making transmitting not possible on all standard FM broadcasting frequencies. In this case it is recommended to:
1. Compile executable with option to use GPIO21 instead of GPIO4 (PIN 40 on GPIO header):
```
make GPIO21=1
```
2. Change either ARM core frequency scaling governor settings to "performance" or to change ARM minium and maximum core frequencies to one constant value (see: https://www.raspberrypi.org/forums/viewtopic.php?t=152692 ).
```
echo "performance"| sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```
3. Using lower FM broadcasting frequencies (below 93 MHz) when transmitting.
### Supported audio formats
You can transmitt uncompressed WAV (.wav) files directly or read audio data from stdin, eg.:
```
sudo apt-get install sox
sox acoustic_guitar_duet.wav -r 22050 -c 1 -b 16 -t wav - | sudo ./fm_transmitter -f 100.6 -
```
Please note only uncompressed WAV files are supported. If you receive the "corrupted data" error try converting the file, eg. by using SoX:
```
sudo apt-get install sox libsox-fmt-mp3
sox my-audio.mp3 -r 22050 -c 1 -b 16 -t wav my-converted-audio.wav
sudo ./fm_transmitter -f 100.6 my-converted-audio.wav
```

To add MP3 support, we need to compile and install FFMPEG, since FFMPEG is not available through packages for the Raspbian operating system we will have to do all of this manually.
```
cd /usr/src
sudo git clone https://code.videolan.org/videolan/x264.git
cd x264
sudo ./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
sudo make
sudo make install
cd /usr/src
sudo git clone git://source.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
sudo ./configure --arch=armv7-a --target-os=linux --enable-gpl --enable-libx264 --enable-nonfree --extra-cflags='-march=armv7-a -mfpu=neon-vfpv4 -mfloat-abi=hard'
sudo make -j4
sudo make install
```
These steps might take quite awhile; just be patient!

Lastly, you can run:
```
cd ~/fm_transmitter
sudo python ./PiStation.py -f 100.3 example.mp3
```
To play any .mp3 file of choice.

Included sample audio was created by [graham_makes](https://freesound.org/people/graham_makes/sounds/449409/) and published on [freesound.org](https://freesound.org/)
