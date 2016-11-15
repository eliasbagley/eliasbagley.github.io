---
layout: post
title:  "Downloading Huge Files in Chunks, For Use on Unreliable Networks"
date:   2016-11-11 23:22:15
comments: true
categories: shell
---

I'm currently in Rio de Janeiro, trying to download Xcode 8 from a slow connection in a coffee shop. The download makes it to about 200 Mb out of 4 Gb, and the connection gets interruped and I have to start again. There's probably an intelligent download manager somewhere out there that can solve this, but I just ended up writing my own script using `curl` to fetch the file in chunks, and recombine it at the end with `cat`.

`curl` can't automatically fetch the Xcode file, since you have to be logged in to the Apple developer portal. To solve this, I logged in to the developer portal, found the download link, and opened the network tab in the chrome developer tools. You can right click on the logged network request for the xcode file, and choose "Copy as curl", to get the right request with the correct cookies already set.

I then wrote a script to modify the curl request slightly to request the file in chunks of 20 Mb, and combine them all together at the end.

You can also stop the script at anytime, and rerun it to continue where you left off.

Here's the script: you'll have to modify the curl commands to use your own session token that you copied from the chrome developer tools.

```bash
#!/bin/bash
# Author: Elias Bagley
#
# This script fetches a file in parts, and combines them at the end
###################################################################

#automatically fetch the session cookie using apple email and password
#Take the file name in on the command line
#make sure file extension can be sorted, ie out.001 instead of out.1
#cleanup the part files after a successful signature


URL="http://adcdownload.apple.com/Developer_Tools/Xcode_8.1/Xcode_8.1.xip"
HEADER_1="Accept-Encoding: gzip, deflate, sdch"
HEADER_2="Accept-Language: en-US,en;q=0.8,pt;q=0.6"
HEADER_3="Upgrade-Insecure-Requests: 1"
HEADER_4="User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36"
HEADER_5="Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"
HEADER_7="Connection: keep-alive"

# Get the total file size
FILE_SIZE=$(curl -sI $URL -H $HEADER_1 -H $HEADER_2 -H $HEADER_3 -H $HEADER_4 -H $HEADER_5 -H '... copy from chrome ...' -H 'Connection: keep-alive' --compressed | grep Content-Length | awk '{print $2}' | tr -d $'\r')
echo "Total file size: $FILE_SIZE bytes"

# param order:
# $1 - filename
# $2 - lower byte range
# $3 - upper byte range (can be blank to fetch the rest of the file)
# note: repaste from curl
function fetch_part {
  curl --range $2-$3 -o $1 'http://adcdownload.apple.com/Developer_Tools/Xcode_8.1/Xcode_8.1.xip' -H 'Accept-Encoding: gzip, deflate, sdch' -H 'Accept-Language: en-US,en;q=0.8,pt;q=0.6' -H 'Upgrade-Insecure-Requests: 1' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H '... copy from chrome ...' -H 'Connection: keep-alive' --compressed
}

PART_SIZE=20000000 # size in bytes to download for each part

OUTPUT_FILENAME=$(basename $URL)
COUNT=0
LOWER_RANGE=0
UPPER_RANGE=$(($PART_SIZE-1))
PART_FILE=out.$COUNT
NUM_PARTS=$((FILE_SIZE/PART_SIZE))

# increments the global count variables for the main loop
function increment_count {
  COUNT=$(($COUNT+1))
  LOWER_RANGE=$(($LOWER_RANGE+$PART_SIZE))
  UPPER_RANGE=$(($UPPER_RANGE+$PART_SIZE))
  PART_FILE=out.$COUNT
}

# returns the file size in bytes of the first argument
function file_size {
  ls -l $1 | awk '{print $5}'
}

# loop as long as the upper range is less than the total
while [ $UPPER_RANGE -lt $FILE_SIZE ]
do
  echo "Fetching part $COUNT of $NUM_PARTS"

  # check to see if the part already exists - if so we can skip it
  if [ -f $PART_FILE ]; then
    # check to see if it has the full size
    SIZE=$(file_size $PART_FILE)
    echo "Part size: $SIZE"

    if [ $SIZE -lt $PART_SIZE ]; then
      # rm this file so it can be fully refetched
      rm $PART_FILE
      echo "Refetching $PART_FILE..."
    else
      echo "Part $COUNT already exist"
      increment_count
      continue
    fi

  fi

  #otherwise, fetch the part
  fetch_part $PART_FILE $LOWER_RANGE $UPPER_RANGE

  # update for next loop
  increment_count
done

#download the remainder till the end of the file
fetch_part $PART_FILE $LOWER_RANGE

#combine back into single file, by reverse time of creation
echo "Combining output..."
ls -1tr out.* | while read fn ; do cat "$fn" >> $OUTPUT_FILENAME; done

COMBINED_SIZE=$(file_size $OUTPUT_FILENAME)
if [ $COMBINED_SIZE -ne $FILE_SIZE ]; then
  echo "File sizes don't match! Something went wrong"
else
  echo "File sizes match!"
fi

# Check the signature of the xip archive
pkgutil --check-signature $OUTPUT_FILENAME
```
