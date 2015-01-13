# Xcode Server experiments

The goal of this repository is just to test Xcode server and provides some receipts for the following usecases :
- [x] [Cocoapods](#cocoapods)
- [x] [Runing tests](#tests)
- [x] [KIF](#kif)
- [x] [build a specific branch](#git-branch)
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

![Test Target](Images/scheme-tests.png)

## KIF (used for integration testing) <a id="kif"></a>

The KIF build is pretty similar to the [CI build](#tests).
The best way to install/use KIF is probably to set it up with Cocoapods.

TODO: find a way to restart the build if it's fail.

## Build a specific branch <a id="git-branch"></a>

This is obviously helpfull if you use git branch for different stats of your project (to be tested, releasable stuffs, completed features...)

First you need to match the local branches with te remote branches.

Checkout the branch you want to build and create you bot.

You can check which branch is selected (as Xcode see it) using the "Source Control" menu (May seams obvious if you use Xcode to commit your code).

![Git Branch](Images/git-branch.png)

You can display the current branch while building using `git branch`