---
title: CapistranoDeployProcess
date: 2014-09-09 20:59 EST
tags: ruby capistrano unicorn
---

## Capistrano, Automatic Deployment to Staging and Prod

#### Capistrano is the de-facto ruby deployment process toolbox. 

I use RVM extensively for all my ruby and rails projects and also use it in my staging and deployment environments. Capistrano works well with it and here is a set of samples of how I do my deployments. This sample is from a rest-api that is part of one of my recent projects. The project name is changed to "sample".

All the basic info can be found on the [Capistrano](http://capistranorb.com) website. 

First up my CAPFILE: (the rvm1/capistrano3 gem works really well for dealing with RVM deployments)

     1 # Load DSL and Setup Up Stages
     2 require 'capistrano/setup'
     3 
     4 # Includes default deployment tasks
     5 require 'capistrano/deploy'
     6 
     7 # Includes tasks from other gems included in your Gemfile
     8 #
     9 # For documentation on these, see for example:
    10 #...
    16 #
    17 require 'rvm1/capistrano3'
    18 require 'capistrano3/unicorn'
    19 #...
    27 # Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
    28 Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }


I use RVM and setting the correct ruby and gemset worked really well..

The next set of code snippets are all from my config/deploy.rb

	# - Configuration settings for the deployment process
	#
	set :rvm_ruby_version, 'ruby-2.1.1@sampleapi'
	set :rvm1_ruby_version, 'ruby-2.1.1@sampleapi'
	#
	set :default_env, { rvm_bin_path: '~/.rvm/bin' }
	SSHKit.config.command_map[:rake] = "#{fetch(:default_env)[:rvm_bin_path]}/rvm ruby-#{fetch(:rvm_ruby_version)} do bundle exec rake"
	#
	# config valid only for Capistrano 3.1
	lock '3.2.1'
	#
	set :application, 'sampleapi'
	#
	#default_run_options[:pty] = true
	#
	set :repo_url, 'git@bitbucket.org:sample-user/sampleapi.git'
	#

Remote cache is much faster..

	# Deply via remote cache
	set :deploy_via, :remote_cache
	#
	# Default branch is :master
	set :branch, "master"
	# ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

Set the deploy folder

	# Default deploy_to directory is /var/www/my_app
	set :deploy_to, '/opt/www/sampleapi'

We use GIT

	# Default value for :scm is :git
	set :scm, :git


	# Default value for :format is :pretty
	set :format, :pretty

Turn on debug log level if you have a failed deploy to figure out what went wrong

	# Default value for :log_level is :debug
	#set :log_level, :debug
	set :log_level, :info
	#
	# SSH connections made to non standard port
	set :ssh_options, {
	  config: false,
	}
	#

This ensures any RVM updates and gem updates are always considered in each deploy step

	before 'deploy', 'rvm1:install:rvm'
	before 'deploy', 'rvm1:install:ruby'
	before 'deploy', 'rvm1:install:gems'
	#
	# - These two lines are for when  Pubkey Auth is not enabled - i.e. prompts for the password
	#set :password, ask('Server password', nil)
	#server 'sample.cloudapp.net', user: 'deployer', port: 2122, password: fetch(:password), roles: %w{web app db}
	#

We use public key auth for SSH - the two lines above allow you to use password auth instead

	# - This is for when Pubkey Auth is enabled..
	server 'samplecloudapp.net', user: 'deployer', port: 2122, roles: %w{web app db}
	#
	# Stages in our case is just set to production
	set :stages, ["production"]
	#
	# Default value for :pty is false
	# set :pty, true
	set :pty, true

Here we make sure the yml config are not removed with each deploy - since they are symlinked from a shared folder

	# Default value for :linked_files is []
	# This means that the database.yml and redis.yml files are not get blatted with each new deploy!
	#
	set :linked_files, %w{config/database.yml config/redis.yml}
	#
	# Default value for linked_dirs is []
	set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}
	# Default value for keep_releases is 5
	set :keep_releases, 5
	# touch assets etc - to ensure they are refreshed with each deploy during development
	#set :normalize_asset_timestamps, %{public/images public/javascripts public/stylesheets}

This api uses unicorn as the webserver..

	# set some unicorn values
	set :unicorn_pid, '/opt/www/sampleapi/shared/pids/unicorn.pid'
	#
	set :unicorn_config_path, '/opt/www/sampleapi/current/config/unicorn.rb'
	#
	set :environment, "production"
	#

This is the final set of customised steps to link our configurations files, migrate the DB if required and 
recompile all the assets with each deploy.

	namespace :deploy do
	  desc 'assets and db migration'
	  task :dbmigrate do
	    on roles(:app) do
	      execute "cd #{fetch(:deploy_to)}/current/"
	      execute "cd #{fetch(:deploy_to)}/shared/config"
	      within "#{fetch(:deploy_to)}/shared/config/" do
	        execute "ln -sfn #{fetch(:deploy_to)}/shared/config/database.yml #{fetch(:deploy_to)}/current/config/database.yml"
	        execute "ln -sfn #{fetch(:deploy_to)}/shared/config/redis.yml #{fetch(:deploy_to)}/current/config/redis.yml"
	      end
	      execute "cd #{fetch(:deploy_to)}/current/" # just so we are in the app folder
	      within "#{fetch(:deploy_to)}/current/" do
	        with RAILS_ENV: fetch(:environment) do
	          execute :rake, "db:migrate"
	          execute :rake, "assets:precompile"
	        end
	      end  
	    end
	  end  

And finally restart UNICORN..

	  desc 'restart unicorn'
	  task :restart do
	    invoke 'unicorn:restart'
	  end  

	  after :publishing, :dbmigrate
	  after :dbmigrate, :restart

	end

With this you should have a working automatic deployment..
