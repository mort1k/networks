== Steps to upgrade comdex chain ( This is to fix the oracle module in testnet)

. stop the comdex chain

* Checkout below tag to update the chain( this is noe time manual step, other upgades will be done using cosmovisor)

```shell
cd comdex
git fetch --tags
git checkout v0.0.2
```
* Install
```shell
make all
```
. Build the binary and copy the binary to ~/.comdex/cosmovisor/genesis/bin path.
. Further follow the cosmovisor setup guide.

== Setup Cosmovisor

1. Install the latest version of cosmovisor,using the below command:

    go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest

2. Copy the cosmovisor binary to Go path bin directory.

    cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor

== Set up folder structure

. Create the required directories

    mkdir -p ~/.comdex/cosmovisor
    mkdir -p ~/.comdex/cosmovisor/genesis
    mkdir -p ~/.comdex/cosmovisor/genesis/bin
    mkdir -p ~/.comdex/cosmovisor/upgrades

. Cosmovisor expects a certain folder structure:

    .
    ├── current -> genesis or upgrades/<name>
    ├── genesis
    │   └── bin
    │       └── $DAEMON_NAME
    └── upgrades
        └── <name>
            └── bin
                └── $DAEMON_NAME


. Set up the environment variables:

    echo "# Setup Cosmovisor" >> ~/.profile
    echo "export DAEMON_NAME=comdex" >> ~/.profile
    echo "export DAEMON_HOME=$HOME/.comdex" >> ~/.profile
    echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=false" >> ~/.profile
    echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.profile
    echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.profile
    echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.profile
    source ~/.profile

. Copy the running comdex binary into the cosmovisor/genesis folder

    cp $GOPATH/bin/comdex ~/.comdex/cosmovisor/genesis/bin

. To check your work, ensure the version of cosmovisor and comdex should be same:

    cosmovisor version
    comdex version

== Set Up Cosmovisor Service

Set up a service to allow cosmovisor to run in the background as well as restart automatically if it runs into any problems:

. create the service file

    sudo nano /etc/systemd/system/cosmovisor.service

.  Change the contents of the below to match your setup - cosmovisor is likely at ~/go/bin/cosmovisor regardless of which installation path you took above, but it's worth checking.

    [Unit]
    Description=cosmovisor
    After=network-online.target
    [Service]
    User=ubuntu
    ExecStart=/home/ubuntu/go/bin/cosmovisor start
    Restart=always
    RestartSec=3
    LimitNOFILE=4096
    Environment="DAEMON_NAME=comdex"
    Environment="DAEMON_HOME=/home/ubuntu/.comdex"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
    Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
    Environment="DAEMON_LOG_BUFFER_SIZE=512"
    [Install]
    WantedBy=multi-user.target

. Finally, enable the service and start it.

    sudo -S systemctl daemon-reload
    sudo -S systemctl enable cosmovisor

. STOP THE COMDEX CHAIN and START USING COSMOVISOR

    sudo systemctl start cosmovisor

. Check it is running using

    sudo systemctl status cosmovisor

. To view logs of the service

    journalctl -u cosmovisor -f

. To stop cosmovisor using

    sudo systemctl stop cosmovisor

== Update Comdex chain to v0.1.0 using Cosmovisor

    mkdir -p ~/.comdex/cosmovisor/upgrades/v0.1.0/bin
    cd $HOME/comdex
    git pull
    git checkout {{NewBranch}}
    make install
    make all

.   check the upgraded version:

    comdex version

.   copy the upgraded comdex version to cosmovisor upgraded directory

    cp $GOPATH/bin/comdex ~/.comdex/cosmovisor/upgrades/v0.1.0/bin
