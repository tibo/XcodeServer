# Xcode Server experiments

The goal of this repository is just to test Xcode server and provides some receipts for the following usecases :
- [x] [Cocoapods](#cocoapods)
- [x] [Runing tests](#tests)
- [x] [KIF](#kif)
- [x] [Build a specific branch](#git-branch)
- [x] [Get results as variables after a build](#result-variables)
- [x] [Post notifications on Slack](#slack)
- [ ] [Build pull request automaticaly](#pull-request)
- [ ] [Deploy to testflight](#testflight)
- [x] [Trigger build manually from an other system (backend deployment for instance)](#manual-trigger)
- [ ] [Provide build status/badges](#status)

## Global tricks

When creating a bot, you need to **be in the exact configuration** you want to build on you Xcode Server (same branch, same scheme,...)

Make sure the schemes you want to build are shared.

If you are using a workspace move scheme to workspace instead of project and create your bots from workspace. (ie: do you Cocoapods setup first)

![Scheme](Images/schemes.png)

If issues (don't run tests, don't find libs, don't build pods...) delete the bot, check your workspace configuration and re-create the bot.

Other tips: check if the app is runing as a Testflight build:
```
[[[[NSBundle mainBundle] appStoreReceiptURL] lastPathComponent] isEqualToString:@"sandboxReceipt"]
```
(credit [Paul Haddad](https://twitter.com/tapbot_paul/status/557551769496997888)))

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

## Result Variables <a id="result-variables"></a>

You can display all the environement variable before or after a build using the `set` command.

Here is an example of it's output:
```
BUILD RESULT
Apple_PubSub_Socket_Render=/private/tmp/com.apple.launchd.7olKoZBGgD/Render
BASH=/bin/bash
BASH_ARGC=()
BASH_ARGV=()
BASH_LINENO=([0]="0")
BASH_SOURCE=([0]="/var/folders/65/nc5b0s2s5qd_p1zch8nc44h8000086/T/F636AFD8-B41E-43E6-AF8B-A76FE4C29652-24395-00001AA57C68B4BD")
BASH_VERSINFO=([0]="3" [1]="2" [2]="53" [3]="1" [4]="release" [5]="x86_64-apple-darwin14")
BASH_VERSION='3.2.53(1)-release'
DIRSTACK=()
EUID=262
GROUPS=()
HOME=/var/_xcsbuildd
HOSTNAME=ThibautsMacBook
HOSTTYPE=x86_64
IFS=$' \t\n'
LOGNAME=_xcsbuildd
MACHTYPE=x86_64-apple-darwin14
OPTERR=1
OPTIND=1
OSTYPE=darwin14
PATH=/Applications/Xcode.app/Contents/Developer/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin
PIPESTATUS=([0]="0")
PPID=24395
PS4='+ '
PWD=/Library/Developer/XcodeServer/Integrations/Caches/43b017268c0bed5812627a58641cce7e/Source
SHELL=/bin/false
SHELLOPTS=braceexpand:hashall:interactive-comments
SHLVL=1
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.p8UZMaMv1j/Listeners
TERM=dumb
TMPDIR=/var/folders/65/nc5b0s2s5qd_p1zch8nc44h8000086/T/
UID=262
USER=_xcsbuildd
XCS=1
XCS_ANALYZER_WARNING_CHANGE=0
XCS_ANALYZER_WARNING_COUNT=0
XCS_BOT_ID=43b017268c0bed5812627a58641cce7e
XCS_BOT_NAME='Branch: My Awesome Feature'
XCS_BOT_TINY_ID=F4B9E80
XCS_ERROR_CHANGE=0
XCS_ERROR_COUNT=2
XCS_INTEGRATION_ID=43b017268c0bed5812627a58641fbe7f
XCS_INTEGRATION_NUMBER=3
XCS_INTEGRATION_RESULT=build-errors
XCS_INTEGRATION_TINY_ID=00B279C
XCS_OUTPUT_DIR=/Library/Developer/XcodeServer/Integrations/Integration-43b017268c0bed5812627a58641fbe7f
XCS_SOURCE_DIR=/Library/Developer/XcodeServer/Integrations/Caches/43b017268c0bed5812627a58641cce7e/Source
XCS_TESTS_CHANGE=0
XCS_TESTS_COUNT=0
XCS_TEST_FAILURE_CHANGE=0
XCS_TEST_FAILURE_COUNT=0
XCS_WARNING_CHANGE=0
XCS_WARNING_COUNT=0
XCS_XCODEBUILD_LOG=/Library/Developer/XcodeServer/Integrations/Integration-43b017268c0bed5812627a58641fbe7f/build.log
XPC_FLAGS=0x0
XPC_SERVICE_NAME=com.apple.xcsbuildd
_='BUILD RESULT'
__CF_USER_TEXT_ENCODING=0x106:0x0:0x0
```

The `XCS_INTEGRATION_RESULT` variable provide status of the build.
It takes the following values : 
- `unknown`: unknown, the build is probaly still in progress
- `build-errors`: the build has failed
- `warnings`: the build has succeded with warnings
- `analyzer-warnings`: static analyzes issues
- `test-failures`: the tests failed
- `succeeded`: probably the status you want to check

See also: [Xcode CI script variables](https://gist.github.com/quellish/f279f7b00c1bfd343468)

## Post notifications on Slack <a id="slack"></a>

Just create a post build trigger and post it as a incoming Slack webhook using curl.

```
PAYLOAD="{\"channel\": \"#general\", \"username\": \"XcodeServer\", \"text\": \"${XCS_BOT_NAME} failed with status: ${XCS_INTEGRATION_RESULT}\", \"icon_emoji\": \":ghost:\"}"
echo $PAYLOAD
curl -X POST --data-urlencode "payload=${PAYLOAD}" https://hooks.slack.com/services/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
```

![Slack Notification](Images/slack.png)

You can either check the value of `$XCS_INTEGRATION_RESULT` yourself or use the options of the trigger step:

![Script Options](Images/script-options.png)

You should be able to do something similar with Hipchat or others.


## Build pull request automaticaly <a id="pull-request"></a>

[xcode-bot-builder](https://github.com/modcloth-labs/github-xcode-bot-builder) doesn't support Xcode 6/Xcode Server 2.0 (Yosemite) for now.

Also I don't know how far we can configure the bot that will be created by this tool.

Keeping a eye on this https://github.com/modcloth-labs/github-xcode-bot-builder/issues/10

List all pull requests of a Github repository: https://api.github.com/repos/AFNetworking/AFNetworking/pulls + authorization if private repository

No official way to script the creation of a new bot.
The "API" xcode-bot-builder was using with Xcode Server 1.0 doesn't seems to be available on Yosemite.


## Deploy to _the new_ testflight <a id="testflight"></a>
 
First of you need to build the IPA and sign it with your App Store identity. An archive action (in Xcode or using xcodebuild) with the right configuration should do the job.

Than you need to create the package for `Application Loader` -> create a folder (bundle) with `itmsp` as extension (package.itmsp for instance), put your IPA in this bundle/folder and create a `metadata.xml` file (still in the bundle) with the following template:
```
<?xml version="1.0" encoding="UTF-8"?>
<package version="software5.2" xmlns="http://apple.com/itunes/importer">
    <software_assets apple_id="{{YOUR_APP_ID}}"
        bundle_short_version_string="{{YOUR_MARKETING_VERSION}}"
        bundle_version="{{YOUR_BUILD_VERSION}}"
        bundle_identifier="{{YOUR_BUNDLE_ID}}">
        <asset type="bundle">
            <data_file>
                <file_name>{{YOUR_IPA_FILE.ipa}}</file_name>
                <checksum type="md5">{{MD5_OF_THE_IPA}}</checksum>
                <size>{{SIZE_OF_THE_IPA}}</size>
            </data_file>
        </asset>
    </software_assets>
</package>
```

take care to set :
- the apple_id of your app (App Store/iTunes connect id)
- the short bundle version of the app
- the build number of the build
- the bundle identifier of your app
- the name of your IPA file
- the MD5 hash of the IPA file (use the `md5` command from the terminal)
- the size of the IPA file in byte

Your should also be able to set some release notes in the manifest file (must dig in the manifest spec).

Still looking for a way to "enable" the build for Prerelease automaticaly.

related linsk: 
- https://github.com/drewcrawford/CaveJohnson
- https://github.com/KrauseFx/deliver
- http://fastlane.tools/
- http://bou.io/UploadingScreenshotsWithITMSTransporter.html

## Trigger build manually from an other system <a id="manual-trigger"></a>

Manually trigger a new integration by POSTing with Basic/digest Authentication to:
https://{xcodeserver}/xcode/api/bots/{botid}/integrations

## Provide build status/badges <a id="status"></a>

Not really possible with Xcode Server itself.

Maybe we should use the [Github status](https://developer.github.com/v3/repos/statuses/) from a post build script step.
