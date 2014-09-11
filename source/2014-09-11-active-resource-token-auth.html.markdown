---
title: active-resource-token-auth
date: 2014-09-11 17:30 EST
tags: ruby, active-resource, token-authentication, ngnix
---

## Active Resource, Token Auth and Nginx

#### Active Resource works really nicely with Rest Api from Rails

I am currently building a reasonably large web application that consumes several rest api based data sources. Each of these are integrated into the main application and joined by a company id that is guid based. Each api sits on the inside of the network at the moment but might not future growth. The system will authenticate with each api using random generated tokens. These are added to the request headers as well as the company id to ensure the correct scoping for each api call.

I have a base class for each of the services that requires authentication and here I set the auth token and the company id for each request. 

	module RestServices
	  class ServiceBase < ActiveResource::Base

	    cattr_reader :api_token, :company_id
	    cattr_accessor :static_headers

	    self.static_headers = headers

	    def self.headers
	      # - override the headers to ensure we always have the request based/current ones
	      new_headers = static_headers.clone
	      new_headers["Authorization"] = set_token_header
	      new_headers["company_id"] = set_company_header
	      new_headers
	    end  
		
	    def self.set_token_header
	      "Token token=#{@@api_token}"  
	    end  
		
	    def self.set_company_header
	      @@company_id.to_s
	    end  
		
	    def self.authorise(token, company_id)
	      @@api_token = token
	      @@company_id = company_id
	      #
	      # - raise an appropriate error if we don't have the right credentials!
	      check_valid_authentication
	    end  
		
	    private
	    def self.set_credentials?
	      check_api_token? && check_company?
	    end
		
		# check_api_token AND check_company - business logic commented out here..
	    def self.check_valid_authentication
	      # - we should raise an error to be caught at the application level of the credentials have not been set to ensure service calls include a valid token in the header
	      raise ApplicationErrors::InvalidCredentialError unless set_credentials? 
	    end
	  end    
	end  
	


In each rest api controller you then just need a before filter to ensure the request headers are populated:

    prepend_before_filter :require_login
    before_filter :enable_service_authenticator

And the authorise method: How you obtain the token and the company id etc.. is up to your business logic..

    def enable_service_authenticator
      Resource.authorise(resource_api_token, current_company.id)
    end  
	

One issue that I had with nginx is that by default it removes any custom headers that have an underscore in it - took me a while to figure this one out. Here is the (nginx docu on configuring custom headers)[http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers]

After that I was all set with token auth against the rest api resources. The final piece was to add some rescue_from statements into the ApplicationController with relevant responses to gracefully handle connection issues.._

    # - Catch all situations where we called for a record that does not exist
    rescue_from ActiveResource::ResourceNotFound, :with => :rescue_not_found

    # - Catch all situations where the services api-s are down
    rescue_from Errno::ECONNREFUSED, :with => :rescue_service_unavailable

Enjoy!

