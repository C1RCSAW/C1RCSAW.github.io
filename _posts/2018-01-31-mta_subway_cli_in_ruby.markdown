---
layout: post
title:      "MTA Subway CLI in Ruby"
date:       2018-02-01 01:25:14 +0000
permalink:  mta_subway_cli_in_ruby
---

Ok here we go first project in programming thats not a pre configured lab. We needed to come up with our own project for this one, develop a command line interfacing program written in Ruby. I can at least remember the days of MS DOS on computers from childhood before PCs had a GUI operating system where programs needed to be evoked from the prompt. Ha, other than entering the .exe command for my favorite games that was about the extent of my early memories of the command line. With all this previous experience and just begining to comprehend the principles of object oriented programming, I was ready to build the next market disrupting app! Not really, I was still grasping at the begining of understanding class objects, taking a little on faith and moving forward to learn more with experience. It really is worth it to struggle with this concept, try to grasp it at several angles, this will lead you to one of many rewarding "AH HA!' moments when learning to write code! As an example, Avi in his videos and lessons warns coders about the "zipper" pitfall. In an earlier iteration of my project I had successfully scraped all the different attributes for the different subway lines I wanted to use in my project, Line.name, Line.status, and Line.details however these attributes werent connected to their corresponding Line class object, they didnt seem to be sticking around and were separate items unrelated to each other when I was trying to recal the different attributes in IRB. At the time my instance method *#self.get_lines* looked somthing along the lines like this:

def self.get_lines
    (0..get_names.size-1).to_a.each do |i|
      line_name = get_names[i]
      line_status = get_status[i]
      line_details = "http://www.mta.info/status/subway/#{get_names[i]}"
		end
		  all
end

It seems obvious to me now but at the time I was several days, hours and minutes in that I had lost some field of vision. What I was missing from the method was a way to persist my Line class objects in memory and recall them later for the CLI interface. Eventually, through reviewing previous lessons and video lectures and knd help from the Learn community on Slack (no question is too dumb or silly, seriously we are all eager to help without judgement!), I honed in on this final method.

def self.get_lines
    (0..get_names.size-1).to_a.each do |i|
      line_name = get_names[i]
      line_status = get_status[i]
      line_details = "http://www.mta.info/status/subway/#{get_names[i]}"

      line = self.new
      line.name = line_name
      line.status = line_status
      line.details = line_details
      line.save
    end
    all
  end
	
	the additional segement of code added to the loop would evoke a new Line object on each iteration of the loop and assign it attributes using the scrapper methods I had written for the class as well as save each occurance of a line object using my save class method to retain the collection of Line objects in an array for use later in the CLI. 
	
this is just one example of where I woud reach what I thought was a dead end but eventually would find a different direction or solution throug consulting not only the material from learn and ruby docs but also the larger ruby community. the world of ruby gems is populated with many lovely bits of code to add to your programs as well as consult as examples to see other programmers coding style. a simple example is "colorize" which allows one to add some text color to your ruby CLI for some design points. I also came across a similar repository of github that helped me solve some of my scraping issues that scraped a site using xml tags rather than css tags.  I couldnt seem to get drilled down to where i wanted using just css tags. As I learn more I imagine I could even come back and utilize more efficient css selectors. so instead of using doc.css() on the Nokogiri object I used doc.xpath("//name") for instance to scrape the names for all the subway lines. 

To explore more of the code I came up for this project check out
		

https://github.com/C1RCSAW/nyc_subway_status
