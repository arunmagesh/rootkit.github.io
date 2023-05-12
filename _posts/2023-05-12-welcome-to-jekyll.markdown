---
layout: post
title:  "First Post: Reversing aliexpress LED sound bar"
date:   2023-05-12 18:20:22 +0200
categories: reversing
---
Have you seen those RGB LED soundbars with Bluetooth control on AliExpress? I got one and wanted to figure out how it worked. Turns out, it uses Bluetooth Low Energy to talk to your phone and has a custom binary protocol for LED colors and patterns.

![alt text](images/1/3.png)

To reverse engineer the protocol, I used a Bluetooth HCISNOOP to capture the packets sent by the app and analyzed their contents. Once I understood how the protocol worked, I could control the soundbar directly from my phone without the default app. 

![alt text](images/1/1.png)
The communication is pretty simple. it uses the handle 0x0009 and sends the data of length 9 byte to control the device. 

The app has an feature to get the input from the phone's mic and control of the rgb light. it is useless because they control the static RGB data and not the pattern from the audio input. 

![alt text](images/1/2.png)

You can download the hcisnoop data [here](data/001_ble_rgb.log.gz) (it is in log format. open it from wireshark)

{% highlight ruby %}
7e0705039eff0010ef is an example packet breakdown for RGB control.
7e0503c506ffff00ef is an example packet breakdown for pattern control.

7e - Header
07/05 - Mode (07 for static RGB control and 05 for pattern control)
05/03 - seems to be size in use. for rgb control it is 05 and pattern control it is 03.
03/c5 - in RGB control mode it is 03 and pattern index in pattern control 
9eff00 - RGB value in hex
10/00  - 10/20 for rgb color mode or music mode and 00 for pattern control.
ef - end header

{% endhighlight %}

To set a pattern the data should be 
{% highlight ruby %}
7e0503 c5 06ffff00ef
{% endhighlight %}

To set the rgb the data should be 
{% highlight ruby %}
7e070503 9e ff 00 10ef
{% endhighlight %}


if you want to test it you can either use nrfconnect or chrome's internal bluetooth GATT tool {% highlight ruby %} about://bluetooth-internals {% endhighlight %}
