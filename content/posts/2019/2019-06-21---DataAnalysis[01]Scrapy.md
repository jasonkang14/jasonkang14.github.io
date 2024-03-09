---
title: Data Crawling Using Scrapy
date: "2019-06-21T19:56:37.121Z"
template: "post"
draft: false
slug: "/posts/DataAnalysis-Scrapy"
category: "Python"
tags:
  - "Python"
description: "Data Analysis project using Scrapy and Luigi"
---

With a colleague at WeCode, which is a coding bootcamp based in Seoul, South Korea, I am doing a data analysis project on the relationship betwen box office and music daily chart. We are trying to see how box office affects what people listen to in their daily lives. Not sure how much correlation there would be, but this would be beneficial.

#1. What is Scrapy?
According to the official document, `Scrapy` is an application framework for crawling web sites and extracting structured data which can be used for a wide range of useful applications, like data mining, information processing or historical archival.

#2. Starting a `Scrapy` project
I intalled `Scrapy` in a virtual environment created with `Miniconda`.

```
(bedataproj) pip install scrapy
```

And created a `Scrapy` project

```
(bedataproj) scrapy startproject project_name
```

A `genspider` command helps create a data crawling spider.

```
(bedataproj) scrapy genspider spider_name website
```

Below is the spider

```
##spider.py

import scrapy

class MusicSpiderSpider(scrapy.Spider):
    name = 'music_spider'
    start_urls = ['http://www.mnet.com/chart/TOP100/20190623']

    def parse(self, response):
        top_selector = '.MMLITitle_Box'

        for music in response.css(top_selector):
            musician = '.MMLITitle_Info .MMLIInfo_Artist ::text'
            song = '.MMLITitleSong_Box .MMLI_Song ::text'
            album = '.MMLITitle_Info .MMLIInfo_Album ::text'
            yield {
                "song" : music.css(song).extract_first(),
                "musician" : music.css(musician).extract_first(),
                "album" : music.css(album).extract_first(),
            }

```

I will explain each component of the spider. <br>

1. `name` is the name of the spider<br>
2. `start_urls` is the website you would like to crawl your data from <br>
3. `parse` function is how you are going to parse the crawled data

When you crawl data from a webiste using `Scrapy`, the easiest way is using `class` and/or `id` as if you are applying CSS to HTML.

Appending `::text` to a selector means that I am going to crawl innerText of the class. <br>
And then I called `extract_first()` on the object returned by `music.css(selector)` because I am extracting the first element that matches the selector--in case there are more than one.

Then you run `Scrapy` like below

```
scrapy runspider spider_name
```
