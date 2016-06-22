---
layout: post
title:  "Finding Flakey Tests"
date:   2016-06-22 12:00:00
comments: true
categories: iOS
---

Sometimes I run across the case where I have a flakey test that fails only once in every several runs. This indicates that there is a race condition in the code or in my test. To guarantee that my fix for the race condition works, I would ideally like to run the single test multiple times in Xcode and only report success if all runs of the test pass, but this isn't currently possible. However, it is possible with a tool call [xctool][xctool-link] and a little bit of shell script:

```bash
#!/bin/sh

function run() {
  cmd_output=$(eval $1)
  return_value=$?
  if [ $return_value != 0 ]; then
    echo "Command $1 failed"
    exit -1
  fi
  return $return_value
}


for run in {1..10}
do
  echo "Running $run of 10"
  run "xctool -workspace SudoKit.xcworkspace -scheme SudoKitTests test -only SudoKitTests:StateMachineTests/test_StateMachineContainer_releasesUnusedStateMachines"
done

echo "All passed"

```

The `run` function will fail the script if any of the commands fail

[xctool-link]: https://github.com/facebook/xctool

