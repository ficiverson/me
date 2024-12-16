---
title: "Mobile app revamp using Flutter in two weeks"
meta_title: "Mobile app revamp using Flutter in two weeks"
description: "CUAC FM port post"
date: 2020-03-14T08:00:00Z
image: "/images/blog/cuacfm/old_iphone.png"
categories: ["Architecture"]
author: "Fernando Souto"
tags: ["CUAC FM", "Android", "iOS", "Flutter", "Mobile"]
draft: false
---

This is the story of how, in exactly 2 weeks, I made an entire revamp of the **[CUAC FM](https://cuacfm.org/)** app considering that working time was limited to the restrictions of a project developed in my spare time.

## Current situation

The situation was as follows.

**CUAC FM** is a *non-profit community media radio station* in **A Coruña (Galicia, Spain)** that had two really different apps created in different years and with two different codebases, which made really difficult to update both separately, so I decided to use the best mobile cross-platform development kit: **[Flutter](https://flutter.dev/)**.

![Android App 2017](/images/blog/cuacfm/old_android.png)

![iOS App 2018](/images/blog/cuacfm/old_iphone.png)

## Radioco

**[Radioco](http://radioco.org/es/)** is a radio management application suitable for community radios that makes easy scheduling, live recording and publishing.

**CUAC FM** is currently using **Radioco** as the provider for podcast content generation and program scheduling. If you are interested in this awesome free software please visit the official website in [http://radioco.org/es/](http://radioco.org/es/)

The new **CUAC FM** mobile app is also free software and its called **radiocom-flutter**. You can find the source code on **[Github](https://github.com/ficiverson/radiocom-flutter)**. Feel free to use it and remember: **PRs and feedback are welcome !!**

## Development and scope

The first requirement was that this app should go live in a two-week sprint (the time restriction was very important to reach **CUAC FM** anniversary) and the initial scope of the app were the following epics:

- Live streaming
- Search podcast content and episode detail
- Streaming of the podcasts
- News reader of the station and news detail
- Additional information of the station, image gallery, contact form, privacy policy, software licenses and links to social media*

The development happened very fast and without any complications since the app fits perfectly into what **Flutter** can give us and I even managed to add *push notifications* and support for *dark mode* on both platforms. This is very important because both things were planned to be part of the second release.

The support for dark mode was funny because in **Flutter** the integration with both platforms is very very easy and I tried to release this because past week **WhatsApp** announced that they had it and I said: **Ok, CUAC FM too**.
>  In Flutter everything is so fast that the iteration on the product is continuous

## Screenshots of the app two weeks after

![Android and iOS 2020](/images/blog/cuacfm/new_app.png)

![Android and iOS dark mode 2020](/images/blog/cuacfm/new_app_dark.png)

## Download links

If you want to try the app, it would be great to hear feedback from you :)

You can try both apps here:

[**Download in Google Play](https://play.google.com/store/apps/details?id=com.app.cuacfm.radio.coruna&hl=es)
[Download in Appstore](https://apps.apple.com/es/app/cuac-fm/id536600585)**
