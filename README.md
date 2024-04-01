# hfdl_install
Instructions for installing soapysdr with RSP1a + RTL (Airspy support to be added later) and dumphdfl. Written for debian based systems. Tested on Raspberry Pi OS and Ubuntu.
**Do not** plug in your SDR until instructed by the instructions below. (Incomplete at this time)
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
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.14.0.run
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
make
sudo make install
sudo ldconfig
cd
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

### 6) Plug in the RSP1a/RTL-SDR to the computer / Pi, then check the radio is picked up by the computer.
```
SoapySDRUtil -info
```
then for RSP1a
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

### Other stuff to sort and organise

```#Launch dumpHFDL on startup: go to the dumphfdl build directory and install to /usr/local/bin

cd dumphfdl/build
sudo make install
cd ..
sudo cp etc/dumphfdl.service /etc/systemd/system/
sudo cp etc/dumphfdl /etc/default



#You will need to create 4 total dumphfdl.service files- one for each SDR you are running (assuming you are running 4 SDR's) with a different set of dumphfdl frequencies. 
#In each of the dumphfdl.service files copy the original service file contents by using winSCP to open the original "dumphfdl.service" and copy its contents.
#Once copied use Putty to open each service file listed below and paste the copied contents from the original "dumphfdl.service" file in the previous step into the files listed below. 
#Edit the copied contents to match the service name of each dumphfdl.service; i.e. dumphfdl1.service, dumphfdl2.service, etc... 

sudo nano /etc/systemd/system/dumphfdl1.service
sudo nano /etc/systemd/system/dumphfdl2.service
sudo nano /etc/systemd/system/dumphfdl3.service
sudo nano /etc/systemd/system/dumphfdl4.service



#We then need to delete the original dumphfdl.service file 
cd /etc/systemd/system/
#Then
sudo rm dumphfdl.service



#We then need to setup timer service files to start each dumphfdl service file in sequence; 15 seconds apart on boot, so that each SDR is loaded in sequence to avoid connection errors with VRS.
#Create and add the following to each timer service file, make sure to modify the service name for each dumphfdl service; i.e. dumphfdl1.service, dumphfdl2.service, etc... 



#dumphfdl1
sudo nano /etc/systemd/system/dumphfdl1.timer

#Then copy and paste the text below into the file and save

[Unit]
Description=Timer for dumphfdl1 service

[Timer]
Unit=dumphfdl1.service
OnBootSec=15sec

[Install]
WantedBy=timers.target
EOF



#dumphfdl2
sudo nano /etc/systemd/system/dumphfdl2.timer

#Then copy and paste the text below into the file and save

[Unit]
Description=Timer for dumphfdl2 service

[Timer]
Unit=dumphfdl2.service
OnBootSec=30sec

[Install]
WantedBy=timers.target
EOF



#dumphfdl3
sudo nano /etc/systemd/system/dumphfdl3.timer

#Then copy and paste the text below into the file and save

[Unit]
Description=Timer for dumphfdl3 service

[Timer]
Unit=dumphfdl3.service
OnBootSec=45sec

[Install]
WantedBy=timers.target
EOF



#dumphfdl4
sudo nano /etc/systemd/system/dumphfdl4.timer

#Then copy and paste the text below into the file and save

[Unit]
Description=Timer for dumphfdl4 service

[Timer]
Unit=dumphfdl1.service
OnBootSec=60sec

[Install]
WantedBy=timers.target
EOF



#Edit /etc/default and uncomment the DUMPHFDL_OPTIONS= line and put your preferred dumphfdl options there.
#You need to copy each dumphfdl config file and rename it- each file should have different frequencies, different serial numbers (the one that matches the SDR you are using), and the IP address of the computer running VRS.


sudo nano /etc/default/dumphfdl1
#Copy this into the file
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=2239001699 --freq-as-squawk --sample-rate 2000000 5451 5463 5502 5508 5514 5523 5529 5538 5541 5544 5547 5583 5589 5622 5652 5655 5720 6529 6532 6535 6559 6565 6589 6596 6619 6628 6646 6652 6661 6712 --output decoded:basestation:tcp:address=192.168.1.155,port=44441


sudo nano /etc/default/dumphfdl2
#Copy this into the file
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=2239001799 --freq-as-squawk --sample-rate 2000000 8825 8831 8834 8843 8885 8886 8894 8912 8921 8927 8936 8939 8942 8948 8957 8977 10027 10060 10063 10066 10075 10081 10084 10087 10093 --output decoded:basestation:tcp:address=192.168.1.155,port=44442


sudo nano /etc/default/dumphfdl3
#Copy this into the file
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=2239002799 --freq-as-squawk --sample-rate 6000000 11184 11288 11306 11312 11315 11318 11321 11327 11348 11354 11384 11387 13264 13270 13276 13303 13312 13315 13321 13324 13342 13351 13354 15025 --output decoded:basestation:tcp:address=192.168.1.155,port=44443


sudo nano /etc/default/dumphfdl4
#Copy this into the file
DUMPHFDL_OPTIONS="--soapysdr driver=sdrplay,serial=2239002A99 --freq-as-squawk --sample-rate 6000000 17901 17912 17916 17919 17922 17928 17934 17952 17958 17967 17985 21928 21931 21934 21937 21940 21946 21949 21955 21973 21982 21988 21990 21997 --output decoded:basestation:tcp:address=192.168.1.155,port=44444



#Reload systemd configuration
sudo systemctl daemon-reload



#Start the service timer
sudo systemctl start dumphfdl1.timer
sudo systemctl start dumphfdl2.timer
sudo systemctl start dumphfdl3.timer
sudo systemctl start dumphfdl4.timer



#Enable the service timer
sudo systemctl enable dumphfdl1.timer
sudo systemctl enable dumphfdl2.timer
sudo systemctl enable dumphfdl3.timer
sudo systemctl enable dumphfdl4.timer



#Reload systemd configuration
sudo systemctl daemon-reload



#Verify if it's running after a reboot of the system and waiting at least 1 minute:
systemctl status dumphfdl1
#CTRL-Z to get a prompt
systemctl status dumphfdl2
#CTRL-Z to get a prompt
systemctl status dumphfdl3
#CTRL-Z to get a prompt
systemctl status dumphfdl4
#CTRL-Z to get a prompt```
