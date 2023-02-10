---
Title: GitHub adds WakaTime as Secret Scanning Partner
Date: 2023-02-10
Image: https://wakatime.com/static/img/blog/leaky-faucet.jgp
Description: GitHub now notifies WakaTime of any leaked access tokens found in public repositories so they can be revoked.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: Engineering
Tags: github, secret-scanning
---

<img src="https://wakatime.com/static/img/blog/leaky-faucet.jpg" class="img-thumbnail" alt="leaky faucet" style="width:90%" />

<div style="font-size:10px;text-align:right;width:90%;margin-top:-18px;margin-bottom:10px;">
  Photo by <a href="https://unsplash.com/@sanatoga?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Joe Zlomek</a> on <a href="https://unsplash.com/photos/zrb_TkHPVtE?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</div>

We’re proud to announce GitHub has partnered with WakaTime to scan for leaked api keys and other secret WakaTime tokens.

WakaTime tokens allow users to programmatically access their WakaTime code statistics.
GitHub secret scanning protects users by searching repositories for secret tokens, and flagging them for revokation.
By identifying and flagging these secrets, GitHub helps prevent data leaks and fraud.

GitHub will forward access tokens found in public repositories to WakaTime, who will immediately revoke the leaked token and email the token’s owner with instructions on next steps.
You can read more information about WakaTime tokens [here][auth docs], and view the original announcement on GitHub’s [Changelog Blog][gh blog].

[auth docs]: https://wakatime.com/developers/#authentication
[gh blog]: https://github.blog/changelog/2023-02-10-wakatime-is-now-a-github-secret-scanning-partner/
