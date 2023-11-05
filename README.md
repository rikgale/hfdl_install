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
