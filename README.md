# hfdl_install
Instructions for installing soapysdr (with RSP1a + RTL. Airspy to be added later) and dumphdfl. Written for debian based systems. Tested on Raspberry Pi OS and Ubuntu.
**Do not** plug in your SDR until instructed by the instructions below. (Incomplete at this time)
Credit to Alex KD6VPH for putting the initial iteration together.

### 1) Ensure system is fully up-to-date.
```shell
sudo apt update && sudo apt upgrade -y
```

### 2) Install dumpHFDL and other related (RTL-SDR) dependancies.
```shell
sudo apt install build-essential cmake pkg-config libglib2.0-dev libconfig++-dev libliquid-dev libfftw3-dev rtl-sdr librtlsdr-dev git
```

### 3) Install SDRPlay API  (x64 systems - **not Raspberry Pi/ARM based systems** - see below).
```shell
cd
#Install SDRPlay API
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.07.1.run
#Change permission so the run file is executable
chmod 755 ./SDRplay_RSP_API-Linux-3.07.1.run
#Execute the API installer (follow the prompts)
./SDRplay_RSP_API-Linux-3.07.1.run
```
  For Raspberry Pi and other ARM based systems (Bookworm - to accept the licence agreement scroll to the end and push q, this should bring up the prompt to accept the agreement)
```shell
cd
#Install SDRPlay API
wget https://www.sdrplay.com/software/SDRplay_RSP_API-ARM64-3.07.1.run
#Change permission so the run file is executable
chmod 755 ./SDRplay_RSP_API-ARM64-3.07.1.run
#Execute the API installer (follow the prompts)
./SDRplay_RSP_API-ARM64-3.07.1.run
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

### 6) Plug in the RSP1a to the computer / Pi, then check the radio is picked up by the computer.
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

