---
layout: post
title:      "Frustrations With Ruby, Shotgun, and the Rabbit Hole Down Windows"
date:       2018-12-30 21:03:09 -0500
permalink:  frustrations_with_ruby_shotgun_and_the_rabbit_hole_in_windows
---


I'm a very lazy person by nature. Out of the seven sins, the clear sin that I ascribe to is "sloth". I enjoy tinkering with code, but whenever I come to a part where I may not enjoy that much - it takes forever for me to do.

Having encountered a previous lab where "shotgun" on my Windows desktop ran into a few issues, I immediately went back to using the Learn Online IDE, because, I hate dealing with set up issues. I did not want to deal with figuring out what the problem was or how to work around it. However, having this always at the back of my mind, I knew I would have to deal with it when it came to setting up my Sinatra project.

Well, here we are. And I have dragged my feet in getting my environment set up because I knew it wouldn't just be a couple hours issue to solve and in general, I reeeeally hate setting up workspace environments. I always prefer getting into the thick of it (which is the coding and design). For me, setting up libraries, or in this case, compatible gems are my least favorite part, though necessary.

I started off copying and pasting the Gemfile from the Sinatra CRUD lab since I couldn't remember what the resulting error was. Created app folder with MVC folders and config.ru, config/environment.rb based on the CRUD lab.

```
source 'http://rubygems.org'

gem 'activerecord', :require => 'active_record'
gem 'sinatra-activerecord', :require => 'sinatra/activerecord'

gem 'sinatra'
gem 'pry-nav'
gem 'rake'
gem 'rspec'
gem 'rack-test'
gem 'database_cleaner', git: 'https://github.com/bmabey/database_cleaner.git'
gem 'require_all'


group :development do
  gem "capybara"
  gem "pry"
  gem "sqlite3"
  gem "shotgun"
end
```

**Shotgun Error, the Error That Started the Rabbit Hole**

I performed a "bundle install" and then entered "shotgun" in the command line. Lo and behold I received the same error from when I had attempted to run shotgun on my desktop:

```
C:/Ruby24-x64/lib/ruby/gems/2.4.0/gems/shotgun-0.9.2/bin/shotgun:142:in `trap': unsupported signal SIGQUIT (ArgumentError)
        from C:/Ruby24-x64/lib/ruby/gems/2.4.0/gems/shotgun-0.9.2/bin/shotgun:142:in `block in <top (required)>'
        from C:/Ruby24-x64/lib/ruby/gems/2.4.0/gems/shotgun-0.9.2/bin/shotgun:141:in `each'
        from C:/Ruby24-x64/lib/ruby/gems/2.4.0/gems/shotgun-0.9.2/bin/shotgun:141:in `<top (required)>'
        from C:/Ruby24-x64/bin/shotgun:23:in `load'
        from C:/Ruby24-x64/bin/shotgun:23:in `<main>'

```

When I visited this error last, I quickly came to the conclusion that shotgun was not supported in Windows due to the fork() command not being recognized in Windows. This time around, after googling for more answers, I came to the same conclusion. I was a bit frustrated that there was no mention of this in any of the labs that shotgun would not work on Windows machines and what a workaround could be. It would have saved me so much more time.

**Sinatra Reloader, Rerun, Rackup and Rake**

At this point in time, I just removed the shotgun gem, and installed a few other gems - sinatra-reloader and rerun, both promising to be able to do the same as shotgun. Well, not quite.

Several times I just wiped the slate clean by deleting ALL the gems using the following command (thanks to Stackoverflow!):
```
ruby -e "`gem list`.split(/$/).each { |line| puts `gem uninstall -Iax #{line.split(' ')[0]}` unless line.empty? }"
```
Then I reinstalled the bundler gem and installed all the gems by calling bundler install (as per the advice on stack overflow)

After finding out that sinatra-reloader is now under the gem 'sinatra-contrib', and that it worked if I used rackup with sinatra-reloader, it seems as though if I refreshed the browser, it would update without having to use sinatra. However, that joy was short lived since I then ran into issues with running rake, which was necessary for creating sql statements and creating databases (a very important aspect of the Sinatra Project!). I got this error:
```
$ rake
rake aborted!
NoMethodError: undefined method `task' for Sinatra::Application:Class
```

**Sinatra-Contrib and Thin**

This error then prompted its own bit of googling which told me that it appears that there is an error with the latest version of sinatra-contrib, and I have to use ' 1.3.1', which leads me to have to use sinatra 1.3.0. This lead me to look into exactly what the gem 'thin' was (which I think shotgun is built on top of it) as I saw it being mentioned in one of the comments, suspecting I might be able to run a localhost using thin. I decided to enter a thin command in my command prompt to see if it would serve up the code on my desktop, which resulted in another error:

```
$ thin -R config.ru -a 127.0.0.1 -p 8080 start
Unable to load the EventMachine C extension; To use the pure-ruby reactor, require 'em/pure_ruby'
Unable to load the EventMachine C extension; To use the pure-ruby reactor, require 'em/pure_ruby'
C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- 2.4/rubyeventmachine (LoadError)
```

**Eventmachine Mingw32 Error**

I looked it up and there was a conflict and x64-mingw32 was missing something in the Event Machine that was loaded with x-64mingw32. It was then recommended to install eventmachine separately with the following command:
```
gem install eventmachine --platform ruby
```
and then to uninstall the x64-mingw32 version.

**Pry**

This seemed to appear to run okay until I created a very simple shell of a program. I found it was not really possible to run pry without errors. This would not be acceptable when trying to work. I needed pry. Or tux. But neither worked. When using git bash, the enter key refused to be recognized. Frustrated, I tried using the Windows Powershell. I got further using it with pry, but it would not recognize the "=" character. The same behaviour is seen using "cmd". I started to wonder if it was my git shell program that did not have the required libraries to run the gems, so I decided to install cygwin.

**Installing cygwin**

I kept on running out of space when I attempted to install cygwin. Since I wasn't sure exactly which libraries I needed in order for cygwin to run ruby and install the gems, I chose installing python, ruby, development, libs, but I ran out of space quickly that the installation itself started becoming corrupted after I tried installing the libraries I believed I needed. At first I had ran a very simple cygwin with just basic libraries, but found that if I tried to do a "bundle install" it would complain about needing certain libraries. I would then search for it in the list of libraries for the specific file, and found it took a really long time searching for each item to figure out which component needed to be installed that I decided to install the entire library. However, once that happened, I just, unfortunately, ran out of space. I was at a bit of a loss.

**Back to the Learn IDE**

At this point, it had been several days, almost a week of trying to get ruby, shotgun, pry working in any sort of command shell on my Windows Desktop which lead me back to questioning whether I could just use the Learn IDE application instead of using a standalone IDE of my choice. Installing the gems and trying to find gems that would work with one another was taking far too long, and seeing as I was working full time and often overtime, I just didn't have the time to fiddle with gems any further. I decided to fire up Learn IDE, cloned my repository, pushed my current state to a separate branch, then changed the Gemfile to include all the default gems all my previous Sinatra projects, installed the gems, ran shotgun, and voila, it worked. Which meant that I could probably use tux and pry as well without much of a problem. I wished I had used it from the beginning to save some time, but I had been hoping I could just code using my own setup. Ah well, maybe for the next project I'll be able to figure it out. I'll chalk all of this up to a learning experience.
