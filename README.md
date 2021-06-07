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

## Deploy an Instance of IBM MQ on Cloud & Create a Queue Manager

https://cloud.ibm.com/docs/mqcloud?topic=mqcloud-mqoc_create_qm

Once deployed, click the three dots and note or download the Connection Details.

## Create a Test Queue

Follow the instructions to use MQ Console to create a local test queue.  Name the queue TEST.QUEUE instead of DEV.TEST.1.

https://cloud.ibm.com/docs/mqcloud?topic=mqcloud-mqoc_admin_mqweb

## Create an Application User for MQ

From the IBM Cloud service defintion page, select the "Application credentials" tab to add a new application user (e.g. "ivtapp"), taking note of its API Key:

![ivtapp](https://user-images.githubusercontent.com/8861294/121043358-71a6ef00-c7ac-11eb-84f6-dccf30128960.png)


## Install the RA Installation Verification Test (IVT) App

You can use the IVT EAR to test the configuration of the Resource Adapter and MQ on Cloud.

Copy the supplied wmq.jmsra.ivt.ear to the server dropins folder, i.e. /usr/servers/servermq/dropins

Now add the required server.xml configuration for the IVT app.  This is where things get interesting as the official documentation does not include configuration for TLS, which is enabled by default for MQ on Cloud Queue Manager of v9.2.1 and above.

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


When creating the Queue Manager with MQ on Cloud, the default cipher spec is "ANY_TLS12_OR_HIGHER", therefore the sslCipherSuite for the RA is set accordingly to "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256".  Note that the JRE dictates the appropriate values for sslCipherSuite.  On Mac, the only option is Oracle JRE but for othe platforms an IBM JRE is available.  See the following link for the equivalent values for IBM JRE:
https://www.ibm.com/docs/en/ibm-mq/9.2?topic=jms-tls-cipherspecs-ciphersuites-in-mq-classes


server start servermq

WORK IN PROGRESS





