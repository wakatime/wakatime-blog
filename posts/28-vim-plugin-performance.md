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

**Upgrade your Vim WakaTime plugin for a huge performance improvement!**

Vim 8 recently added support for async jobs.
This is great news for all Vim plugins!
Previously, background processes would block Vim until the process finished forking, which usually caused noticeable lag.
With async jobs, we can fork background processes without affecting the UI.
This makes Vim much snappier with plugins like WakaTime!

To take advantage of this update, first make sure you’re using Vim 8 or Neovim, then run this command to update your WakaTime plugin:

    vim +PluginUpdate

P.S. Here’s the [commit][commit] to see how this update looks in the plugin source code.


[plugin]: https://wakatime.com/vim
[commit]: https://github.com/wakatime/vim-wakatime/commit/99ffbf39cf57c9c10204e02f8991e113f43bbf88
