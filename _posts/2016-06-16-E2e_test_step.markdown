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

###-Create a test case:

    To create an auto test case we will do the following steps:
    1.Define steps with Capybara,Ruby in a file named *_steps.rb located at folder spec/steps/ (As above, we declared: Dir.glob("spec/steps/*_steps.rb") { |f| load f, true } in spec_helper.rb so steps we put in spec/steps/*_step.rb wil be loaded automatically when running)
    2.Create feature files with Gerkin syntax from steps defined in step file
    3.Run the feature file with the command line

###Example:###

    Create a test case in Kango:
    +Access link "https://kangokyujin-ex.ventura.test:9090/area/13/occupation/3"
    +Click a button Apply Job
    +In the Apply page, click Submit button without input anything
    +Verify if the error message displayed
    
__1.Set steps in step file__

    Create a file named *_steps.rb at folder spec/steps. Ex:spec/steps/common_steps.rb. In this file we will define some steps as below:
    All steps will have the same structure:
      step 'step name :value' do |value|
        Do something
      end
    :value starting with ':' is considered as a variable which will be shown in detail in feature file.
      
    step 'access :site' do |site| 
      Capybara.app_host = site 
    end
    'access :site': set Capybara.app_host = site because Capybara.app_host will be used for method visit (':site' is considered as variable, so user will set its value in the feature file)
    
    step 'visit :path' do |path| 
      visit path 
    end
    'visit :path': visit path means access the link '#{Capybara.app_host}/path' (':path': a variable will be set in feature file)
    
    step 'click :tag' do |tag| 
        first(:xpath, tag).click 
    end
    'click :tag': click an elemant.':tag' will be the xpath of the element we want to click.To use this step in the feature file, we have to determine the xpath of the element we want to click and replace ':tag' with it.
    
    step 'response code is :code' do |code| 
      expect(page.status_code).to eq(code.to_i) 
    end
    'response code is :code': return True if the response code is correct
    
    step 'response includes :string' do |string|
      expect(page).to have_content(string)
    end
    'response includes :string': return True if the response message includes expected string (it is used to check if the m essage is correct)

__2.Create feature files:__

    Create feature files with above steps.Here is spec/features/test.feature:
    
    Feature: Apply 
      Background: 
        Given access 'https://kangokyujin-ex.ventura.test:9090/'
    
    Scenario:Error message when the required fields are empty
    When visit "/area/13/occupation/3"
    And click "(//a[@class='btn_red w_250px f_right ml_10'])[1]"
    Then response code is "200"
    And click ".//input[@value='確認ページへ']"
    Then response includes "メールアドレスを入力してください。"
    
    Step in Background will be used for all scenarios in the file. The first step is started with 'Given'
    There should be many scenarios in a file, each scenario is corresponding to a test case, including many steps. The first step in a scenario should be started with 'When'. The following steps starts with 'And'.The first verifyng step right after an action step should be started with 'Then'.After that, continues with 'And'.
    
    First of all we have: access 'https://kangokyujin-ex.ventura.test:9090/'. 'https://kangokyujin-ex.ventura.test:9090/' is replaced ':site' in the step "access :site". It means that with all scenario, Capybara.app_host='https://kangokyujin-ex.ventura.test:9090/'.
    
    So with step: visit "/area/13/occupation/3" we will visit 'https://kangokyujin-ex.ventura.test:9090/area/13/occupation/3'."/area/13/occupation/3" is for ':path' of 'visit :path'.
    
    In the page https://kangokyujin-ex.ventura.test:9090/area/13/occupation/3, we want to click an Apply job button by using step "click :tag" We have to get the xpath of the Apply job button for :tag (Install Firebug,Firepath extension of Firefox to get it).Here "(//a[@class='btn_red w_250px f_right ml_10'])[1]" is the xpath of the Apply job button.
    
    After clicking Apply button, we need the response code should be 200. If the response code is 500, test case is failed here. It means that The Apply page is not shown correctly, maybe it's a bug.
    
    If the the page is returned normally. We click the Submit button. In this step we continue to use the step "click :tag" but the ':tag' will be replaced with the xpath of the Submit button. So it becomes: And click ".//input[@value='確認ページへ']". Because of clicking Submit without inputting anything, the error message should be displayed.To verify it we use the step 'response includes :string'.In this case ':string' will be "メールアドレスを入力してください。"
    
__3.Run the auto test:__

    In the first time running the test, we should run: bundle install to install the required gem
    After finishing all steps and scenario, we can run with this command:
    rspec featurefilename --format documentation
    In this case, it will be: rspec spec/features/test.feature --format documentation
    The auto test will run and return the result pass or fail
