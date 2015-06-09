---
Title: Custom Visualizations using KeenIO
Date: 2015-06-10
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Categories: Engineering
---

Want more visualizations from your WakaTime data? Now you can export your WakaTime data into [Keen.io][keenio] and take advantage of their powerful query explorer to create custom visualizations.

1. Install [wakadump][wakadump] with:

    `sudo pip install wakadump`

2. Export your logged time from WakaTime's [settings page][settings page].

3. Upload your WakaTime data to KeenIO by running this command:

    ```wakadump --input wakatime-user.json --format keen.io
    keen.io Project ID: 5570da806f31a24af926025e
    keen.io project Write Key: XXXX
    Preparing keen.io events...
    Uploading events to keen.io...
    Complete.```

4. Create custom queries using KeenIO's [Data Explorer][keenio data explorer]

Some examples visualizations:

## Most productive day of the week

<img src="https://wakatime.com/static/img/blog/keenio-weekdays.png" alt="KeenIO Visualization" title="KeenIO Visualization" width="100%" class="img-thumbnail"/>

## Top files worked in over the last 2 years

<img src="https://wakatime.com/static/img/blog/keenio-files.png" alt="KeenIO Visualization" title="KeenIO Visualization" width="100%" class="img-thumbnail"/>

## Editor usage over the last 2 years

<img src="https://wakatime.com/static/img/blog/keenio-editors.png" alt="KeenIO Visualization" title="KeenIO Visualization" width="100%" class="img-thumbnail"/>

## Programming languages used over the last 2 years

<img src="https://wakatime.com/static/img/blog/keenio-languages.png" alt="KeenIO Visualization" title="KeenIO Visualization" width="100%" class="img-thumbnail"/>


[wakadump]: https://github.com/wakatime/wakadump
[settings page]: https://wakatime.com/settings
[keenio]: https://keen.io/
[keenio data explorer]: https://keen.io/blog/114588771746/introducing-data-explorer
