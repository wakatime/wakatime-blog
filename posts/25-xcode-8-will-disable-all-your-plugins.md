---
Title: Xcode 8 Will Disable All Your Plugins
Date: 2016-08-18
Image: https://wakatime.com/static/img/editor-icons/xcode-128.png
Description: Xcode 8 introduces Source Editor extensions, but that means all your current extensions will be disabled.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Category: Engineering
Tags: plugins, xcode
---

*TLDR: Apple will disable all your Alcatraz plugins in Xcode 8. Read this
[GitHub issue][issue] for more info and [give Apple feedback][radars] to let
them know we need a fully functioning replacement before disabling the
existing extensions.*

*Update:* [This tool][patch] will re-enable extensions in your Xcode8 app.

Xcode 8 comes with some new [features][features], like the ability to write
custom extensions officially called Source Editor extensions. With Source
Editor extensions, you can run code to interact, to a very limited extent,
with your Xcode workspace.

### Official Source Editor extensions not ready for prime time

Unfortunately, Source Editor extensions are very limited: They can only
manipulate text and selections and only when the user selects them from a
menu. This is only a small portion of the features provided in the existing
[Alcatraz plugin ecosystem][alcatraz] for Xcode.

### Unofficial plugins will be disabled when Xcode 8 is released

With Xcode 8, Apple has decided to disable all existing
[Xcode plugins][alcatraz] in favor of new Source Editor extensions. However,
because Source Editor extensions only provide a small portion of the features
required by plugins like [WakaTime][xcode-wakatime], that means most existing
plugins can not be converted to Source Editor extensions. Hopefully someday
Source Editor extensions will be able to receive notifications from user
events and run without you having to select them from a menu each time. Until
then, you will unfortunately have to live without your existing Xcode plugins.

### Give Apple Feedback

Giving Apple feedback by creating Radar Issues describing the plugin features
you need might help improve Source Editor extensions. You can create a radar
issue [here][radars]. Also, this [GitHub issue][issue] contains more history
and a discussion from plugin developers.

[features]: https://developer.apple.com/xcode/features/
[xcode-wakatime]: https://github.com/wakatime/xcode-wakatime
[alcatraz]: http://alcatraz.io/
[radars]: https://bugreport.apple.com/
[issue]: https://github.com/alcatraz/Alcatraz/issues/475
[patch]: https://s3-us-west-1.amazonaws.com/wakatime/MakeXcodeGr8Again.app.zip
