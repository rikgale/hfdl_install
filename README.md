# SoapySDR and dumpHFDL Install
Instructions for installing soapysdr with RSP1(A/B) + RTL (Airspy support to be added later - for use with soapysdr only) and dumphdfl. Written for debian based systems. Tested on Raspberry Pi OS and Ubuntu.

**READ all** instrctions from top to bottom before starting
**Do not** plug in your SDR until instructed by the instructions below.

Credit to Alex KD6VPH for putting the initial iteration together.

### 1) Ensure system is fully up-to-date.
```shell
sudo apt update && sudo apt upgrade -y
```

### 2a) Install dumpHFDL dependancies.
```shell
sudo apt install build-essential cmake pkg-config libglib2.0-dev libconfig++-dev libliquid-dev libfftw3-dev git
```

### 2b) Install RTL-SDR dependancies 
Do not update drivers from OS package manager
```shell
# purge any old rtlsdr drivers
sudo apt purge ^librtlsdr
sudo rm -rvf /usr/lib/librtlsdr* /usr/include/rtl-sdr* /usr/local/lib/librtlsdr* /usr/local/include/rtl-sdr* /usr/local/include/rtl_* /usr/local/bin/rtl_*

# download and build new rtlsdr drivers
sudo apt install libusb-1.0-0-dev git cmake pkg-config
git clone https://github.com/rtlsdrblog/rtl-sdr-blog
cd rtl-sdr-blog/
mkdir build
cd build
cmake ../ -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo cp ../rtl-sdr.rules /etc/udev/rules.d/
sudo ldconfig
echo 'blacklist dvb_usb_rtl28xxu' | sudo tee --append /etc/modprobe.d/blacklist-dvb_usb_rtl28xxu.conf
cd
```

### 3) Install SDRPlay API 

```shell
cd
#Install SDRPlay API
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.15.2.run
#Change permission so the run file is executable
chmod 755 ./SDRplay_RSP_API-Linux-3.14.0.run
#Execute the API installer (follow the prompts)
./SDRplay_RSP_API-Linux-3.14.0.run
```
### 4) Install SoapySDR.
```shell
mkdir ~/Dev
cd ~/Dev
sudo rm -rf SoapySDR
git clone https://github.com/pothosware/SoapySDR.git
cd SoapySDR
mkdir build
cd build
cmake ..
make -j`nproc`
sudo make install -j`nproc`
sudo ldconfig #needed on debian systems
SoapySDRUtil --info
```
### 5a) Install Soapy SDR plugin for *SDRPlay*.
```shell
cd ~/Dev
sudo rm -rf SoapySDRPlay
git clone https://github.com/pothosware/SoapySDRPlay.git
cd SoapySDRPlay
mkdir build
cd build
cmake ..
sudo make install
sudo ldconfig
cd
```

### 5b) Install Soapy SDR plugin for *RTL-SDR*.
```shell
cd ~/Dev
sudo rm -rf SoapyRTLSDR
git clone https://github.com/pothosware/SoapyRTLSDR.git
cd SoapyRTLSDR
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig
cd
```

### 6) Plug in the RSP1(A/B)/RTL-SDR to the computer / Pi, then check the radio is picked up by the computer.
```
SoapySDRUtil -info
```
then for RSP1(A/B)
```
SoapySDRUtil --probe=driver=sdrplay
```
or for RTL-SDR
```
SoapySDRUtil --probe="driver=rtlsdr"
```

Trouble shooting see the relevant SoapySDR wiki page
[SDRPlay](https://github.com/pothosware/SoapySDRPlay3/wiki)/[RTL-SDR](https://github.com/pothosware/SoapyRTLSDR/wiki)

### 7) Install libcars.
```shell
sudo apt-get install zlib1g-dev
sudo apt-get install libxml2-dev
sudo apt-get install libjansson-dev
wget https://github.com/szpajder/libacars/archive/refs/tags/v2.2.0.zip
sudo apt install unzip
unzip v2.2.0.zip
cd libacars-2.2.0
mkdir build
cd build
cmake ../
make
sudo make install
sudo ldconfig
cd
```

### 8) Install optional stats and messaging tools for HFDL.
```shell
#Install libsqlite3
sudo apt install libsqlite3-dev
sudo ldconfig
```
```shell
#Install Etsy statsd-c-client
git clone https://github.com/romanbsd/statsd-c-client.git
cd statsd-c-client
make
sudo make install
sudo ldconfig
cd
```
```
#Install ZeroMQ
sudo apt install libzmq3-dev
```

### 9) Install dumpHFDL
```shell
cd
git clone https://github.com/szpajder/dumphfdl.git
cd dumphfdl
mkdir build
cd build
cmake ../
make
sudo make install
cd
```

### 10) Launch dumpHFDL on startup
Go to the dumphfdl build directory and install to /usr/local/bin

```shell
cd dumphfdl/build
sudo make install
cd ..
sudo cp etc/dumphfdl.service /etc/systemd/system/
sudo cp etc/dumphfdl /etc/default
```

You will need to create 4 total dumphfdl.service files - one for each SDR you are running (assuming you are running 4 SDR's) with a different set of dumphfdl frequencies.

Copy the contents of dumphfdl.service (example below)
```bash
[Unit]
Description=HFDL decoder
Documentation=https://github.com/szpajder/dumphfdl/blob/master/README.md
Wants=network.target
After=network.target

[Service]
Type=simple
EnvironmentFile=/etc/default/dumphfdl
# If you don't want to run the program as root, then uncomment
# the following line and put a desired user name in it.
# Note that the user must have access to the SDR device.
# User=pi
ExecStart=/usr/local/bin/dumphfdl $DUMPHFDL_OPTIONS
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

into 4 seperate dumphfdl.service files 

```shell
sudo nano /etc/systemd/system/dumphfdl1.service
sudo nano /etc/systemd/system/dumphfdl2.service
sudo nano /etc/systemd/system/dumphfdl3.service
sudo nano /etc/systemd/system/dumphfdl4.service
```

Change the line `EnvironmentFile=/etc/default/dumphfdl` to match the service number. For example for dumphfdl2.service this would change to `EnvironmentFile=/etc/default/dumphfdl2` 
Save and exit the file. 
Do for each of the created service files.

Delete the original dumphfdl.service file 
```bash
cd /etc/systemd/system/
#Then
sudo rm dumphfdl.service
cd
```

Setup timer service files to start each dumphfdl service file in sequence; 15 seconds apart on boot, after 30 seconds so that each SDR is loaded in sequence to avoid connection errors with VRS/tar1090.
Create and add the following to each timer service file. 

#### Timer 1 
```bash
sudo nano /etc/systemd/system/dumphfdl1.timer
```

Then copy and paste the text below into the file and save

```bash
[Unit]
Description=Timer for dumphfdl1 service

[Timer]
Unit=dumphfdl1.service
OnBootSec=30sec

[Install]
WantedBy=timers.target
EOF
```

#### Timer 2 
```bash
sudo nano /etc/systemd/system/dumphfdl2.timer
```

Then copy and paste the text below into the file and save

```bash
[Unit]
Description=Timer for dumphfdl2 service

[Timer]
Unit=dumphfdl2.service
OnBootSec=45sec

[Install]
WantedBy=timers.target
EOF
```

#### Timer 3 
```bash
sudo nano /etc/systemd/system/dumphfdl3.timer
```

Then copy and paste the text below into the file and save

```bash
[Unit]
Description=Timer for dumphfdl3 service

[Timer]
Unit=dumphfdl3.service
OnBootSec=60sec

[Install]
WantedBy=timers.target
EOF
```

#### Timer 4 
```bash
sudo nano /etc/systemd/system/dumphfdl4.timer
```

Then copy and paste the text below into the file and save

```bash
[Unit]
Description=Timer for dumphfdl4 service

[Timer]
Unit=dumphfdl1.service
OnBootSec=75sec

[Install]
WantedBy=timers.target
EOF
```

#### Options File (required)

dumpHFDL options files will need to be created for each dumpHFDL want to run. Put your preferred options in the files. Examples below for SDRPlay devices.
Create the file and as a minimum you need to specify the soapysdr driver, serial number (different for each file - one SDR per dumpHFDL instance), sample rate + frequencies, and at least one output option.
Other options to add to the options can be extracted from the [dumpHFDL readme](https://github.com/szpajder/dumphfdl?tab=readme-ov-file#basic-usage)

##### dumphfd1
 An example of SDRPlay device with serial number of wwwwwwwwww, sample rate of 2000000 covering frequencies in the band 5 and band 6 range, outputting basesation data to a specified IP address on port 50001.

```bash
sudo nano /etc/default/dumphfdl1
```

Copy this into the file.

```bash
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=wwwwwwwwwwww --freq-as-squawk --sample-rate 2000000 5451 5463 5502 5508 5514 5523 5529 5538 5541 5544 5547 5583 5589 5622 5652 5655 5720 6529 6532 6535 6559 6565 6589 6596 6619 6628 6646 6652 6661 6712 --output decoded:basestation:tcp:address=192.168.x.yyy,port=50001
```

##### dumphfd2
An example of SDRPlay device with serial number of xxxxxxxxxx, sample rate of 2000000 covering frequencies in the band 8 and band 10 range, outputting basesation data to a specified IP address on port 50002, also outputting ZMQ data in server mode on port 20558 (Can be used to feed hfdl message into [ACARSHub](https://github.com/sdr-enthusiasts/docker-acarshub?tab=readme-ov-file#sdr-enthusiastsacarshub))

```bash
sudo nano /etc/default/dumphfdl2
```
Copy this into the file.

```bash
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=xxxxxxxxxx --freq-as-squawk --sample-rate 2000000 8825 8831 8834 8843 8885 8886 8894 8912 8921 8927 8936 8939 8942 8948 8957 8977 10027 10060 10063 10066 10075 10081 10084 10087 10093 --output decoded:basestation:tcp:address=192.168.x.yyy,port=50002 --output decoded:json:zmq:mode=server,endpoint=tcp://0.0.0.0:20558
```

##### dumphfd3
An example of SDRPlay device with serial number of yyyyyyyyyy, sample rate of 6000000 covering frequencies in the band 11, band 13 and band 15 range, outputting basesation data to a specified IP address on port 50003 with the station name set as XX_HFDL3_11-15

```bash
sudo nano /etc/default/dumphfdl3
```
Copy this into the file

```bash
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=yyyyyyyyyy --freq-as-squawk --sample-rate 6000000 11184 11288 11306 11312 11315 11318 11321 11327 11348 11354 11384 11387 13264 13270 13276 13303 13312 13315 13321 13324 13342 13351 13354 15025 --output decoded:basestation:tcp:address=192.168.x.yyy,port=50003 --station-id XX_HFDL3_11-15
```

##### dumphfd4
An example of SDRPlay device with serial number of zzzzzzzzzz, sample rate of 6000000 covering frequencies in the band 17 and band 21 range, outputting basesation data to a specified IP address on port 50004, also outputting ZMQ data in server mode on port 20559 (Can be used to feed hfdl message into [ACARSHub](https://github.com/sdr-enthusiasts/docker-acarshub?tab=readme-ov-file#sdr-enthusiastsacarshub)) and messages to a text file called 17to21msg.log at the stated location, set to locate and also save the system table at the given location, access a BaseStation.sqb to extract aircraft data and populate the message with this data, with the station name set as XX_HFDL3_17-21.
```bash
sudo nano /etc/default/dumphfdl4
```

Copy this into the file

```bash
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=zzzzzzzzzz --freq-as-squawk --sample-rate 6000000 17901 17912 17916 17919 17922 17928 17934 17952 17958 17967 17985 21928 21931 21934 21937 21940 21946 21949 21955 21973 21982 21988 21990 21997 --output decoded:basestation:tcp:address=192.168.x.yyy,port=50004 --output decoded:json:zmq:mode=server,endpoint=tcp://0.0.0.0:20559 --output decoded:text:file:path=/home/pi/hfdl4logs/17to21msg.log,rotate=daily --system-table /home/pi/dumphfdl/etc/systable.conf --system-table-save /home/pi/dumphfdl/etc/systable.conf --bs-db /home/pi/BaseStation.sqb --ac-details verbose --station-id XX_HFDL4_17-21"

```

Once the service files, timer services and options files are created do the following

Reload systemd configuration
```shell
sudo systemctl daemon-reload
```


Start the service timer
```shell
sudo systemctl start dumphfdl1.timer
sudo systemctl start dumphfdl2.timer
sudo systemctl start dumphfdl3.timer
sudo systemctl start dumphfdl4.timer
```



Enable the service timer
```shell
sudo systemctl enable dumphfdl1.timer
sudo systemctl enable dumphfdl2.timer
sudo systemctl enable dumphfdl3.timer
sudo systemctl enable dumphfdl4.timer
```


Reload systemd configuration again
```shell
sudo systemctl daemon-reload
```

Reboot the system
```shell
sudo reboot now
```



Verify if it's running after a reboot of the system and waiting at least 90 seconds:
```
systemctl status dumphfdl1
```

```
systemctl status dumphfdl2
```

```
systemctl status dumphfdl3
```

```
systemctl status dumphfdl4
```
CTRL-Z to get back to prompt.

Debug as required.

### 11) Notes

1) When sending SBS3/Basestation messages to an `IP:port` make sure that `IP:port` is active before starting up dumpHFDL otherwise the output will fail. If it was the only output specified dumpHFDL will shutdown.
2) If using an RTL-SDR replace `driver=sdrplay` with `driver=rtlsdr`
3) Sample rates for RTL-SDR are stable up to 2560000, but can go up to 3200000 - Y.M.M.V.
4) DumpHFDL won't work with Airspy devices apart from the speicific ones for HF or ones with a down converter.
