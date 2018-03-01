---
title: Compiling xmr-stak for Ubuntu 14.04
date: 2018-03-01 16:50:47
tags:
- Crypto
- Ubuntu

autoThumbnailImage: yes
coverImage: ./mexico-monument.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: mexico-monument.jpg
thumbnailImagePosition: right
---

Recently i found a few old machines I had that were running Ubuntu 14.04, rather than getting rid of them I figured that I could put them to use by setting them to mine CPU minable Cryptocoins.
Coins like Monero, Aeon, Electroneum, Turtlecoin and a number of other coins can be mined using a project called [xmr-stak](https://github.com/fireice-uk/xmr-stak).

xmr-stak has an install guide for new and older versions of ubuntu but include using your nvidia or amd cards as well as part of the compilation process. It also includes a 2% dev fee.
The following script is adapted from [this](https://github.com/fireice-uk/xmr-stak/blob/master/doc/compile_Linux.md) link but removes the dev fee and also compiles for your machines CPU only.

```bash
# Ubuntu 14.04 xmr-stak CPU only with no donation
# adapted from https://github.com/fireice-uk/xmr-stak/blob/master/doc/compile_Linux.md

sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install -y gcc-5 g++-5 make libmicrohttpd-dev libssl-dev libhwloc-dev
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 1 --slave /usr/bin/g++ g++ /usr/bin/g++-5
curl -L http://www.cmake.org/files/v3.4/cmake-3.4.1.tar.gz | tar -xvzf - -C /tmp/
cd /tmp/cmake-3.4.1/ && ./configure && make && sudo make install && cd -
sudo update-alternatives --install /usr/bin/cmake cmake /usr/local/bin/cmake 1 --force
git clone https://github.com/fireice-uk/xmr-stak.git
mkdir xmr-stak/build
cd xmr-stak/build
echo '#pragma once\nconstexpr double fDevDonationLevel = 0.0 / 100.0;' > ./xmr-stak/donate-level.hpp
cmake .. -DCUDA_ENABLE=OFF -DOpenCL_ENABLE=OFF
make install
cd bin
```

Once you've run this just run `./xmr-stak` and begin configuring your miner!

<!-- more -->