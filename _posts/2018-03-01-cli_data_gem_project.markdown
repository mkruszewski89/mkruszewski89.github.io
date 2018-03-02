---
layout: post
title:      "CLI Data Gem Project"
date:       2018-03-02 03:26:52 +0000
permalink:  cli_data_gem_project
---


The first portfolio project in my Flatiron School curriculum is to build an application that provides a Command Line Interface (CLI) to an external data source. My education and early career featured the financial industry, and an interest in markets has persisted. This CLI will interface to Google Finance and allow users to enter a ticker to receive financial data. As a new developer, I am not wild about "black boxes" that provide functionality without my understanding, and seeing as this is my first from-scratch project, this post will be extremely detailed. This post focuses on interactions and structure, I will not get into the details of the actual code beyond presenting it for all to see.

I begin this project at the user's entry point: the executable. I want the executable to be as simple as possible, so I will just have it instatiate a new instance of an object and call a function that encapsulates all of the run logic:

in the command line:
```
mkdir market-data-cli-gem 
cd market-data-cli-gem
mkdir bin && touch bin/market-data
```

in 'market-data' add:
```
#!/usr/bin/env ruby

CLI.new.call
```


My task now is to create all the pieces that will allow this to run without error.

I create the CLI class and think about how I initially imagine the program running. It should ask the user for a ticker and allow them to predictably exit. If a ticker is entered, it should display data (dummy data is fine for now):

in the command line:
```
mkdir lib && touch lib/cli.rb
```

in 'cli.rb' add:
```
class CLI

  def call
    input = nil
    while input != 'exit'
      puts "Enter a ticker symbol or 'exit':"
      input = gets.strip.downcase
      display
    end
    puts "Goodbye!"
  end

  def display
    puts <<-DOC
      ticker | name | price | today_change
    DOC
  end

end
```

At this point I will stop to ensure everything is working as expected:

in the command line:
```
market-data-cli-gem/bin/market-data
```
returns:
```
Traceback (most recent call last):
./bin/market-data:4:in `<main>': uninitialized constant CLI (NameError)
```

The executable does not know about the CLI class...of course it doesn't, I never told it! I imagine there will be a lot of file dependencies in this app, so I will create a file to keep track of it all. I will then have my executable require this reference file.

in the command line:
```
mkdir config && touch config/environment.rb
```

in 'environment.rb' add:
```
require_relative '../lib/cli'
```

in 'market-data' add:
```
require_relative '../config/environment'
```

Now the program executes, but the real work of making it useful is just beginning. To replace the dummy data in CLI#display I will need to scrape the live web. Such a task merits its own encapsulation, so I create a Scraper class.

in the command line:
```
touch lib/scraper.rb
```

in 'environment.rb':
```
require_relative '../lib/scraper'
```

in 'scraper.rb':
```
Class Scraper

end
```

To interact with the web I will need some outside help.  A Gemfile is a single place for gems to be required and versions specified. Bundler helps set this up.

in the command line:
```
bundle init
```

in the newly-created 'Gemfile' add:
```
source "https://rubygems.org"

gem 'pry'
gem 'nokogiri', '1.8.2'
```

in 'environments.rb' add:
```
require 'bundler'
Bundler.require
require 'open-uri'
```

in the command line:
```
bundle install
```

Now I can use nokogiri and pry to find the data I need from the URL passed into Scraper#scrape_ticker_page. This can be a frustrating and time consuming exercise. Ultimately this method should return a hash of the desired data for us to use elsewhere.

in 'scraper.rb' add the class method:
```
  def self.scrape_ticker_page(input)
    url = "https://finance.google.com/finance?q=" + input
    doc = Nokogiri::HTML(open(url))
    ticker_data = {}
    ticker_data[:ticker] = # find the correct selector
    ticker_data[:name] = # find the correct selector
    ticker_data[:price] = # find the correct selector
    ticker_data[:day_change] = # find the correct selector
    ticker_data
  end
```

Now I want to bring this functionality to the CLI. As things are getting more concrete, some re-factoring is in order to ensure we remain in-keeping with best design practices. I realize that the ticker being displayed merits its own encapsulation, and displaying will require some helper methods. I also realize I can use a case statement within a loop to better plan for the future. I refactor as follows:

in 'cli.rb' modify so the file reads:
```
class CLI

  def call
    loop do
      puts "Enter any ticker symbol or 'exit':"
      input = gets.strip.downcase
      case input
      when 'exit'
        break
      else
        display_ticker(input)
      end
    end
    puts "Goodbye!"
  end
	
	def display_ticker(input)
    ticker = create_ticker(input)
    ticker.display
  end

  def create_ticker(input)
    ticker_data = Scraper.scrape_ticker_page(input)
    Ticker.new(ticker_data)
  end

end
```

in the command line:
```
touch lib/ticker.rb
```

in 'environment.rb' add:
```
require_relative '../lib/ticker'
```

in 'ticker.rb' add:
```
class Ticker
  attr_accessor :ticker, :name, :price, :day_change

  def initialize(ticker:, name:, price:, day_change:)
    self.ticker = ticker
    self.name = name
    self.price = price
    self.day_change = day_change
  end

  def display
    puts "#{ticker}  |  #{name}  |  #{price}  |  #{day_change}"
  end

end
```

At this point the project works as I initially imagined! Of course, once you get something working you constantly think of ways to add and modify. Thankfully, the way this has been created makes it very amenable to change. 

As an example, let's add the functionality to open the ticker's webpage for more details, and some initial data to exhibit behavior, which might be helpful to a new user:

update 'ticker.rb':
```
class Ticker
  attr_accessor :ticker, :name, :price, :day_change, :url # :url is added

  @@all = [] # line added to hold initial data

  def self.all # method added to access initial data
    @@all
  end

  def initialize(ticker:, name:, price:, day_change:, url:) # url: is added
    self.ticker = ticker
    self.name = name
    self.price = price
    self.day_change = day_change
    self.url = url # line is added
  end

  def display
    puts "#{ticker}  |  #{name}  |  #{price}  |  #{day_change}"
  end

end
```

update 'scraper.rb':
```
class Scraper

  def self.scrape_ticker_page(input)
    url = "https://finance.google.com/finance?q=" + input
    doc = Nokogiri::HTML(open(url))
    ticker_data = {}
    ticker_data[:ticker] = input.upcase
    ticker_data[:url] = url # line added
    ticker_data[:name] = doc.css(".elastic").first.css("h3").first.text
    ticker_data[:price] = doc.css(".elastic").first.css(".pr").text.gsub("\n",'')
    ticker_data[:day_change] = doc.css(".elastic").first.css(".id-price-change").text.gsub("\n",'')
    ticker_data
  end

end
```

update 'cli.rb'
```
class CLI

  def initialize #method added
    Ticker.all << create_ticker("dji")
    Ticker.all << create_ticker("inx")
    Ticker.all << create_ticker("rut")
    Ticker.all << create_ticker("xwd")
  end

  def call
    loop do
      puts "Enter any ticker symbol, 'list' for some examples, or 'exit':" #added 'list' option to prompt
      input = gets.strip.downcase
      case input
      when 'exit'
        break
      when 'list' #case added
        list
      else
        display_ticker(input)
      end
    end
    puts "Goodbye!"
  end

  def list #method added
    Ticker.all.each_with_index {|ticker, index|
      puts "#{index+1}. #{ticker.ticker}  |  #{ticker.name}"
    }
    puts "Please enter a number to see more or 'exit' to go back:"
    input = gets.strip.downcase
    return if input == 'exit'
    input = Ticker.all[input.to_i-1].ticker
    display_ticker(input)
  end

  def display_ticker(input)
    ticker = create_ticker(input)
    ticker.display
    puts "Would you like to view the webpage for more details (y/n)?:" #line added
    input = gets.strip.downcase #line added
    system("open -a Safari #{ticker.url}") if input == "y" #line added
  end

  def create_ticker(input)
    ticker_data = Scraper.scrape_ticker_page(input)
    Ticker.new(ticker_data)
  end

end
```

As you can see, with this design pattern we can focus on the new rather than the old and changes are easy.



