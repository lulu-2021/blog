---
title: building c# with albacore
date: 2014-05-03 09:30 EST
tags:
---


## Albacore, fun and easy build tool for .NET

#### xml, ms build and the likes are no fun whereas Albacore's ruby dsl makes for a great build tool for .NET projects

Over the last 2 years I have been doing more work with Ruby, Rails when it comes to building web apps and also building some
infrastructure automation in my day time job using [chef](http://www.getchef.com/chef/). Since I am not a huge fan of XML and
avoid it when possible, I came accross [Albacore](http://albacorebuild.net/) - which is a suite of [Rake](http://rake.rubyforge.org/)
tasks with a nice DSL to very quickly put together build scripts for .NET projects. Even if you are not a seasoned Ruby developer
the amount of Ruby required to complete the build related tasks for .NET in Albacore is learned really quickly.

The basic requirement to get started with Albacore is to install Ruby. Windows support for Ruby is getting better. Most of my usual
Ruby dev work is done in OSX and I use RVM to be able to have project based versioning on the Ruby side, you can't go passed this one.. There is a somewhat more limited third party tool called [Pik](https://github.com/vertiginous/pik) which does all you need to get started with building .NET with Ruby.

Create a `Gemfile` in the root of the project like this sample here:

	 source 'https://rubygems.org'

	 gem 'albacore', '0.3.5'
	 gem 'rubyzip', '0.9.9'

Run `bundle` or `bundle install` to get the relevant gems installed. I had some issues with newer versions of the Albacore gem.

Then create a `Rakefile` in the root of your project which has all the build, test and publish related steps in it.

At the start of the build script - you will typically configure the environment related stuff:

	#
	ENVIRONMENT = "Debug"
	#ENVIRONMENT = "Release"
	#
	BUILD_VERSION = 1001
	BUILD_DIR = "build"
	SOLUTION_NAME = "ManageAzure"
	PUBLISH_BASE = "c:/publish"
	PUBLISH_DIR = "#{PUBLISH_BASE}/#{SOLUTION_NAME}"
	SOLUTION = "#{SOLUTION_NAME}.sln"
	PROJECT_DIR = "ManageAzure"
	CONFIGURATION_OPT = "Debug"
	#CONFIGURATION_OPT = "Release"

Since I also use xunit for my unit tests in C# projects - you will need to configure the runner..

	#
	XUNIT_CONSOLE = "./Packages/xunit.1.9.2.runner/xunit.console.clr4.exe"
	UNIT_TEST_PROJECT = "ManageAzureTests"
	UNIT_TEST_ASSEMBLY = "./#{UNIT_TEST_PROJECT}/bin/#{ENVIRONMENT}/#{UNIT_TEST_PROJECT}.dll"

Then the actual `build` and `test` steps are really simple Ruby code blocks:

	#
	desc "Run All Unit Tests"
	xunit :unittest do |xunit|
	    xunit.command = XUNIT_CONSOLE
	    xunit.assembly = UNIT_TEST_ASSEMBLY
	end
	#
	desc "Run the build step - in this case clean and build"
	msbuild :build do |cmd|
		cmd.solution = "#{SOLUTION}"
		cmd.targets = [:Clean, :Build]
		cmd.properties = {:Configuration => CONFIGURATION_OPT}
	end

Finally you can chain steps into rake tasks to make things easier:

	task :test => [:build, :unittest] do
	end


Here is a [Sample](https://github.com/netflakes/AzureManagement/blob/master/RakeFile) build `Rakefile` for a recent project of mine.

The Albacore [Wiki](https://github.com/Albacore/albacore/wiki) has all the detailed information to get started.  When it comes to doing up a build script for .NET - this is so much more fun than fiddling with XML descriptors etc.

Enjoy!