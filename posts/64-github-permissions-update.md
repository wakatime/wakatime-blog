---
Title: GitHub Permissions Update
Date: 2023-09-22
Image: https://wakatime.com/static/img/blog/github-integration.png
Description: We’ve improved the security of our GitHub integration.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: Engineering
Tags: integrations, github
---

We’ve improved the security of our [GitHub integration][integrations] by migrating to GitHub Apps, for fine-grained permissions and read-only repo access.

## Why GitHub Apps?

Ever since the GitHub released [GitHub Apps][apps release], [many][sentry] [companies][code climate] have transitioned from OAuth to GitHub Apps.
`GitHub Apps` is the term for GitHub’s new OAuth client implementation... it’s GitHub’s re-write of their API’s OAuth authentication flow, but it’s still just OAuth behind the scenes.
The main reason to use GitHub Apps is fine-grained permissions.

When using GitHub’s API without GitHub Apps, the permission scopes are way too broad.
For example, want read-only access to source code?
That’s not possible without using GitHub Apps.

GitHub could have just saved some energy and fixed their existing OAuth scopes, but their product team didn’t ask me before deciding to rewrite the whole thing ;)

## What’s changed?

Now that we’re using GitHub Apps, when you connect the [WakaTime GitHub integration][integrations] you will only be asked to grant read-only permission to your repos.
We don’t actually read your source code, just your commit messages.
But, having access to commit messages means GitHub also grants read-only access to repo contents.

## Onboarding changes

With the old OAuth app, users would grant access to the integration in one step, called the OAuth consent screen.
However, with the new GitHub Apps after the OAuth consent screen redirects back to wakatime.com, users are redirected to a second GitHub Apps install consent screen.
This GitHub Apps install page doesn’t use the OAuth `redirect_uri`, it always redirects back to the `Post installation setup url` defined in the GitHub Apps settings.
It also only redirects back if the user makes changes to the GitHub App’s permissions or selected repository access.
That means the onboarding flow breaks if we redirect to the GitHub Apps install page when the app is already installed and no changes are necessary.
For that reason, after you first authorize the WakaTime GitHub App you’re only redirected to the second GitHub App consent screen if you don’t already have the GitHub App installed.
To find that out, we [list your WakaTime app installations][github api] using the GitHub API and only redirect to the second consent screen if we don’t find any installs.

## Changing the repos WakaTime can access

If you need to edit the repo permissions you’ve granted WakaTime access, click the `Adjust GitHub permissions` link on the [GitHub integration settings][github integration] page.

This migration to GitHub Apps has been something we’ve wanted to do for a while now, and I’m pleased we’ve improved the security of the WakaTime GitHub integration for our users.
Happy coding!


[sentry]: https://github.blog/2018-10-11-sentry-guest-post/
[code climate]: https://github.blog/2018-09-25-migrating-to-github-apps-code-climate-shares-their-story/
[apps release]: https://developer.github.com/changes/2017-05-22-github-apps-production-ship/
[integrations]: https://wakatime.com/integrations
[github api]: https://docs.github.com/en/rest/apps/installations?apiVersion=2022-11-28#list-app-installations-accessible-to-the-user-access-token
[github integration]: https://wakatime.com/integrations/github
