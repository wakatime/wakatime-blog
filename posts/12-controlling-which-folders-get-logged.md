---
Title: Controlling Which Folders Get Logged
Date: 2015-03-06
Image:
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: New Features
Tags: dashboard, privacy
---

The latest version of WakaTime allows more control over what gets logged. You can now whitelist directories so only certain directories get logged with WakaTime.

All you need to do is paste this in your `~/.wakatime.cfg` INI file:

    exclude =
        .*
    include =
        path/to/my/projects

What this does is prevent all files from being logged except those listed with `include`.

So if for some reason you want to only track some files, this makes it easier. More power for you!
