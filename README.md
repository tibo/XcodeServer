# Xcode Server experiments

The goal of this repository is just to test Xcode server and provides some receipts for the following usecases :
- [x] [Cocoapods](#cocoapods)
- [x] [Runing tests](#tests)
- [x] KIF
- [x] build a specific branch
- [ ] get results as variables after a build
- [ ] post notification on Slack
- [ ] build pull request almost automaticaly
- [ ] upload to testflight/itunes connect/hockey app automaticaly
- [ ] trigger build manually from an other system (backend deployment for instance)
- [ ] provide build status/badges

## Global tricks

When creating a bot, you need to **be in the exact configuration** you want to build on you Xcode Server (same branch, same scheme,...)

Make sure the schemes you want to build are shared.

If you are using a workspace move scheme to workspace instead of project and create your bots from workspace. (ie: do you Cocoapods setup first)

![Scheme](Images/schemes.png)

If issues (don't run tests, don't find libs, don't build pods...) delete the bot, check your workspace configuration and re-create the bot.

## Cocoapods <a id="cocoapods"></a>

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

## Runing tests <a id="tests"></a>

Clean your scheme: only one target in the test section for each scheme.

![Test Target](Images/scheme-test.png)