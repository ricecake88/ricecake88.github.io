---
layout: post
title:      "Trials and Tribulations - CLI Gem Project"
date:       2018-06-01 15:10:49 -0400
permalink:  trials_and_tribulations_-_cli_gem_project
---


**Picking a project**

As a huge lover of movies and one that is constantly looking up information on film, I thought I would attempt to make my own CLI project based on upcoming films.

At first, like any other student, you start dreaming of all the features you want in your project. Me? I wanted to be able to look up actors in upcoming releases, which distributor distributed the film, and what genre the film was going to be, with all the information of run time, a blurb on what the movie was - the sky was going to be the limit!

After watching the CLI Gem Walkthrough with Avi, I knew I had to wittle down my project completely. I wanted all kinds of features. Whenever I work on coding projects myself, I tend to work on the parts that don't matter. In this instance, the scraper. I loved scraping and being able to grab the information I wanted off a webpage, so I started with that first, even prior to watching the CLI Gem Walkthrough or starting the OO Student Scraper lab. That wasn't the smartest idea.

**Watching the video helped guide me**

The most important take-away from watching the video is something I'm going to have to change in how I approach my projects. I have to first work on basic functionality before jumping into something that would impede my development later. I shouldn't have worked on the scraper. I should have set out the basic functionality of what I wanted in my CLI program, as Avi had outlined for his Daily Deal. Dealing with loads of data in the beginning would have been too much to code with, and so I felt that stubbing out information to make sure your Classes work seemed to be the most logical step.

Outlining what I wanted my CLI program to do and how it would interact with the user was the very first step.
- Spit out a menu for the user upon running the program with a list of options
- This menu would give the user the following options
 1. an option of all the upcoming movies being released this month
 2. ask the user if they want to choose a movie they want to know more about which will print out information about the film
 3. an option of listing all the upcoming movies for the rest of the year
 4. an option of listing all the actors with upcoming movies for this month
 5. an option of listing all the actors with upcoming movies for the rest of the year
 6. a submenu of listing more information about a certain actor
 7. ability for user to quit the program

If time permitted I would give the option to the user to list certain genres of movies and distributors. (Later note: I didn't have enough time, but want to revisit this!)

**The difficulties with deciding which page to scrape**

The IMDB "coming soon" page organizes their movies by date but leave off the year. I made the mistake of scraping the month and date from this page, leaving the year empty which posed a problem. Scraping the page for the date and storing the date for each movie was already a challenge given that the date was on the same tree level as the movie information, which required a bit of creativity. I expand on this problem further down the page.

The solution was to scrape the individual movie profile page for the release date, but I felt it required a reworking of getting the data into the movie database, and nothing else in my project was accomplished yet so I left that as pending to-do item while I decided that changing my stub actor strings into Actor Objects was a higher priority.

**Disaster in "How Many Through"**

Trying to model the concepts of our Song, Artist, Genre assignment in earlier labs, I had designed my CLI program to have a movie instance add itself everytime it scraped a new actor. Basically:

```
a = Actor.new("name_of_actor")
movieInstance.actors << a
if !a.movies.any?{|movie| moveInstance.name ==movie.name}
      a.movies << self
end
```

However, instead of the expected order of :
```
<Movie Object #id1 
@name="Sleepless in Seattle"
@actors=<Actor Object #id1 @name="Meg Ryan" @movie=[<Movie Object  #id1 @name="Sleepless in Seattle"
                                    @actors=[<Meg Ryan Object>, <Tom Hanks Object>]
        <Actor Object #id2 @name="Tom Hanks" @movie=[<Movie Object #id1 @name="Sleepless in Seattle"
                                    @actors=[<Meg Ryan Object>, <Tom Hanks Object>]
@date = _release_date_of_Sleepless_in_Seattle_
>
```

I got something like this:
```
<Movie Object #id1 
@name="Sleepless in Seattle"
@actors=<Actor Object #id1 @name="Meg Ryan" @movies=[<Movie Object  #id1 @name="Sleepless in Seattle" 
                                    @actors=<Meg Ryan Object>] <-- this is missing other actors
        <Actor Object #id2 @name="Tom Hanks" @movies=[<Movie Object #id1 @name="Sleepless in Seattle"
									@actors=[<Meg Ryan Object><Tom Hanks Object>]
@date = _release_date_of_Sleepless_in_Seattle_
>
```

That would prove to be disastrous and just wrong when I looked at a single movie object. The movie information would be wrong everytime a new Actor was created. It would have a list of all the previous actors that were added but none of the ones after it. It would have been fine if I had just decided to add the movie.name to the movie array belonging to the Actors, but it's another story when you're adding to an array of Movie Objects where it relates to back to the actor.

After thinking it through, I felt the best solution was to create all the actors while I scraped each actor names, but not add the array of actors until all the scraping of the cast was retrieved.  So the new code looked like this:

```
if releaseDate != nil && movie != nil 
    m = UpcomingMovies::Movie.new({:name=>movie, :month=>month, :date=>date,
            :year=>"2018", :url=>profile_url })

   @@actors.each do |actor|
         m.add_actor(actor)
   end
end
```

**Refactoring the Date**

When I first started scraping the page, I had unfortunately assumed that I had to get all the data on one page. This was before I started OO Student Scraper lab. Retrieving dates proved to be complex if I stuck with just the "Coming Soon" page.

The format for organizing the movies on the page was:
<p>
-June 1<br/>
-Movie 1<br/>
-Movie 2<br/>
-Movie 3<br/>
-June 8<br/>
-Movie 1<br/>
-Movie 2<br/>
</p>

*Problem 1*: June 1 in the CSS was on the same level in terms of hierarchy as the Movies it listed which made it harder to store the date in the movie instances, but I cobbled it together, making sure I had a value for a date AND a value for the movie before creating a new Movie instance. I felt it was definitely hacking information together

*Problem 2*:  How in names was I going to figure out what year the date corresponded to? So I knew I would have to change this code and grab the date off of the profile page of the movie, where it was listed, which meant reworking my code. To temporarily solve my problem so I could work on other things, I had hard coded the year to be "2018", and I knew if I wanted to scrape the rest of the pages, I would need to have a real year being stored.

My cobbled together code so all movies would have a release date:
```
class UpcomingMovies::Scraper

    BASE_PATH = "https://imdb.com/"
    def scrape_upcoming(url)
        puts "Please wait a few minutes, retrieving data..."
        imdb_url = open(url)
        imdbDoc = Nokogiri::HTML(imdb_url)
        titles = imdbDoc.css("div.list.detail h4")

        releaseDate = nil
        month = ""
        date = ""
        movie = nil
        titles.each do |node|

            if node.attributes["class"] == nil
                movie = "#{node.text}"
                profile_url = node.children[0].attributes["href"].value
        
            elsif node.attributes["class"].value == "li_group"
                releaseDate = "#{node.text}"
                dateInfo = releaseDate.split(" ")
                month = dateInfo[0]
                date = dateInfo[1].gsub(/\u00A0/, "")
            end

          if releaseDate != nil && movie != nil 
            m = UpcomingMovies::Movie.new({:name=>movie, :month=>month, :date=>date,
                 :year=>"2018", :url=>profile_url })

            @@actors.each do |actor|
                m.add_actor(actor)
            end
          end
        end
         
    end
```

Specifically this section:
```
        titles.each do |node|

            if node.attributes["class"] == nil
                movie = "#{node.text}"
                profile_url = node.children[0].attributes["href"].value
        
            elsif node.attributes["class"].value == "li_group"
                releaseDate = "#{node.text}"
                dateInfo = releaseDate.split(" ")
                month = dateInfo[0]
                date = dateInfo[1].gsub(/\u00A0/, "")
            end

          if releaseDate != nil && movie != nil 
            m = UpcomingMovies::Movie.new({:name=>movie, :month=>month, :date=>date,
                 :year=>"2018", :url=>profile_url })

            ...
						
          end
        end
```
		
**Slowly adding to the Movie Class**

After having populated the actors array with Actor objects,  and being able to retrieve movies of actors and actors with movies, I could move on by getting more attributes for the movies. That meant getting the description, the rating and genres. As a future todo, I wanted to store the genres into a Genre Class, but felt that I should move forward in refactoring my code first since I didn't think I would have time to finish writing up Genres (because, spending way too much time on a project is a real thing!)

## Refactoring CLI Class

In my CLI class, I was doing too many calculations in order to display movies in a specified time frame (week, current month or all). Knowing that the CLI class is essentially the "View" in MVC which should only be concerned with showing the data and not working with generating data, I had to move the methods that dealt with retrieving what date was the next Friday and what month we were currently in elsewhere. I did not feel like it belonged anywhere in my existing code.

It had two problems:
1. Contained code that didn't have anything to do with CLI - **#current_month_year**, **#date_of_next_friday** and **#futureMovie**?
2. **#listmoviesmonth** and **#list_movies_week** did not simply just print out the list of movies, it still processed and operated on the data to decide which list to print

Ugly code.. yes?


```
    def date_of_next_friday
        date  = Date.parse("Friday")
        delta = date > Date.today ? 0 : 7
        [(date + delta).strftime("%m"), (date+delta).strftime("%d"), (date+delta).strftime("%Y")]
    end

    def current_month_year
        monthNo =  Date.today.strftime("%m")
        currentYear = Date.today.strftime("%Y")
        monthString = @@months.key(monthNo)
        [monthString, currentYear]
    end

    #class method that checks whether or not the movie release date is in the future
    def futureMovie?(year, month, date)
        strDate = year + "-" + @@months[month] + "-" + sprintf("%02i", date)
        newDate = Date.parse(strDate)
        today = Date.today
        if newDate >= today
            return true
        else
            return false
        end        
    end

    def list_movies_week
        @movies_week = []
        date = date_of_next_friday
        UpcomingMovies::Movie.all.each_with_index do |movie,index|
            day = sprintf("%02i", movie.date) 
            if @@months[movie.month] == date[0] && day == date[1] && movie.year == date[2]
                puts "#{index} #{movie.month} #{movie.date} #{movie.name}"
                @movies_week << movie.name
            end
        end
        puts "Select a movie by choosing a number associated with the movie listed."
        input = gets.strip.to_i
        if !(input > @movies_week.length || input < 0)
           puts @movies_week[input]
        else
            puts "Error"
        end
    end

    def list_movies_month
        current_info = current_month_year
        UpcomingMovies::Movie.all.each do |movie|
            if movie.month == current_info[0] && movie.year == current_info[1]
                puts "#{movie.month} #{movie.date} #{movie.name}"
            end
        end
    end

    def list_movies
        UpcomingMovies::Movie.all.each do |movie|
            if futureMovie?(movie.year, movie.month, movie.date)
                puts "#{movie.month} #{movie.date} #{movie.name}"
            end
        end
    end
```
		

But where could I move it to? I knew I could write class methods to return the list of upcoming movies in the timeframe I wanted in the Movie class. But the methods of **#current_month_year** and **#date_of_next_Friday** didn't seem like it belonged in the Movie class either. They were helper methods. This resulted in the idea of creating a Helper module to store the the helper methods, and I could call them from any class, though it would mostly be used by the Movie class in my program.

Resulting Helper Module:
```
module Helper
    @@months = {'January'=>'01', 
        'February'=>'02', 
        'March'=>'03', 
        'April'=>'04', 
        'May'=>'05', 
        'June'=>'06', 
        'July'=>'07', 
        'August'=>'08', 
        'September'=>'09', 
        'October'=>'10', 
        'November'=>'11', 
        'December'=>'12'}

    def date_of_next_or_this_friday
        date  = Date.parse("Friday")
        delta = date >= Date.today ? 0 : 7
        [(date + delta).strftime("%m"), (date+delta).strftime("%d"), (date+delta).strftime("%Y")]
    end

    def current_month_year
        monthNo =  Date.today.strftime("%m")
        currentYear = Date.today.strftime("%Y")
        monthString = @@months.key(monthNo)
        [monthString, currentYear]
    end

    def self.months
        @@months
    end
end
```
After moving the methods to the Helper Module and moving **#current_month_year** and **#date_of_next_Friday** class methods to the Helper module, I decided I could further compact listing the movies into one method and pass the curated arrays to the methods as opposed to having multiple methods in the CLI class that processed the data, resulting in more readable code that would only be tasked on doing "one thing".

```
def list_movies(movie_array)
		if movie_array.length != 0
				movie_array.each_with_index do |movie, index|
						puts "#{index+1}. #{movie.month} #{movie.date} #{movie.name}"
				end
				sub_movie_menu(movie_array)
		else
				puts "There are no upcoming movies for this time frame."
		end
end
```

Much better than the above example right? After organizing my list_movies method, I felt the same could be done with listing actors and simplified the methods down to the following:
```
	def list_actors(actor_array)
			if actor_array.length != 0
					actor_array.sort! do |x,y|
							x.name <=> y.name
					end
					actor_array.uniq.each_with_index do |actor, index|
							puts "#{index+1}. #{actor.name}"
					end
					sub_actor_menu(actor_array)
			else
					puts "There are no actors with upcoming movie releases in this time frame."
			end
	end
```

**End.. but never the end**

The program was finally at a place where I was satisfied with it. I knew I could spend a lot more time on the project if I wanted to and would hopefully come back to it sometime in the future. But for the purposes of my project, since the timeline for part-time students is 7 days, I felt like I didn't want to dwell too much about it since it's easy to go down the rabbit hole and spend months on it. Knowing that whatever I would learn in the future would help improve on what I have here and potentially move away from a command line into a web interface, I decided to stop and ask for an evaluation.

I scheduled a 1:1 session to get the opinion of a technical coach to see if what I had was sufficient enough to submit my code and to give me some feedback and see if my project was at a point where it was good enough to submission. Turns out it was! Thanks Dakota! After a few changes, and fixing up my README instructions and gem information, I think I'm ready to submit!

**Things I would like to improve upon:**
1. Calculate for all the possible movie releases next week, not only Friday.
2. Allow the submenu to keep on asking whether or not a user wanted to look at another movie or actor's information before quitting out of the menu, until the user inputted "q" to quit out of the submenu.
3. Place the genres into its own Genre classes
4. Give a list of available genres for the user to choose from, then ouput the movies of that genre by the choice of the user.
5. Give a list of distributors for the user to choose from, then output the movies of that distributor by the choice of the user. 
- This item I thought would be easy to incorporate, but I have to scrape a child link from the movie profile page in order to retrieve this information, and search for the USA distributor, as IMDB lists all international distributors as well.
6. After learning more, I'd like to make the scraping to be a little more efficient. Right now it is taking too long to scrape all the information.
7. Scrape more than just the month of  "Coming Soon" releases. I was originally going to do all the pages listed in IMDB, but due to how long it took to scrape just the first page, I decided I would hold off on this idea, otherwise it would take possibly 10 minutes or more to scrape all the information, if scraping the one page was taking between 1-2 minutes. 
