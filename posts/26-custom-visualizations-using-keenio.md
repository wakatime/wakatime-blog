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
    keen.io project Write Key:
    Preparing keen.io events  [####################################]  100%
    Uploading events to keen.io...
    Complete.```

4. Create custom queries using KeenIO's [Data Explorer][keenio data explorer]

Some examples of visualizations:

* which files have i spent the most time in over the last year
* which editors have i used the most over the last year
* which weekday do i spend the most time programming

My first visualization was to discover which languge I used the most over the last 2 years.

<img src="https://wakatime.com/static/img/blog/keenio-visualization-3.png" alt="KeenIO Visualization" title="KeenIO Visualization" width="100%" class="img-thumbnail"/>


[wakadump]: https://github.com/wakatime/wakadump
[settings page]: https://wakatime.com/settings
[keenio]: https://keen.io/
[keenio data explorer]: https://keen.io/blog/114588771746/introducing-data-explorer
