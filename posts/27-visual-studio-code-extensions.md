---
Title: Visual Studio Code Extensions
Date: 2015-11-18
Image: https://wakatime.com/static/img/vscode-wakatime-release.png
Description: If you haven't heard, Visual Studio Code is now open source on GitHub. Along with this announcement, Visual Studo Code now supports 3rd party extensions!
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Categories: Product
Tags: plugins, vscode
---

If you haven't heard, Visual Studio Code is now [open source](https://github.com/Microsoft/vscode) on GitHub.
Along with this announcement, Visual Studo Code now supports [3rd party extensions](https://marketplace.visualstudio.com/#VSCode)!

<a href="https://wakatime.com/help/plugins/vscode"><img src="https://wakatime.com/static/img/vscode-wakatime-release.png" alt="vscode-wakatime release" class="img-responsive" /></a>

### Browsing & Installing Extensions

Inside Visual Studio Code, press `CMD + Shift + P` (`F1` on Windows) then type `ext`.

<a href="https://marketplace.visualstudio.com/#VSCode"><img src="https://raw.githubusercontent.com/wakatime/vscode-wakatime/master/images/type-install.png" alt="vscode marketplace" class="img-responsive" /></a>

### Creating Extensions

Extensions are written in [TypeScript](https://en.wikipedia.org/wiki/TypeScript), basically JavaScript with types.
To create your own extension, use the Yeoman generator and generate an example plugin:

1. Navigate to `~/.vscode/extensions` in a Terminal.

2. Run `sudo npm install -g yo generator-code && yo code`.

3. Choose `New Extension (TypeScript)` and name your extension.

4. Restart Visual Studio Code and open the Developer Console. You should see a log message `Congratulations, your extension "Test" is now active!`.

5. To edit your extension, open the `src/extension.js` file inside your generated extension directory.

6. If you add node dependencies to your `package.json`, remember to run `npm install` inside your extension folder.

The [WakaTime for VSCode](https://marketplace.visualstudio.com/items/WakaTime.vscode-wakatime) extension is a great example to start with.
Check out the extension source code from the [GitHub repository](https://github.com/wakatime/vscode-wakatime).
You can also use the [WakaTime extension](https://marketplace.visualstudio.com/items/WakaTime.vscode-wakatime) to measure how long it takes you to build your extension!

Here's the [Official Extension Reference](https://code.visualstudio.com/docs/extensionAPI/overview). Have fun creating new Vscode extensions!
