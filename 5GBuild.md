# 5G Build Guide

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

### srsRAN_Project

prerequisite: sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev

1. git clone https://github.com/srsRAN/srsRAN_Project.git
2. cd srsRAN_Project
3. mkdir build
4. cd build
5. cmake ../
6. make -j $(nproc)
7. sudo make install
8. sudo ldconfig

More details on srsRAN_Project installation can be found at: https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html#manual-installation

### Open5GS

prerequisite: sudo apt install python3-pip python3-setuptools python3-wheel ninja-build build-essential flex bison git cmake libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libidn11-dev libmongoc-dev libbson-dev libyaml-dev libnghttp2-dev libmicrohttpd-dev libcurl4-gnutls-dev libnghttp2-dev libtins-dev libtalloc-dev meson

#### Getting MongoDB

MongoDB is used as database for NRF/PCF/UDR and PCRF/HSS.
1. sudo apt update
2. sudo apt install -y mongodb-org
3. sudo systemctl start mongod (if '/usr/bin/mongod' is not running)
4. sudo systemctl enable mongod (ensure to automatically start it on system boot)

#### Building Open5GS

1. git clone https://github.com/open5gs/open5gs
2. cd open5gs
3. meson build --prefix=`pwd`/install
4. ninja -C build
5. cd build
6. ninja install

#### Setting up TUN device (not persistent after rebooting)

Create the TUN device with the interface name ogstun.
1. cd open5gs
2. sudo ./misc/netconf.sh

#### Building the WebUI of Open5GS

Node.js is required to build WebUI of Open5GS
1. sudo apt install curl
2. curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
3. sudo apt install nodejs

Install the dependencies to run WebUI
1. cd webui
2. npm ci

The WebUI runs as an npm script.
1. npm run dev

#### Register Subscriber Information

Connect to http://127.0.0.1:3000 and login with admin account. (Username : admin; Password : 1423)

To add subscriber information, you can do WebUI operations in the following order:

1. Go to Subscriber Menu.
2. Click + Button to add a new subscriber.
3. Fill the IMSI, security context(K, OPc, AMF), and APN of the subscriber.
4. Click SAVE Button

Write the above parameters into the programmable USIM card through GRSIMWrite or [pySim](https://github.com/osmocom/pysim).

#### Adding a route for the UE to have WAN connectivity

In order to bridge between the PGWU/UPF and WAN (Internet), you must enable IP forwarding and add a NAT rule to your IP Tables.

To enable forwarding and add the NAT rule, enter
1. sudo sysctl -w net.ipv4.ip_forward=1
2. sudo sysctl -w net.ipv6.conf.all.forwarding=1
3. sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
4. sudo ip6tables -t nat -A POSTROUTING -s 2001:db8:cafe::/48 ! -o ogstun -j MASQUERADE

Configure the firewall correctly. Some operating systems (Ubuntu) by default enable firewall rules to block traffic.
1. sudo ufw disable

More details on Open5GS installation can be found at: https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/

## Config Open5GS and gNB
More details can be found at: https://docs.srsran.com/projects/project/en/latest/tutorials/source/cotsUE/source/index.html#configuration