---
layout: post
title:  "Các step viết E2E test"
date:   2016-06-16 15:20:00
summary: Giới thiêu các step cơ bản để viết E2e test.
categories: [web testing]
tags: ["E2E"]
images: 
author: TrucDT
---
### Configuration
__-Config in Gemfile:__

  gem "selenium-webdriver" 
  gem 'turnip' 
  gem 'capybara', '~> 2.2' 
  gem 'poltergeist'

__-Config in spec_helper.rb:__

    require 'turnip' 
    require 'turnip/capybara' 
    require 'capybara/dsl' 
    require 'capybara/poltergeist' 
    require 'capybara/rspec' 
    require 'selenium-webdriver'

    Dir.glob("spec/steps/*_steps.rb") { |f| load f, true } : all steps will be defined in files ended with '_steps.rb', located in folder spec/steps/
    
    +Config so that  poltergeist driver will be used with phantomjs:
    Capybara.register_driver :poltergeist do |app| 
      Capybara::Poltergeist::Driver.new(app, { phantomjs: 'phantomjs', 
                                               js_errors: false, 
                                               debug: false, 
                                               phantomjs_options: ['--load-images=no', '--disk-cache=true'], 
                                               timeout: 360 }) 
    end 
    Capybara.run_server = false 
    Capybara.default_driver = :poltergeist 
    Capybara.javascript_driver = :poltergeist 
    Capybara.ignore_hidden_elements = false 

__-Config in rails_helper.rb:__

    require 'spec_helper'
    require 'rspec/rails'

__-Config in .rspec file:__

    --color --format documentation -r turnip/rspec 
    --require spec_helper

__-Install PhantomJS:__

    Download binary file from http://phantomjs.org/download.html
    tar -xvjf phantomjs-2.1.1-linux-x86_64.tar.bz2
    sudo mv phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/local/bin
    phantomjs -v

__-Steps are defined in *_steps.rb:__

    step 'access :site' do |site| 
      Capybara.app_host = site 
    end
    
     step 'visit :path' do |path| 
      visit path 
    end
    
    step 'click :tag' do |tag| 
      if xpath?(tag) 
        first(:xpath, tag).click 
      else 
        first(:css, tag).click 
      end 
    end
    
    xpath? is defined in /spec/supports/link_helper.rb in order to check if a string a xpath or css
    
    module LinkHelper 
      def xpath?(path) 
        return true if path.include? '//' 
        false 
      end
    +To use the module LinkHelper, you should delare as following in spec_helper.rb:
    require_relative './supports/link_helper.rb'

    step 'response code is :code' do |code| 
      expect(page.status_code).to eq(code.to_i) 
    end
    
    step 'current url contains :url' do |url| 
      expect(URI.parse(current_url).request_uri.to_s).to match url 
    end

__-Create feature files:__

    Create feature files with above steps (Ex:spec/features/test.feature):
    
    Feature: Apply 
      Background: 
        Given access 'https://kangokyujin-ex.ventura.test:9090/'
    
    Scenario: Apply a job without logging in Kango: register->confirm->finish 
      When visit "/area/13/occupation/3" 
      And click "(//a[@class='btn_red w_250px f_right ml_10'])[1]" 
      Then response code is "200" 
      And current url contains "/entry/new/" 

__-Run the auto test:__

    bundle install
    rspec featurefile --format documentation
    Ex: rspec spec/features/test.feature --format documentation
    The auto test will run and return the result pass or fail
