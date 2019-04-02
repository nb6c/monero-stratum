# monero-stratum

High performance CryptoNote mining stratum with Web-interface written in Golang.

[![Go Report Card](https://goreportcard.com/badge/github.com/sammy007/monero-stratum)](https://goreportcard.com/report/github.com/sammy007/monero-stratum)
[![CircleCI](https://circleci.com/gh/sammy007/monero-stratum.svg?style=svg)](https://circleci.com/gh/sammy007/monero-stratum)

**Stratum feature list:**

* Be your own pool
* Rigs availability monitoring
* Keep track of accepts, rejects, blocks stats
* Easy detection of sick rigs
* Daemon failover list
* Concurrent shares processing
* Beautiful Web-interface

![](https://cdn.pbrd.co/images/jRU3qJj83.png)

## Installation

Dependencies:

  * go-1.6
  * Everything required to build Monero
  * Monero >= **v0.14.0.2** (sometimes `master` branch required)

### Linux

Use Ubuntu 16.04 LTS.

Compile Monero source (with shared libraries option):

    sudo apt-get install git cmake build-essential libssl-dev pkg-config libboost-all-dev
    git clone https://github.com/monero-project/monero.git
    cd monero
    git checkout tags/v0.14.0.2 -b v0.14.0.2
    cmake -DBUILD_SHARED_LIBS=1 .
    make

Install Golang and required packages:

    sudo apt-get install golang

Clone stratum:

    git clone https://github.com/nb6c/monero-stratum.git
    cd monero-stratum

Build stratum:

    CURUSER=$(whoami)
    MONERO_DIR=/home/$CURUSER/monero cmake .
    make

`MONERO_DIR=/path/to/monero` is optional, not needed if both `monero` and `monero-stratum` is in the same directory like `/opt/src/`. By default make will search for monero libraries in `../monero`. You can just run `cmake .`.

When you get an error like 'package context: unrecognized import path "context" (import path does not begin with hostname)', see https://github.com/golang/dep/issues/1985 .

    sudo apt-get purge golang*
    sudo rm -rf /usr/lib/go-1.6/ /usr/lib/go-1.6/src/ /usr/lib/go-1.6/src/runtime/ /usr/lib/go-1.6/src/runtime/race
    curl -O https://storage.googleapis.com/golang/go1.11.1.linux-amd64.tar.gz
    sudo tar -C /usr/local -xzf go1.11.1.linux-amd64.tar.gz
    mkdir -p ~/go; echo "export GOPATH=$HOME/go" >> ~/.bashrc
    echo "export PATH=$PATH:$HOME/go/bin:/usr/local/go/bin" >> ~/.bashrc
    source ~/.bashrc


### Mac OS X

Compile Monero source:

    git clone https://github.com/monero-project/monero.git
    cd monero
    git checkout tags/v0.14.0.0 -b v0.14.0.0
    cmake .
    make

Install Golang and required packages:

    brew update && brew install go

Clone stratum:

    git clone https://github.com/sammy007/monero-stratum.git
    cd monero-stratum

Build stratum:

    MONERO_DIR=/path/to/monero cmake .
    make

### Running Stratum

    ./build/bin/monero-stratum config.json

If you need to bind to privileged ports and don't want to run from `root`:

    sudo apt-get install libcap2-bin
    sudo setcap 'cap_net_bind_service=+ep' /path/to/monero-stratum
    
To run monero-stratum as a service :

    CURUSER=$(whoami)
    sed -e "s/__USER__/$CURUSER/g" monero-stratum.service | sudo tee /lib/systemd/system/monero-stratum.service >/dev/null

    sudo systemctl daemon-reload
    sudo systemctl enable monero-stratum
    sudo systemctl start monero-stratum
    
    To check status :
        sudo systemctl status monero-stratum
    
    To stop service :
        sudo systemctl stop monero-stratum
    

## Configuration

Configuration is self-describing, just copy *config.example.json* to *config.json* and run stratum with path to config file as 1st argument.

```javascript
{
  // Address for block rewards
  "address": "YOUR-ADDRESS-NOT-EXCHANGE",
  // Don't validate address
  "bypassAddressValidation": true,
  // Don't validate shares
  "bypassShareValidation": true,

  "threads": 2,

  "estimationWindow": "15m",
  "luckWindow": "24h",
  "largeLuckWindow": "72h",

  // Interval to poll daemon for new jobs
  "blockRefreshInterval": "1s",

  "stratum": {
    // Socket timeout
    "timeout": "15m",

    "listen": [
      {
        "host": "0.0.0.0",
        "port": 1111,
        "diff": 5000,
        "maxConn": 32768
      },
      {
        "host": "0.0.0.0",
        "port": 3333,
        "diff": 10000,
        "maxConn": 32768
      }
    ]
  },

  "frontend": {
    "enabled": true,
    "listen": "0.0.0.0:8082",
    "login": "admin",
    "password": "",
    "hideIP": false
  },

  "upstreamCheckInterval": "5s",

  "upstream": [
    {
      "name": "Main",
      "host": "127.0.0.1",
      "port": 18081,
      "timeout": "10s"
    }
  ],
  
  "statsDir": "stats",
  "statsInterval": 5
}
```

You must use `anything.WorkerID` as username in your miner. Either disable address validation or use `<address>.WorkerID` as username. If there is no workerID specified your rig stats will be merged under `0` worker. If mining software contains dev fee rounds its stats will usually appear under `0` worker. This stratum acts like your own pool, the only exception is that you will get rewarded only after block found, shares only used for stats.

## Stats
Blocks found and round progress will be saved across runs in `$statsDir/stats.json` if you have enabled them by setting 
`statsDir` in your `config.json`. You can also choose the interval (seconds) that stats are collected with by setting 
`statsInterval`. The default is 5 seconds. 

### Donations

**XMR**: `47v4BWeUPFrM9YkYRYk2pkS9CubAPEc7BJjNjg4FvF66Y2oVrTAaBjDZhmFzAXgqCNRvBH2gupQ2gNag2FkP983ZMptvUWG`

![](https://cdn.pbrd.co/images/GP5tI1D.png)

Highly appreciated.

### License

Released under the GNU General Public License v2.

http://www.gnu.org/licenses/gpl-2.0.html
