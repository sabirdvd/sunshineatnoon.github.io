---
layout: post
title: How to use Scrapy to scrape Amazon.com and save to sqlite
published: true
---

I am currently working on a project, which needs some backpack images and information. After spending some time googling, I found a handy tool called [Scrapy](http://scrapy.org/)--a python module which helps me crawl Amazon.com easily and neatly. In this post, I want to write down how I implemented this. Since I am a newbie in Scrapy, please feel free to point out my mistakes. Thanks.

## System.

All my code runs fine on CentOS7 and MacOS.

## Installation.

If you have [pip](https://pypi.python.org/pypi/pip), then the installation is easy, just run:

        pip install scrapy
If you don't have pip, I strongly recommend you to install it:

		sudo yum install python-pip

## Start a project.

To start a scrapy project, cd to where you want to put the project and then run:

		scrapy startproject amazon
Of course you can name your project whatever you like. For me, I name it amazon. By running the command above, you will find a folder named amazon, this is our project folder. In order finish our crawler, we need to modify three files: items.py, pipelines.py, settings.py and create a new file within spiders folder and call it amazonSpider.py.
