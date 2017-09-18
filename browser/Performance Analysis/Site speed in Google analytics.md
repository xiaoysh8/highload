Site speed in Google analytics

Google analytics allows you to measure the speed of loading pages for visitors. Since analytics works on the client side, you can understand the real speed of loading pages for visitors. Google analytics Page Timings

Information is available in the "Site Speed" section: Google analytics Site Speed

Page Timings

The Page Timings section will show the slowest pages on the site: Google analytics Page Timings

Speed ​​Suggestions will show the recommendations of Pagespeed for your site.

User Timing

The most powerful tool for speed analysis is User Timing. This section allows you to determine the duration of any events, for example:

Time before clicking on the banner
Time to focus on the form field
Time to load external libraries
Time of writing a comment
Time spent watching the video
Google analytics User Timing
To measure the time of an event, you need to insert a call to ga (assuming that the Google Analytics code is on the site):

ga('send', 'timing', 'Категория', 'Название события', 20, 'Дополнительная метка');
For example:

ga('send', 'timing', 'Images', 'Load all images', 1470, 'Media content');
# It is understood that we send the total time it took to download all the pictures

ga('send', 'timing', 'Comment', 'Send comment', 5250, 'User content');
# It is understood that we send the total time it took to write a comment

Download time of external resources

An example of how you can measure the load time of a JS library or any other external resource:

var start = new Date().getTime();
var js = document.createElement('script');
js.src = '/jquery.js';

var s = document.getElementsByTagName('script')[0];
js.onload = function() {
    var delta = (new Date().getTime()) - startTime;
    ga('send', 'timing', 'jQuery', 'Load', delta, 'Resources');
}
s.parentNode.insertBefore(js, s);
Picture upload time

We measure and send the download time of all the pictures on the site:

var start = new Date().getTime();
var js = document.createElement('script');
js.src = '/jquery.js';

var s = document.getElementsByTagName('script')[0];
js.onload = function() {
    var delta = (new Date().getTime()) - startTime;
    ga('send', 'timing', 'jQuery', 'Load', delta, 'Resources');
}
s.parentNode.insertBefore(js, s);
# We measure the time from the insertion of "script" to the "load" event.

A utility for tracking the duration of events

This JS code will help you to easily track the speed of events:

var ga_time_tracker = {
    times: {},
    start: function(category, variable, label)
    {
        ga_time_tracker.times[category + ':' + variable + ':' + label] = new Date().getTime();
    },
    
    end: function(category, variable, label)
    {
        ga('send', 'timing', category, variable, (new Date().getTime()) - ga_time_tracker.times[category + ':' + variable + ':' + label], label);
    }
}
It is very easy to use (you can not use the label parameter):

ga_time_tracker.start('images', 'load all');

# делаем какие-то действия, длительность которых хотим измерить
ga_time_tracker.end('images', 'load all');
Using User Timing in PHP

You can also measure events in PHP. For example:

Total time spent on all Mysql requests.
The time for the image to be processed, after downloading.
The working hours of the API methods.
In PHP, it's enough to define the duration of the event and then make a JS-call with this value:

<?
$t = round(microtime(true)*1000);

# какой-то код, например работа с mysql
$delta = round(microtime(true)*1000) - $t;
?>
<script>ga('send', 'timing', 'mysql', 'queries duration', <?=$delta?>, 'server');</script>
The most important

Analyzing the speed of the pages with the help of User Timing will allow you to determine the real time of loading the site for visitors and decompose it into small events.