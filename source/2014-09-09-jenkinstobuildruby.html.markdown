---
title: JenkinsToBuildRuby
date: 2014-09-09 19:42 EST
tags:
---

## Jenkins, Continuous Integration Server and Ruby

#### Jenkins is really easy to install on a vps and the configuration of building ruby is a snap

I have recently started my own business, building a Web Application with Ruby and Rails. I use [Capistrano](ttp://capistranorb.com/) for automated build deployment. Since I develop using a TDD approach, I was keen to have a continuous deployment process to the cloud up and running from the start.

In my last job I used Windows Azure extensively and for comfort reasons decided to fire up a few Ubuntu virtual machines for a continuous build and deployment process. I have three virtual machines, one running [Jenkins](http://jenkins-ci.org/) as a build server, one application server and one database server. I use [Bitbucket](https://bitbucket.org/) for Git source control. 

The basic flow - each time code is deployed to the master branch in Git, this triggers a build in Jenkins and in turn all unit and integration tests are run. On succes a second project in Jenkins does a full capistrano based automatic deployment to the Azure environment. This gives me a daily fresh build of all the work done.

I don't check in yml configuration files such as the database.yml and other such files into source control. They are just symlinked to shared configuration folders both in my local dev and in the deployment environments. For the Jenkins build process a simple shell script generates these files on the fly during the build process. 

The jenkins configuration also allows you to 

		Build when a change is pushed to Bitbucket
		Run the build in a RVM-managed environment (simple add your ruby/gemset here!)


The shell script below is the meat of the build (sensitive bits overwritten with ##..)

	#!/bin/bash -x

	unlink config/database.yml
	unlink config/redis.yml
	unlink config/services.yml

	export RAILS_ENV=test

	bundle install

	read -d '' database_yml <<"EOF"  
	login: &login  
	  adapter: postgresql
	  encoding: unicode
	  username: #########
	  password: #########
	  host: #########

	test: &test  
	  database: ###web_test
	  <<: *login
	EOF

	echo "$database_yml" > config/database.yml  

	read -d '' redis_yml <<"EOF"
	default:
	  host: localhost
	  port: 6379
	test:
	  db: 1
	production:
	  db: 2
	EOF

	echo "$redis_yml" > config/redis.yml

	read -d '' services_yml <<"EOF"

	services: &services
	  mail_api_baseurl: http://localhost:3001
	  mail_api_current_version: api/v1
	  ###_api_baseurl: http://localhost:3002
	  ###_api_current_version: api/v1
	  #####_api_baseurl: http://localhost:3003
	  #####_api_current_version: api/v1

	development:
	  <<: *services

	test:
	  <<: *services

	production:
	  <<: *services

	EOF

	echo "$services_yml" > config/services.yml


	rake db:create db:test:prepare  

	bundle exec rspec spec/models

	bundle exec rspec spec/fast

	bundle exec rspec spec/controllers

The second project that does the actual deployment from Jenkins has these two simple shell scripts that handle the Capistrano deployment - nice and simple and it works:

One to do the actual prep work..

	#!/bin/bash -x
	export RAILS_ENV=production
	bundle install

And the second to do the deploy..

	eval `ssh-agent`
	echo $SSH_AGENT_PID > agent.pid
	ssh-add
	bundle exec cap production deploy


Apart from the actual Capistrano deploy script that is pretty much it!

Enjoy.

