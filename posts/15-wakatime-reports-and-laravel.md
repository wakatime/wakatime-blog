---
Title: WakaTime Reports and Laravel
Date: 2015-02-02
Image: https://wakatime.com/static/img/blog/laravel-site.png
Author: Mario Basic
AuthorUrl: https://wakatime.com/@mabasic
AuthorGravatar: https://secure.gravatar.com/avatar/c21c351355a4e7d63bddbd25ab53b757?s=150&d=identicon
Categories: Tips, Technology
Tags: laravel, php, api
Released: true
---
In this post I will show you how to use the PHP package that I wrote for WakaTime [wakatime-php-api](https://github.com/mabasic/wakatime-php-api) with Laravel.

I was in a situation where I wanted to show my time logged for the last 30 days (on all projects) on my website. The website is built using Laravel. WakaTime provides a good API for this purpose, but I wanted to consume it the PHP way, so I created a PHP package that anyone can use.

<p style="text-align:center;">
    <a href="http://basicit.company/"><img class="img-thumbnail" src="https://wakatime.com/static/img/blog/laravel-site.png" alt="Basic IT site" title="Basic IT site" /></a>
</p>

_You can view the result on [Basic IT](http://basicit.company)._

[WakaTime](https://wakatime.com) provides Open-source plugins for automatic time tracking & insights into your programming. In short **Analytics for Programmers**.

## WakaTime - Analytics for Programmers

You just have to install a plugin for your favorite text-editor or IDE and WakaTime will do the rest. It tracks the time you spend programming, it detects programming languages, it detects files and time spent on a file, it tracks time you spend on a specific project and has many other features.

The package has a lot of convinient methods like:

- getHoursLoggedFor
- getHoursLoggedForLast
- getHoursLoggedForToday
- getHoursLoggedForYesterday
- getHoursLoggedForLast7Days
- getHoursLoggedForLast30Days
- getHoursLoggedForThisMonth
- getHoursLoggedForLastMonth

and the two official methods from WakaTime 
from which it gets the numbers for the above methods:

- currentUser
- dailySummary

As I mentioned earlier in this post I will show you how to consume this package in Laravel, so let's start and dive in.

## Installation 

Add to your composer.json:

```
"mabasic/wakatime-php-api": "~1.0"
```

and run `composer update` or type this from command line:

```
composer require "mabasic/wakatime-php-api=~1.0"
```

## API key

To get your API key for WakaTime go to [WakaTime Settings](https://wakatime.com/settings) and copy the **secret key**.

We will store that key in Laravel in `app/config/services.php` file. Open that file and write the following:

```
'wakatime' => [
    'api_key' => 'your-secret-key'
]
```

> Or you can use the `.env` way to store the API key. See the [documentation](http://laravel.com/docs/4.2/configuration#protecting-sensitive-configuration).

## Service provider

Now we have to create a Service Provider for WakaTime. We will create a singleton, pass the **api_key** and **guzzle** dependency and return the WakaTime object.

```
<?php namespace Acme\Providers;

use App;
use Illuminate\Support\ServiceProvider;
use Mabasic\WakaTime\WakaTime;
use GuzzleHttp\Client as Guzzle;

class WakaTimeServiceProvider extends ServiceProvider {

    public function register()
    {
        App::singleton('Mabasic\WakaTime\WakaTime', function ($app)
        {
            $key = $app['config']->get('services.wakatime.api_key');

            $wakaTime = new WakaTime(new Guzzle);
            $wakaTime->setApiKey($key);

            return $wakaTime;
        });
    }

}

```

Add this to your `app.php` config file under **providers**:

```
'Acme\Providers\WakaTimeServiceProvider'
```

and run `composer dump-autoload`.

## Controller

Now that we have everything set up, we need to inject the WakaTime class into our controller **(dependecy injection)**. Let's create a **BlankController** that has one method and when called it returns _hours logged for last 30 days_.

```
<?php

use Mabasic\WakaTime\WakaTime;

class BlankController extends \BaseController {

    protected $wakaTime;

    public function __construct(WakaTime $wakaTime)
    {
        $this->wakaTime = $wakaTime;
    }

    /**
     * Returns hours logged for last 30 days
     */
    public function index()
    {
        $workHoursThisMonth = $this->wakaTime
            ->getHoursLoggedForLast30Days();

        return $workHoursThisMonth;

        /**
         * Or you can return some view and pass the variable.
         * 
         * Be sure to create a view `example.blade.php` 
         * in views folder if you are going to do this.
         */
        //return View::make('example')->with('hours', $workHoursThisMonth);
    }

}

```

## Route

So, the route for this method would be:

```
Route::get('/last30days', 'BlankController@index');
```

or if you prefer resourceful routes:

```
Route::resource('last30days', 'BlankController', ['only' => 'index']);
```

If you have done everything correctly you will get your hours logged in last 30 days using WakaTime. If you have questions, suggestions or have found a mistake, please leave me a comment bellow and I will respond as soon as possible.

**P.S.** If you are looking for a command line way of interacting with WakaTime, check this out [See your WakaTime report directly in the terminal](https://www.npmjs.com/package/wakatimecli).

_Originally published at [mariobasic.com](http://mariobasic.com/wakatime-reports-and-laravel/) on February 01, 2015._
