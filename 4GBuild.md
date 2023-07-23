# 4G Build Guide

## Baseline Hardware Requirements

- One PC with Ubuntu
- [One USRP B210](https://www.ettus.com/all-products/ub210-kit/)
- One Programmable USIM Card
- One Mobile Phone
- [One Card Reader](https://www.amazon.com/MCR3516-Reader-Writer-Programmable-Software/dp/B07PVSPLLH?language=en_US)

## Installation from Source

### UHD

prerequisite: sudo apt-get install autoconf automake build-essential ccache cmake cpufrequtils doxygen ethtool g++ git inetutils-tools libboost-all-dev libncurses5 libncurses5-dev libusb-1.0-0 libusb-1.0-0-dev libusb-dev python3-dev python3-mako python3-numpy python3-requests python3-scipy python3-setuptools python3-ruamel.yaml

1. git clone https://github.com/EttusResearch/uhd.git
2. cd \<uhd-repo-path\>/host
3. mkdir build
4. cd build
5. cmake ../
6. make
7. sudo make install
8. sudo ldconfig
9. sudo uhd_images_downloader

More details on UHD installation can be found at: https://files.ettus.com/manual/page_build_guide.html

### srsRAN_4G

prerequisite: sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev

1. git clone https://github.com/srsRAN/srsRAN_4G.git
2. cd srsRAN_4G
3. mkdir build
4. cd build
5. cmake ../
6. make
7. sudo make install
8. sudo ldconfig
9. sudo srsran_4g_install_configs.sh user

This install srsRAN 4G and also copies the default srsRAN 4G config files to ~/.config/srsran_4g.

More details on srsRAN_4G installation can be found at: https://docs.srsran.com/projects/4g/en/latest/general/source/1_installation.html#installation-from-source

## Config

### EPC

The following snippet shows where to change the MCC & MNC & apn values in the EPC config file (~/.config/srsran_4g/epc.conf):
```
[mme]
mme_code = 0x1a
mme_group = 0x0001
tac = 0x0007
mcc = 001
mnc = 01
mme_bind_addr = 127.0.1.100
apn = srsapn
```

### eNB

The following snippets show where to change the MCC & MNC & dl_earfcn values in the eNB config file (~/.config/srsran_4g/enb.conf):
```
[enb]
enb_id = 0x19B
mcc = 001
mnc = 01
```

```
[rf]
#dl_earfcn = 3350
tx_gain = 80
rx_gain = 40
```

### User DB

Add a user to ~/.config/srsran_4g/user_db.csv. An example is as follows. The important parameters are IMSI, Key and Opc.
```
ue3,mil,901700000020936,4933f9c5a83e5718c52e54066dc78dcf,opc,fc632f97bd249ce0d16ba79e6505d300,9000,0000000060f8,9,dynamic
```

### Write Card

Write the above parameters into the programmable USIM card through GRSIMWrite or [pySim](https://github.com/osmocom/pysim).

### Run Masquerading Script

To allow UE to connect to the internet via the EPC, the pre-configured masquerading script (srsRAN_4G/srsepc/srsepc_if_masq.sh) must be run.

1. sudo ./srsepc_if_masq.sh \<interface\>

More details on configuring srsRAN_4G can be found at:
https://docs.srsran.com/projects/4g/en/latest/app_notes/source/cots_ue/source/index.html#connecting-a-cots-ue-to-srsran-4g

## Connecting a mobile phone to 4G

Initialise the EPC by running:
1. sudo srsepc

Initialise the eNB by running:
1. sudo srsenb

Insert the USIM card into the mobile phone and choose 4G mode. The phone should then automatically connect to the 4G network.
