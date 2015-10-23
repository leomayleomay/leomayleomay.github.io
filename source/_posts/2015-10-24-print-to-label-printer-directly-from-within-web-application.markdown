---
layout: post
title: "Print to label printer directly from within web application"
date: 2015-10-24 07:49
comments: true
categories: ruby rails pos commerce
---

I've been helping a local store developing a Point of Sale (POS) system recently, and one of their requirements is printing price tag from within the product management interface. The first time I heard about this, I thought it was impossbile to have a web app to talk to a local printer. "There must be something like a client app, which installed on the client side, retriving the print jobs via a REST api and send it to the label printer.", my initial thought leads me to start looking for [Java Applet](https://en.wikipedia.org/wiki/Java_applet), which pretty much acts as a client app but residing in a web page.

After few google searches, I found a Java Applet named [jZebra](https://code.google.com/p/jzebra/wiki/About), which looks pretty much what I need, and it's now moved to to [QZ](https://qz.io). 

Download and install the free version of the software, now I have QZ-Tray as a broker who's going to talk to the printer, the applet in the web page is going to communicate with QZ-Tray via WebSocket, it works like a magic, but there're few pitfalls I need to mention here hoping it could help someone like me.

* setup the label printer

QZ has provided a really great tutorial on [how to setup the label printer](https://qz.io/wiki/setting-up-a-raw-printer-in-osx), I tried to follow the instructions, but never succeeded. The thing is from within http://localhost:631/admin, you should not click "Add Printer", while you should click "Find New Printers", and follow the rest of the instructions from QZ. You may need to restart the QZ-Tray to pick up the changes.