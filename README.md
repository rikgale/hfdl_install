# hfdl_install
Instructions for installing soapysdr and dumphdfl. Written for debian based systems. Tested on Raspberry Pi OS and Ubuntu

1) Ensure system is fully up-to-date
```shell
sudo apt update && sudo apt upgrade -y
```

2) Install dumpHFDL dependancies
```shell
sudo apt install build-essential cmake pkg-config libglib2.0-dev libconfig++-dev libliquid-dev libfftw3-dev git
```

3) Install libcars
```shell
sudo apt-get install zlib1g-dev
sudo apt-get install libxml2-dev
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

4) Install SDRPlay API (x64 systems - **not Raspberry Pi/ARM based systems** - see below)
```shell
#Install SDRPlay API
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.07.1.run
#Change permission so the run file is executable
chmod 755 ./SDRplay_RSP_API-Linux-3.07.1.run
#Execute the API installer (follow the prompts)
./SDRplay_RSP_API-Linux-3.07.1.run
```
For Raspberry Pi and other ARM based systems
```shell
#Install SDRPlay API
wget https://www.sdrplay.com/software/SDRplay_RSP_API-ARM64-3.07.1.run
#Change permission so the run file is executable
chmod 755 ./SDRplay_RSP_API-ARM64-3.07.1.run
#Execute the API installer (follow the prompts)
./SDRplay_RSP_API-ARM64-3.07.1.run
```
