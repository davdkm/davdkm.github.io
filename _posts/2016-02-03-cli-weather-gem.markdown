---
layout: post
title:  "CLI Data Gem Project"
date:   2016-02-03 21:28:33 -0800
categories:
---

So, I reached the part of my learn-co journey where we’re tasked with creating a ruby gem with a CLI. This gem should get data from an external data source via scraping or through API call.  Furthermore, it should ask for input from the user and then dig into nested data and return the requested information.

For this project, I decided to make a program that would allow the user to receive weather forecasts for a location that he/she specifies. I went with API from openweathermap.org  since it’s freely available and appeared to have everything I needed. It takes in a city name or zip code, and a country code, then returns weather data in JSON/XML format.  

Now for the coding. My CLI would ask the user for input for the city name/zip code followed by a comma, and then the country code. The code would then take that input and create an API call along with options to return the info in XML format and in imperial units. Luckily, the guide on the openweather website had a guide on how to craft your request to do that.  I needed a method that would split the user input and use string interpolation to feed into API call.  For the requesting, I’ll use Nokogiri.

Here’s snippet of the xml that’s returned

{% highlight xml %}
<weatherdata>
  <location>
  </location>
  <forecast>
  <time from="2016-02-03T03:00:00" to="2016-02-03T06:00:00">
    <symbol number="800" name="sky is clear" var="01n"/>
    <precipitation/>
    <windDirection deg="51.5002" code="NE" name="NorthEast"/>
    <windSpeed mps="1.21" name="Calm"/>
    <temperature unit="imperial" value="27.45" min="27.45" max="36.2"/>
    <pressure unit="hPa" value="968.44"/>
    <humidity value="79" unit="%"/>
    <clouds value="clear sky" all="0" unit="%"/>
  </time>
    <time from="2016-02-03T06:00:00" to="2016-02-03T09:00:00">
    <symbol number="800" name="sky is clear" var="01n"/>
    <precipitation/>
    <windDirection deg="50.5029" code="NE" name="NorthEast"/>
    <windSpeed mps="2.51" name="Light breeze"/>
    <temperature unit="imperial" value="21.96" min="21.96" max="30.22"/>
    <pressure unit="hPa" value="969.13"/>
    <humidity value="76" unit="%"/>
    <clouds value="clear sky" all="0" unit="%"/>
  </time>
etc...
{% endhighlight %}

Cool, we have some weather forecasts! But, the weather data returned for the next five days is in a decidedly reader unfriendly format. I need some methods that make sense of this mess of code and output it in way that’s more readable. Nokogiri to the rescue! Mercifully, the #css method works on xml elements.  The elements I’m interested in are :time, :humidity, :temperature, and :symbol which looks like a short description of the weather.  This method, which I named #call, would iterate over all the time elements and create a hash that stored these keys and values.

Great, now I have a store of hashes. I now need a method that outputs the information in these hashes.  But, if I did that, the cli would be flooded with a wall of text. How about the weather for just one day and not the next five. I would need a method that outputs the next 5 days in #day_to_forecast and ask which one the user would like a forecast on in #get_day, then displays the data for that day in #forecast.

I left the code there since the CLI had accomplished what I had set out to do.  However, when I came back to the code, I realized that I was only using class methods and variables.  Instead, I opted to create a new instance of the Weather class and separate the #gets methods and API caller into Gets and Apicall classes. Then I updated my methods to be instance methods to work on the newly created instances on the Weather class.
