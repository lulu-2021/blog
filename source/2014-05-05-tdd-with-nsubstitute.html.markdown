---
title: tdd with nsubstitute
date: 2014-05-05 18:01 EST
tags: c#, tdd, mock
---

## Testing each step in isolation with NSubstitute

#### NSubstitute is a great way to abstract each dependency away from the method being tested

I use [Xunit](https://github.com/xunit/xunit) for unit testing each of layer of abstraction during development. I recently started
to work with a framework called [Nancy](http://nancyfx.org/) - the guys who developed this framework call it the - super-duper-happy-path - of web development. Am beginning to feel the same way as I work my way through all the docu and work with the framework it is really cool. One of the things I really enjoy is the testability. Nancy was developed using TDD and it makes
mocking and testing each individual piece of the pie super simple.

[NSubstitute](http://nsubstitute.github.io/) is a really nice way of doing all the required mocks to ensure you are testing only the actual method/process you want to test!

This is a sample of testing a module in Nancy. Modules here are what Controllers are for traditional MVC. Nancy tries to imitate the
small scale framework called [Sinatra](http://www.sinatrarb.com/) in the world of Ruby. I have used this too - its great to get
small apps and webservices up and running really quickly with a small amount of code. It just gets out of your way. This is the
way of Nancy too, and yet its very flexible in terms of stuff you can add on as and when needed.

Here I am testing a route that adds a new user - during this process the UserService and the UserRepository objects are both required
and yet they are tested separately in other unit tests. Providing you have got proper Interfaces for your Service and Repository layers, NSubstitute will just abstract them away for you and allow you to quickly pre-set what results will be returned from the methods of the mocked objects - cool and really easy.

	[Fact]
    public void TestAddingAnewUser_whereUserDoesNotExist_WithHtmlForm() 
    {
        var testUser = CreateTestUser();
		
		/// mock the User Repository and all methods required for this test
        IUserRepository userRepo = Substitute.For<IUserRepository>();
        userRepo.Add(Arg.Any<User>()).Returns(true);
        userRepo.IsUnique(testUser.UserIdentifier).Returns(true);			
		
		// mock the User Service and the methods required for this test
        IUserService userService = Substitute.For<IUserService>();
        userService.CreateUser(Arg.Any<User>()).Returns(testUser.UserId);
        
        TestBrowser = BrowserSetup(userRepo, userService);
		
        // run the actual test
        var response = TestBrowser.Post(UsersUri, (with) =>
        {
            with.Header("Accept", TextHtml);
            with.HttpRequest();
            with.FormValue("UserIdentifier", testUser.UserIdentifier);
            with.FormValue("UserName", testUser.UserName);
            with.FormValue("PrescriberNumber", testUser.PrescriberNumber.ToString());
            with.FormValue("Fax", testUser.Fax);
            with.FormValue("Mobile",testUser.Mobile );
            with.FormValue("Telephone", testUser.Telephone);
            with.FormValue("Address1", testUser.Address1);
            with.FormValue("Address2", testUser.Address2);
            with.FormValue("Suburb", testUser.Suburb);
            with.FormValue("State", testUser.State);
            with.FormValue("Postcode", testUser.Postcode);
            with.FormValue("PostalSame", testUser.PostalSame.ToString());
            with.FormValue("PostalAddress1", testUser.PostalAddress1);
            with.FormValue("PostalAddress2", testUser.PostalAddress2);
            with.FormValue("PostalSuburb", testUser.PostalSuburb);
            with.FormValue("PostalPostcode", testUser.PostalPostcode);
            with.FormValue("PostalState", testUser.PostalState);
            with.FormValue("LastUpdated", testUser.LastUpdated.ToShortDateString());
        });
        // and check for a proper result
        response.ShouldHaveRedirectedTo(UsersUri + testUser.UserId);
    }

As I work my way through building an API based app - I will add more updates on this.
