# Xcode Server experiments

The goal of this repository is just to test Xcode server and provides some receipts for the following usecases :
- [x] [Cocoapods](#Cocoapods)
- [x] KIF
- [x] build a specific branch
- [ ] get results as variables after a build
- [ ] post notification on Slack
- [ ] build pull request almost automaticaly
- [ ] upload to testflight/itunes connect/hockey app automaticaly
- [ ] trigger build manually from an other system (backend deployment for instance)
- [ ] provide build status/badges

## Global tricks

start by cleaning you scheme scheme: only one target in the test section of each scheme

if you are using a workspace move scheme to workspace instead of project
create bot from workspace

if issues (don't run tests, don't find libs, don't build pods...) delete and re-create the bot.

## Cocoapods 

use this before build trigger:

```
cd LastFMtest

export LC_ALL="en_US.UTF-8"

if [ ! -e "$HOME/.cocoapods/repos" ]
then
    pod setup
fi



pod install
```

note: the `pod setup` will be run only the first time as the `_xcsbuildd` user.