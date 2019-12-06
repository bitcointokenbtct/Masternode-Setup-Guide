<img src="https://raw.githubusercontent.com/bitcointokenbtct/Official-Images/master/github-header3.jpg">

Bitcoin Token Masternode Setup Guide
==========================

## Introduction

This guide is for a single masternode, on a Ubuntu 18.04 64-bit server (VPS) running headless and will be controlled from the wallet on your local computer (Control wallet). The wallet on the the VPS will be referred to as the Remote wallet.

You will need your masternode server details for progressing through this guide including the public IP address.

### VPS Server Recommendations ###

[Digital Ocean](https://www.digitalocean.com/products/droplets/)

[Vultr](https://www.vultr.com/products/cloud-compute/#compute)

**For either of the above VPS services, the economical $5 plan will be sufficient for your masternode.**

---

First the basic requirements:

1. 10,000 BTCT
1. A main computer (your everyday computer) – This will run the Bitcoin Token Core control wallet, hold your collateral 10,000 BTCT and can be opened and closed without affecting the masternode.
1. Masternode Server (VPS Suggested – The wallet daemon that will be on 24/7).
1. A unique IP address for your VPS / Remote wallet.

**For security reasons, you’re are going to need a different IP for each masternode you plan to host.**

## Configuration

**Step 1:** Using the control wallet, enter the debug console `Tools > Debug console` and type the following command:

```
masternode genkey
```

*(This will be the masternode’s privkey variable. We will need this later…)*

**Step 2:** Using the control wallet, enter the following command:

```
getaccountaddress "chooseAnyNameForYourMasternode"
```

**Step 3:** Still in the control wallet, send 10,000 BTCT to the wallet address you generated in Step 2. 

Be 100% sure that you entered the address correctly. You can verify this when you paste the address into the **"Pay To:"** field, the label will auto populate with the name you chose. 

Also make sure this is exactly **10,000 BTCT**; No less, no more.

**Be absolutely sure the send to address is copied correctly and then check it again. We cannot help you if you send 10,000 BTCT to an incorrect address.**

Please allow at least 1 block confirmation to complete before moving on.

**Step 4:** Still in the control wallet, enter the command into the console:

```
masternode outputs
```

*This gets the proof of transaction of sending 10,000 BTCT*

**Step 5:** Still on the main computer, go into the BTCT data directory:

OS | Path to BTCT
------------ | -------------
Windows | `%Appdata%/Roaming/BTCT/`
macOS | `~/Library/Application\ Support/BTCT/`
Linux | `~/.btct/`

Find the masternode.conf file, edit it in your favorite text editor and add the following line to it:

```
<Name of Masternode(Use the name you entered earlier for simplicity)> <Unique VPS Public IP address>:42122 <The result of Step 1> <Result of Step 4> <The number after the long line in Step 4>
```

Example:

```
MN1 34.24.122.10:42122 894FPpFdbr7sr6Si4fdsfssjjapuFzAXwEVCrpPJubnrmU6aKzh c8f4865da57a68d0e6ddd84324dfd28cfbe0c901015b973e7331bb8ce018716999 1
```

Substitute with your own values and without the "<>"s.

Lastly, close the control wallet and open again to load the new configuration file.

## VPS Remote Wallet Install

Install the latest version of the Bitcoin Token Core wallet onto your masternode. The latest version can be found here: [Bitcoin Token Core Releases](https://github.com/bitcointokenbtct/Bitcoin-Token-Core/releases).

**Step 1:** Log in to your VPS via SSH:

```
cd ~
```

**Step 2:** From your home directory, download the latest version from the BTCT GitHub repository:

```
wget https://github.com/bitcointokenbtct/Bitcoin-Token-Core/releases/download/v1.1/btct-x86_64-linux-gnu.tar.gz
```

Always check the releases page for the latest version and update the URL to reflect the most current version.

**Step 3:** Unzip & Extract:

```
tar -zxvf btct-x86_64-linux-gnu.tar.gz
```

**Step 4:** Copy the files to the local bin. **Requires sudo**

```
sudo cp -R btct/bin/* /usr/local/bin/
```

**Step 5:** Note: If this is the first time running the wallet in the VPS, you’ll need to attempt to start the wallet:

```
btctd --daemon
```

**Step 6:** Stop the daemon after the blockchain downloads:

You can verify you have the entire blockchain by comparing the results of `btct-cli getinfo` with the [BTCT Block Explorer](http://explorer.bitcointoken.pw)

Once verified:

```
btct-cli stop
```

**Step 7:** Navigate to the btct data directory:

```
cd ~/.btct
```

**Step 8:** Open the configuration file by typing:

```
nano btct.conf
```

**Step 9:** Make the config look like this with your values:

```
rpcuser=long_random_username
rpcpassword=longer_random_password
rpcallowip=127.0.0.1
server=1
daemon=1
logtimestamps=1
maxconnections=256
masternode=1
externalip=Your VPS unique public ip address
masternodeprivkey=Result of Step 1
```

*Make sure to replace rpcuser and rpcpassword with your own.*

**Step 10:** Save and exit the file:

```
Ctr+x to exit and press Y to save changes and press enter to close
```

**Please be sure to have port 42122 open on your server firewall if applicable for your control wallet to be able start the masternode remotely.**

## Start the Masternode

**Step 1:** Navigate back to the Bitcoin Token VPS server:


**Step 2:** Start the wallet daemon:

```
btctd
```

**Step 3:** From the Control wallet debug console:

```
startmasternode alias false myalias
```

Where "myalias" is the name of your masternode alias (without brackets)

**The following should appear.**

```
"overall" : "Successfully started 1 masternodes, failed to start 0, total 1",
"detail" : [
  {
    "alias" : "<myalias>",
    "result" : "successful",
    "error" : ""
  }
]
```

**Step 4:** Back in the VPS (remote wallet), start the masternode:

```
btct-cli startmasternode local false
```

**Step 5:** Use the following command to check status:

```
btct-cli masternode status
```

You should see something like:

```
{
  "txhash": "c2f85b71a04d111a1ca337fbc3aed1168856e3365b4e846ac9b89a9908c15b1d",
  "outputidx": 1,
  "netaddr": "34.179.22.255:42122",
  "addr": "BTSAZ8JbaDDHTEKnMLd3exqv7n8VBKs9i7",
  "status": 4,
  "message": "Masternode successfully started"
}
```

If you see status Not capable masternode: Hot node, waiting for remote activation, you need to wait a bit longer for the blockchain to reach consensus. It's possible it may take 60 to 120 minutes before the activation can be done. You can also try restarting the VPS wallet `btct-cli stop` and then `btctd` and trying the `btct-cli startmasternode local false` command again.

Bitcoin Token Masternode Setup is Complete!


## Tearing down a Masternode

1. `btct-cli stop` from the masternode to stop the wallet.
1. Then from your control wallet, edit your masternode.conf, delete the MN1 masternode line entry.
1. Restart the control wallet.
1. Your 10,000 BTCT will now be unlocked.
