---
layout: post
title:      "Javascript Project Roadblocks"
date:       2021-03-11 03:07:45 -0500
permalink:  javascript_project_roadblocks
---


I have ruminated for a while on what to write on in regards to my Rails-JS project. It took me more than a year to complete my Habit tracker, which is down right pitiful, most of which I actually completed in the last two weeks. Nothing like a hard deadline like the program I'm in ending in June to light a fire right at my behind! 

Due to the length it took me to complete, I think instead of exploring anything technical in this post, I will write this as a way to offer advice to other programmers and developers, with my most important advice:

## For Every Road Block You Encounter, Write Down How You Solved It. 

Did I follow this advice? No.

I'm not speaking of everyday things we (okay, maybe just me) forget like "how does regex work again?" I'm speaking of road blocks that we spend hours trying to figure out, and often are system integrational issues that are very very hard to solve, and have to piece information from various sources to get your environment up and running. 

This is especially difficult when a lot of tools (especially on Learn) are for Mac users and Unix users, but I am a Windows user, and there seems to be less resources from a student perspective, which takes a lot of Stackoverflow and plenty of tabs to find the answers. 

Please, document as much of what you learn. Laziness often gets in the way (as for me), or especially when one is on a roll and just wants to "get on with it", after they've spent hours, days, maybe even weeks to get past their road block. But I guarantee that it will be helpful for you in the future and save you time instead of having to go googling down a rabbit hole to figure it out again. 

Keep a blog, keep it in notes, even pen and paper, whatever your medium, just document it (and remember where you stored it!)


## My Roadblocks


### Changing from SQLite to Postgres SQL

When I first started working on the project and  was reading over the project instructions, I noticed that some projects were hosted on heroku. I thought, why not try and figure out how to host it on heroku? I read that instead of SQLite, they used Postgres SQL, so I decided to try and figure out how to get Postgres SQL up and running so I could use it as my database to build upon, making it easier to host on heroku once I was finished as opposed to having to convert it later on.

This took me a long time, and was very discouraging. This didn't really have anything to do with actual rails or javascript, and yet, it took me forever to get it set up. Unfortunately, I remember very little of how exactly how I got it running. I had localhost issues, postgres issues, incompatibilties with gems, the list went on. To save me time next time, I should be able to remember how I did all of this right? No.... because I didn't write it down. It was so long ago, I remember little of how I managed to get it up and running.

When I finally got it working I tried pushing it to heroku but was faced with lots of errors, but decided to forego putting my project on heroku as a result.  After all, it isn't one of the project requirements, and I can go back and figure out how to do it at a later time. I was afraid it would take a very long time (and, considering how long I took, that probably was a good idea)!

### User Authentication and JSON Web Tokens:

I got stuck trying to figure out how to authenticate a user using the js/html/rails interface, as I couldn't get sessions to work the way I wanted to. The recommendation was to use JSON web tokens from one of the educational coaches, but, as a STRETCH goal. Well, instead of a stretch goal, it ended up being one of the very first things I worked on. (Also, not very smart).

Authentication is hard. It has always been the area where I have the most difficulty time and time again, and this time it was no different. I stagnated in this area for months, and found it hard to motivate myself to work very long when I just couldn't figure out how to authorize my user. Online guides show how to integrate JSON web tokens in a very basic setup. 

Once I finally figured out how to Authenticate and Authorize my user to login, I had to figure out how to continue to make requests to the server allowing myself to stay authenticated and authorized, and realized that the way to continue to send requests to the server was to attach the authorization token to the configuration file in the headers. I had to search deep into the help forums to find this, as it wasn't obvious to me how to get the information passed through and none of the web tutorials mentioned having to include it in all of the configs sent to fetch. Now, looking back it makes sense, but at the time I was fairly lost, and it got me fairly frustrated. I was moving through this portion like a snail.

### Time

No, not the amount of time it took me to complete -- but time, like UTC time, date time. I was giving myself a mental roadblock because I just couldn't figure out an elegant way of how to show progress on a habit. Ideas I had originally thought - marking off a date on a calendar seemed too difficult for me to design, and I didn't want to spend extra time on something that wasn't required, nor did I just want to use a separate gem. I finally resigned myself to the current design I have, which was letting users see a specific range of marked off habits by way of colored boxes. For a course project, it didn't need to be professional, so I finally accepted that the way I had designed it would be sufficient.

### Re-Organization / UI

This wasn't so much a road block, but took a long time to do. I basically wrote my front end, and then refactored the entire thing after I was done to clean up my code, meaning most functions were assigned to a "thing", as opposed to multiple "things". I've learned this as a programmer that it is better to write very verbose code that you can optimize later, than to try and write optimized code from the beginning. It is easier to abstract repeated elements when you can see what your code is doing than to make an abstraction as you go along, because it is harder to debug when the application isn't fully fleshed out.

However, it was a challenge to figure out where some of the rendering of some of my habit views should go. For example, the summary draws upon the data from both Habit and HabitRecord, but there is no modifications allowed, and is only used as a display of data. I wasn't sure where that should have went. As a newbie to javascript, I hope that as I continue on the journey of learning about javascript, I will have a better idea of how to better organize frontend code, whereas I feel backend code has better structure that lends itself to being more RESTful than frontend.

As for UI, I've always been a stickler of what accounts for good UI, and there are still a few things that I feel could be improved on with the existing way my habit tracker is structured with its current components, not accounting for some of the stretch goals I never got to. The work is never done! But, I'll shelve it, write some todos later on and come back to it once I've graduated, me thinks.


# Summary

All in all, the bulk of my project - the actual coding part was completed in the majority of the last couple weeks, every waking moment I wasn't working, eating or exercising. I'm exhausted now, but I'm proud of what I was able to accomplish given a short period of time. It's reminded me why I enjoyed web design when I was a young teenager back in high school, and now, I'm ready and looking forward to what I can do with React!
