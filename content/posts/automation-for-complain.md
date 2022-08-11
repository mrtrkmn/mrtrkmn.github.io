---
title: "github-actions: complain messages to a company in automated way"
date: 2022-02-25T13:53:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["karaca.com.tr", "karaca", "python","selenium","github","workflow","ci", "cd"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Complain to krc.com.tr with automation"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/krc.png" # image path/url
    alt: "Worklow " # alt text
    caption: "Complain message sent successfully " # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


---
> I am alive !!! 

> Right, I did not post anything for long time and  could not imagine that this will be my next blog post after long time, but it is indeed. 
--- 

This is a story about a company who hold the product (- sent for warranty -) for five months and did not respond any emails. 

Let's start from the beginning of the story, the product itself is not important, instead how they (- company -) approached to situation is important. 

I have sent the product to Karaca (- krc.com.tr -) in November 2021 by hoping that it will be fixed and returned in two or three weeks at most. However, it turns out that it will not be the case, just realized after two or three weeks later :) At that time, I did not give any importance to the situation instead waited more, more and more. 

At the end of four months, I decided to take an action, I have tried to reach them through their "customer services" phone number ( +90 850 2525 572 ). However it connects you to a stupid automated replies, do you remember those ? it replies like, e.g "if you would like to learn about your product, press 1 ", "if you would like to learn about warranty process, press 2" and so on. Although I tried all possibilities to reach them through phone, I could not be successfull enough to get them on the line. 

Afterwards, I realized that they are actually responding to questions/complains if you contact with them through their contact form. ( - https://www.krc.com.tr/contact-form - ) At the beginning, I sent some messages through this form, however they replied me with same message everytime. 

The message was:

> You have an active record with the number "XXX-XXXXXXX-XXXXXX" when the necessary control is made based on your request. You will receive a response as soon as possible.

It was really annoying to receive same replies without providing any information about why they did not sent back the product to me all these past four months. 
Since I annoyed to them, I decided to do an automated way of sending messages ( - by providing same message content  - ) through their contact form using Python and Github actions.

I can not deal with them everyday, but automation can :)

Since there will not be explanation of the code, you can think that it is just a piece of code which automatically fills form and send it using bs4 and selenium.

( - The code may include some bugs or unneceassary statements, you may want to update it to re-use)

You can see it in action from [readme file](https://github.com/merkez/krccomplain/blob/main/readme.md) of the project. 

Github workflow file contains cron job and as well as  manually execution of the code. 

```yaml
on:
  workflow_dispatch: # enable manual run
    inputs:
      git-ref:
        description: Git Ref (Optional)
        required: false
  schedule:               # cron job run each day at 10.00 AM according to GMT+03:00 
    - cron: '0 7 * * *'
```


The repository is using secrets to fill out the contact form. Everyday, it sends message to krc.com.tr, takes screenshot of contact page and commits it to screenshots folder and finally finishes the process. 

```yaml 
 - name: Complain on KRC
   run: |
    python main.py
   env:
     PHONE_NUMBER: ${{ secrets.PHONE_NUMBER }}
     COMPLAIN_MESSAGE: ${{ secrets.COMPLAIN_MESSAGE }}
     EMAIL: ${{ secrets.EMAIL }}
```

Push changes (- screenshot -) with timestamp using a bot user to the repository. 

```yaml
  - name: Commit SS push
    run: |
      git config --global user.email "robotcuk@randommail.com"
      git config --global user.name "robotcuk"
      git add 'screenshots/contact-page-*.png'
      git commit -m "${{ steps.date.outputs.date }} Complain is done to KRC on ${{ steps.date.outputs.date }}"
      git push origin -f main
    env:
      GITHUB_TOKEN: ${{ secrets.ROBOTCUK }}
```

All code regarding to described process can be found here: https://github.com/merkez/krccomplain 

To see it in action you can check out its readme file or watch it on youtube: https://youtu.be/zXrbjEpA_20

It can be used for anyone else who would like to complain to https://www.krc.com.tr through contact form until they update the website or put reCAPTCHA version 2.

Oh by the way, the rest of the story and updates about it, is given on project readme file, check it out if you wonder how it is ended up. 
( spoiler alert: - not as fantastic as you may think - )

Right, I like to automate the stuff. 
