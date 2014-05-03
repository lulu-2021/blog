---
title: working with tinyioc
date: 2014-05-02 23:54 EST
tags:
---

## Inversion of Control with TinyIoC


#### Dependency Injection is made really simple with TinyIoC and for smaller projects

[TinyIoC](https://github.com/grumpydev/TinyIoC/wiki) is really easy to use the setup is simplified and it is great to provide
depencies to internal libraries and swap these out for different purposes. In building my Abstraction layer around the new
Azure Management Client library to manage & report on Azure Cloud Services I used [TinyIoC](https://github.com/grumpydev/TinyIoC/wiki) to inject the Logging and the configuration as in ability to provide the
Azure subscriptionId and the Base64 Encoded Management certificate. In my case I am reading both of these from the downloaded
PublishSettings files for each subscription.

The library I am working on is [here](https://github.com/netflakes/AzureManagement/blob/master/ManageAzure/Lib/AzureManagement.cs)
and here is how the dependencies are injected with TinyIoC:

You have a `Bootstrap` class where the depencies are `registered` with the Container. By using the `Interface` we are effectively
telling the IoC container to fire up a `singleton` based instance - i.e. the only one ever running in the application.

	public static class Bootstrap 
	    {
	        public static void Register(string settingsFile)
	        {
	            IMlogger mLogger = new Mlogger();
	            IAppConfiguration appConfig = new ApplicationConfiguration(settingsFile);
	            TinyIoCContainer.Current.Register<IMlogger>(mLogger);
	            TinyIoCContainer.Current.Register<IAppConfiguration>(appConfig);
	        }
	    }

Then when the application is started:

    static void Main(string[] args)
    {
        Bootstrap.Register(AzurePublishSettingsFile);
        var application = TinyIoCContainer.Current.Resolve<AzureManagement>();

You let the container do the actual dependency resolution and it takes care of building the objects with the right constructor.

In the case of the `IAppConfiguration` dependency I have an interface that determines what methods the class provides to the outside world - i.e. the two requirements for the Azure Management Client - which needs the `SubscriptionId` & the `Base64EncodedManagementCertificate`.

    public interface IAppConfiguration 
    {
        string SubscriptionId();
        string Base64EncodedManagementCertificate();
    }

This is then implemented in my case as: (remembering the bueauty of DI is that this could then be changed later with any impact on the library that uses the data provided by this object/interface)


    public class ApplicationConfiguration : IAppConfiguration
    {
        PublishData AzurePublishData { get; set; }
		
        public ApplicationConfiguration(string settingsFile)
        {
            AzurePublishData = ReadAzurePublishSettingsFile(settingsFile);
        }
		
        public string SubscriptionId()
        {
            return AzurePublishData._Profile._Subcription.Id;
        }
		
        public string Base64EncodedManagementCertificate()
        {
            return AzurePublishData._Profile._Subcription.ManagementCertificate;
        }
		
        /// <summary>
        /// Deserialises the Azure Publish Settings File into a tree of objects that hold the Subscription Id and the Base 64 encoded Management Certificate
        /// </summary>
        /// <param name="settingsPath">Full path and file name of the Azure Publish Settings File</param>
        /// <returns>A PublishData object containing the Subscription Id and the Management Certificate</returns>
        private PublishData ReadAzurePublishSettingsFile(string settingsPath)
        {
            TextReader reader = null;
            try
            {
                XmlSerializer deserializer = new XmlSerializer(typeof(PublishData));
                reader = new StreamReader(@settingsPath);
                object obj = deserializer.Deserialize(reader);
                PublishData XmlData = (PublishData)obj;
                return XmlData;
            }
            catch (InvalidOperationException ioe)
            {
                var message = String.Format("Error Reading the Azure Publish Settings File{0}", ioe);
                throw ioe;
            }
            finally
            {
                reader.Close();
            }
        }

Following on from this theme - my next step with this library will be implement local storage and reporting processes that will also be injected at startup so that they can be swapped out and also developed independantly of changes made to the library that is providing the data.

More to follow soon.
