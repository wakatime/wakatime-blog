---
Title: Vim Plugin Performance
Date: 2017-10-05
Image: https://wakatime.com/static/img/ScreenShots/wakatime-laptop-1200x628.png
Description: The WakaTime plugin for Vim just got a whole lot faster.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Category: New Features
Tags: plugins, vim
---

*TLDR: [WakaTime for Vim][plugin] now supports async jobs with Vim 8, for a noticable performance improvement.*

Vim recently added support for async jobs in Vim 8.
This is great news for all Vim plugins!
Previously, plugins had to jump through hoops to run commands in the background, and usually with a noticable lag.
Now that Vim supports async jobs, we can run plugins in the background making Vim much snappier and responsive.

WakaTime does most of it's work in a background Python process, however forking that process would sometimes cause Vim to lag or freeze for a second or two.
[This week][commit] I've upgraded the WakaTime plugin to use Vim 8's new async features.
That means you should notice improved performance and hopefully zero lag while using Vim with WakaTime.

Make sure you're using Vim 8 or Neovim to take advantage of the improved WakaTime plugin performance!

[plugin]: https://wakatime.com/vim
[commit]: https://github.com/wakatime/vim-wakatime/commit/99ffbf39cf57c9c10204e02f8991e113f43bbf88
