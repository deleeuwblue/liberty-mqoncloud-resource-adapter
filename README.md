# liberty-mqoncloud-resource-adapter
Testing IBM Liberty with MQ Resource Adapter, using MQ on Cloud

This is a tutorial to install, configure and test IBM Liberty, the IBM MQ Resource Adapter and test using an Installation Verification Test with IBM MQ on Cloud.

## Download & Extract IBM Liberty

Download the latest IBM Liberty.  It's important for later in this tutorial to know which JRE will be used.  If installing for Windows or Linux, you can download a package which includes IBM Java.  If installing to Mac (as I am), a JDK for Oracle will be required and you should download the "Web Profile" package. 

https://www.ibm.com/support/pages/websphere-liberty-developers

## Create a Server

/bin/server create servermq

## Download IBM MQ Resource Adapter

The MQ Resource Apdater (RA) allows applications that are running in an application server to access MQ resources.  It implements JCA interfaces and contains the MQ classes for JMS.  This means JMS applications and message driven beans (MDBs) running in Liberty can access the resources of an IBM MQ queue manager.

Follow these steps to download the RA from IBM Fix Central and install into Liberty.  This involves extrating the downloaded archive, referencing it from server.xml and installing the wmqJmsClient-2.0 feature to Libery. 
https://www.ibm.com/docs/en/ibm-mq/9.2?topic=adapter-installing-resource-in-liberty#q128160_

 The server.xml will contain:

    <!-- Enable features -->
    <featureManager>
        <feature>webProfile-8.0</feature>
        <feature>wmqJmsClient-2.0</feature> 
    </featureManager>

     <variable name="wmqJmsClient.rar.location" value="/Users/deleeuw@uk.ibm.com/wlp_21005/wlp/usr/servers/servermq/wmq/wmq.jmsra.rar"/>

You'll need to install the new Liberty feature with:

/bin/installUtility install wmqjmsclient-2.0

## Create an Instance of IBM MQ on Cloud and create MQ Resources



## Install the RA Installation Verification Test (IVT) App

You can use the IVT EAR to test the configuration of the Resource Adapter and MQ on Cloud.

Copy the supplied wmq.jmsra.ivt.ear to the server dropins folder, i.e. /usr/servers/servermq/dropins

Now add the required server.xml configuration for the IVT app.  This is where things get interesting as the official documentation does not include configuration for TLS, which is enabled by default for MQ on Cloud Queue Manager of v9.2.1 and above.

server start servermq

Check the 



