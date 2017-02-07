---
layout: post
title:  "Changing your MAC address for fun and profit"
date:   2017-02-07 23:22:15
comments: true
categories: shell
---

I'm currently in a cafe in Cape Town that only gives free WiFi for one hour, and then makes you pay if you want to continue to use it. You can get around this paywall by simply changing your network interface's MAC address. When you reconnect to the network, it will think you are a new customer and give you an additional hour for free.

A MAC address is a unique identifier that network interfaces use to communicate with each other. It's _kinda_ like an IP address, but whereas IP addresses are addresses used to identify hosts and do packet forwarding on the network layer, MAC addresses are used for communication on the data link layer, which is one layer lower on the OSI stack than the network layer.

Some reasons you might want to change your MAC address include bypassing WiFi paywalls, and for privacy reasons so that routers can't track your computer/network interface between visits. Just incase you forgot your tinfoil hat at home.

Anyways, here's a simple script you can use to automate the process:

```bash
#!/bin/sh
#
# Generates a random mac address and configures en0
###################################################

# Evals a function and exits the script if any error occurred
function run() {
  cmd_output=$(eval $1)
  return_value=$?
  if [ $return_value != 0 ]; then
    echo "Command $1 failed"
    exit -1
  fi
  return $return_value
}

MAC=$(openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//')

run "sudo /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -z"
run "sudo ifconfig en0 ether $MAC"
run "networksetup -detectnewhardware"
```
