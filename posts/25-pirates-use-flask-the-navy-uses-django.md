---
Title: Pirates use Flask, the Navy uses Django
Date: 2015-04-21
Image: https://wakatime.com/static/img/blog/django-vs-flask-performance-insignificant.png
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Category: Engineering
Tags: flask, django, python
---

If you're testing out a new idea or getting a product going, you have to choose a web stack to build it on.
For Python devs, [Flask][flask] and [Django][django] are the two most popular web framework options.
I have experience with both and have chosen one or the other for my myriad of projects and companies.
My current product, [WakaTime][wakatime] is built with Flask and this choice has helped us achieve our goals of building an API heavy product which enables 27,000+ developers to use and extend.

Having experienced the benefits of choosing the right framework, I created a worksheet to help other devs decide.
You can go through it and get a "result" of the best framework to use.

<a href="/django-vs-flask-worksheet" target="_blank" style="color: #ffffff; text-shadow: 0 -1px 0 rgba(0, 0, 0, 0.25); background-color: #5bb75b; background-image: -moz-linear-gradient(top, #62c462, #51a351); background-image: -webkit-gradient(linear, 0 0, 0 100%, from(#62c462), to(#51a351)); background-image: -webkit-linear-gradient(top, #62c462, #51a351); background-image: -o-linear-gradient(top, #62c462, #51a351); background-image: linear-gradient(to bottom, #62c462, #51a351); background-repeat: repeat-x; border-color: #51a351 #51a351 #387038; border-color: rgba(0, 0, 0, 0.1) rgba(0, 0, 0, 0.1) rgba(0, 0, 0, 0.25); display: inline-block; padding: 4px 12px; margin-bottom: 0; font-size: 14px; line-height: 20px; text-align: center; vertical-align: middle; cursor: pointer; -webkit-border-radius: 4px; -moz-border-radius: 4px; border-radius: 4px;">Django vs Flask worksheet</a>

Additionally, if you are interested, here are some comparison points for both Django and Flask.

### Differences between Django and Flask

[Django][django github] is older and larger, but [Flask][flask github] has a more active community according to GitHub.

<div style="width:100%;">
    <div style="width:49%;display:inline-block;padding-left:20px;">
        <p><b>Django</b></p>

        <ul style="padding-left:16px;">
            <li>Born in 2005</li>
            <li>Larger community</li>
            <li>13,820 stars</li>
            <li>607 watchers</li>
        </ul>
    </div>
    
    <div style="width:49%;display:inline-block;padding-left:20px;">
        <p><b>Flask</b></p>

        <ul style="padding-left:16px;">
            <li>Born in 2010</li>
            <li>Newer, active community</li>
            <li>13,489 stars</li>
            <li>1,036 watchers</li>
        </ul>
    </div>
</div>


### Landscape

Many companies use Django and Flask because they save time building the product in the beginning and can scale to handle millions of users as their webites grow.

<div style="width:100%;">
    <div style="width:49%;display:inline-block;padding-left:20px;">
        <p><b>Who uses Django?</b></p>

        <ul style="padding-left:16px;">
            <li><a href="http://www.eventbrite.com/" target="_blank">Eventbrite</a></li>
            <li><a href="https://prezi.com/" target="_blank">Prezi</a></li>
            <li><a href="https://bitbucket.org/" target="_blank">Bitbucket</a></li>
            <li><a href="https://instagram.com/" target="_blank">Instagram</a></li>
            <li><a href="https://www.pinterest.com/" target="_blank">Pinterest</a></li>
            <li><a href="https://zerocater.com/ " target="_blank">Zerocater</a></li>
        </ul>
    </div>

    <div style="width:49%;display:inline-block;padding-left:20px;">
        <p><b>Who uses Flask?</b></p>

        <ul style="padding-left:16px;">
            <li><a href="https://wakatime.com/" target="_blank">WakaTime</a></li>
            <li><a href="https://www.twilio.com/" target="_blank">Twilio</a></li>
            <li><a href="http://arstechnica.com/information-technology/2012/11/how-team-obamas-tech-efficiency-left-romney-it-in-dust/" target="_blank">President Obama</a></li>
            <li><a href="http://close.io/" target="_blank">Close.io</a></li>
            <li><a href="https://keen.io/" target="_blank">Keen.io</a></li>
        </ul>
    </div>
</div>


### Performance

Flask serves JSON responses slightly faster than Django.

<div style="text-align:left;">
  <a href="https://www.techempower.com/benchmarks/" target="_blank"><img src="https://wakatime.com/static/img/blog/django-vs-flask-performance.png" alt="Django vs Flask Performance" title="Django vs Flask Performance" width="80%" class="img-thumbnail"/></a>
</div>
<br />

However, they are both insignificant when compared to frameworks in other languages.
*The reason to use Django or Flask is to increase dev performance, build faster, and have a "fast enough" framework.*

<div style="text-align:left;">
  <a href="https://www.techempower.com/benchmarks/" target="_blank"><img src="https://wakatime.com/static/img/blog/django-vs-flask-performance-insignificant.png" alt="Django vs Flask Performance Insignificant" title="Django vs Flask Performance Insignificant" width="70%" class="img-thumbnail"/></a>
</div>
<br />

In conclusion, the reason to use Django or Flask is to save dev time and build faster.
You should use your best judgment and [this worksheet][worksheet] when deciding which framework is best suited for your use case.

I would also be happy to chat and help you decide between Django or Flask.
Just leave a comment here with your question or jump on the #wakatime channel on irc.freenode.net.

P.S. Pirates use Flask, the Navy uses Django. [WakaTime][wakatime] is a pirate ;)

*Update:* Emilien Klein found an [interesting open-source comparison](https://www.openhub.net/p/_compare?project_0=Flask&project_1=Django) of Flask and Django.


[worksheet]: https://wakatime.com/django-vs-flask-worksheet
[flask]: http://flask.pocoo.org/
[django]: https://www.djangoproject.com/
[wakatime]: https://wakatime.com/
[django github]: https://github.com/django/django
[flask github]: https://github.com/mitsuhiko/flask
