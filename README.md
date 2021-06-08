# liberty-mqoncloud-resource-adapter
Testing IBM Liberty with MQ Resource Adapter, using MQ on Cloud

This is a tutorial to install and configure IBM Liberty with the IBM MQ Resource Adapter, and test a JMS application using an Installation Verification Test, using IBM MQ on Cloud.

## Download & Extract IBM Liberty

Download the latest IBM Liberty.  It's important for later in this tutorial to know which JRE will be used.  If installing for Windows or Linux, you can download a package which includes IBM Java.  If installing to Mac (as I am), a JDK for Oracle will be required and you should download the "Web Profile" package. 

https://www.ibm.com/support/pages/websphere-liberty-developers

## Create a Server

/bin/server create servermq

## Download IBM MQ Resource Adapter

The MQ Resource Apdater (RA) allows applications that are running in an application server to access MQ resources.  It implements JCA interfaces and contains the MQ classes for JMS.  This means JMS applications and message driven beans (MDBs) running in Liberty can access the resources of an IBM MQ queue manager.

Follow these steps to download the RA from IBM Fix Central and install into Liberty.  This involves extrating the downloaded archive, referencing it from server.xml and installing the wmqJmsClient-2.0 feature to Liberty.

https://www.ibm.com/docs/en/ibm-mq/9.2?topic=adapter-installing-resource-in-liberty#q128160

 The server.xml will contain:

    <!-- Enable features -->
    <featureManager>
        <feature>webProfile-8.0</feature>
        <feature>wmqJmsClient-2.0</feature> 
    </featureManager>

     <variable name="wmqJmsClient.rar.location" value="/Users/deleeuw@uk.ibm.com/wlp_21005/wlp/usr/servers/servermq/wmq/wmq.jmsra.rar"/>

You'll need to install the new Liberty feature with:

/bin/installUtility install wmqjmsclient-2.0

## Deploy an Instance of IBM MQ on Cloud & Create a Queue Manager

https://cloud.ibm.com/docs/mqcloud?topic=mqcloud-mqoc_create_qm

Once deployed, click the three dots and note or download the connection details.

![QMconndetails](https://user-images.githubusercontent.com/8861294/121190602-20a60200-c863-11eb-8809-c04a7aa64ea5.png)

## Create a Test Queue

Follow the instructions to use MQ Console to create a local test queue.  Name the queue TEST.QUEUE instead of DEV.TEST.1.

https://cloud.ibm.com/docs/mqcloud?topic=mqcloud-mqoc_admin_mqweb

## Create an Application User for MQ

From the IBM Cloud service defintion page, select the "Application credentials" tab to add a new application user (e.g. "ivtapp"), taking note of its API Key.

![ivtapp](https://user-images.githubusercontent.com/8861294/121043358-71a6ef00-c7ac-11eb-84f6-dccf30128960.png)


## Install the RA Installation Verification Test (IVT) App

You can use the IVT EAR to test the configuration of the Resource Adapter and MQ on Cloud.

Copy the supplied wmq.jmsra.ivt.ear to the server dropins folder, i.e. /usr/servers/servermq/dropins

Now add the required server.xml configuration for the IVT app.  This is where things get interesting as you'll need to configure for TLS which is enabled by default for MQ on Cloud Queue Manager of v9.2.1 and above.

Add the following sections to server.xml, using the connection details for your MQ Queue Manager.  

<!-- IVT Connection factory -->
<jmsQueueConnectionFactory connectionManagerRef="ConMgrIVT" jndiName="IVTCF">
   <properties.wmqJms channel="SSL.SVRCONN" 
     hostname="qmivt-09ca.qm.us-south.mq.appdomain.cloud" 
     port="30932"
     queueManager="QMIVT"  
     transportType="CLIENT" 
     userName="ivtapp" 
     password="<ivtapp user API key>"
     sslCipherSuite="TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256"/>
</jmsQueueConnectionFactory>
<connectionManager id="ConMgrIVT" maxPoolSize="10"/>

<!-- IVT Queues -->
<jmsQueue id="IVTQueue" jndiName="IVTQueue">
   <properties.wmqJms baseQueueName="TEST.QUEUE"/>
</jmsQueue>

<!-- IVT Activation Spec -->
<jmsActivationSpec id="wmq.jmsra.ivt/WMQ_IVT_MDB/WMQ_IVT_MDB">  
    <properties.wmqJms destinationRef="IVTQueue" 
      transportType="CLIENT" 
      queueManager="QMIVT" 
      channel="SSL.SVRCONN"
      hostName="qmivt-09ca.qm.us-south.mq.appdomain.cloud"
      userName="ivtapp" 
      password="<ivtapp user API key>" 
      port="30932" 
      maxPoolDepth="1"
      sslCipherSuite="TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256"/>
</jmsActivationSpec>


When creating the Queue Manager with MQ on Cloud, the default cipher spec is "ANY_TLS12_OR_HIGHER", therefore the sslCipherSuite for the RA is set accordingly to "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256".  Note that the JRE dictates the appropriate values for sslCipherSuite.  On Mac, the only option is Oracle JRE but for othe platforms an IBM JRE is available.  See the following link for the equivalent values for IBM JRE.  For Windows with an IBM JRE you could use "SSL_ECDHE_RSA_WITH_AES_128_CBC_SHA256" instead
https://www.ibm.com/docs/en/ibm-mq/9.2?topic=jms-tls-cipherspecs-ciphersuites-in-mq-classes

Also be aware that the MQ classes in the RA will assume an IBM JSSE provider unless the System Property com.ibm.mq.cfg.useIBMCipherMappings is set to false.  See this blog for a great explanation of the relationship between MQ CipherSpecs and Java Cipher Suites:

https://community.ibm.com/community/user/integration/viewdocument/bitesize-blogging-mq-version-8-the?CommunityKey=183ec850-4947-49c8-9a2e-8e7c7fc46c64&tab=librarydocuments

As I am testing on Mac with an Oracle JRE, I created file jvm.options in the same folder as server.xml, and set the following System Property:

-Dcom.ibm.mq.cfg.useIBMCipherMappings=false

It's also necessary to import the public certificate for the Queue Manager into the trust store used by Liberty.  The MQ on Cloud documentation provides instructions on creating a JKS format keystore for all platforms (the tools required are different for Mac):

https://cloud.ibm.com/docs/mqcloud?topic=mqcloud-mqoc_configure_chl_ssl#mqoc_chl_ssl_keystore_jks

The required certificate can de dowloaded from MQ on Cloud.

![QMcert](https://user-images.githubusercontent.com/8861294/121169601-13c9e400-c84c-11eb-8b80-1292fe82091d.png)

As I am working with Mac, the command to create the JKS key store is as follows:

keytool -importcert -file qmgrcert.pem  -alias qmgrcert -keystore <trustStoreName>.jks -storepass <trustStorePassword>
 
I stored the resulting key.jks file to /usr/servers/servermq/resources/security.  The default key store format for Liberty is PKCS12, but JKS can also be used.  I added the following line to server.xml to reference the previously created JKS key store:
 
 <keyStore id="defaultKeyStore" location="key.jks" password="" type="JKS"/>
 
 
## Setting User Permisions in MQ on Cloud
 
Before testing the IVT app, additional permissions need to be granted to MQ resources for the application user (ivtapp).

Using MQ Console, use the filter icon to "Show system queues", then set permisions on the **SYSTEM.DEFAULT.MODEL.QUEUE** and **TEST.QUEUE** by clicking the three dots, then "View Configuration", then "Security" to grant all permissions for user "ivtapp".
 
![ivtapppermissions](https://user-images.githubusercontent.com/8861294/121187138-a627b300-c85f-11eb-8ec0-f6ea8c59b7bd.png)
 

## Testing the IVT App 

Start the server:

server start servermq
 
Access the web app at http://localhost:9080/WMQ_IVT and press the Run IVT button.  The result should be as follows:
 
![ivtresults](https://user-images.githubusercontent.com/8861294/121187843-644b3c80-c860-11eb-9b1d-568eec463acf.png)

The JMS application is able to use the resource definitions in server.xml to connect to the Queue Manager on IBM Cloud, using the appropriate application user and TLS configuration.
 
You will notice there is a warning "Receiving response message from the MDB... failed to receive response message!".  It's not clear what causes this issue and IBM MQ support are planning to look into it.  However, the main purpose of this tutorial was to demonstrate the configuration required for using the RA with Liberty.
