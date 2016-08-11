# Xively SDK for Android

This is the Xively Android SDK. It is designed to help facilitate building an Android application using Xively Services ( Idm, Blueprint, TimeSeries, Messaging ).

## Table of Contents

1. Introduction
	- Supported Xively Services
	- Contents

2. Getting Started
	- Prerequisites
	- Install Xively SDK
	- Run XivelySDKDemo
	- Run XivelySDKE2E

3. API Reference

4. Tutorials
	- Using the Android SDK
	- Login
	- List Devices
	- List Organizations
	- List End Users
	- Get End User
	- Get Device
	- Update End User
	- Update Device
	- Send a Message
	- Receive Messages
	- Graph TimeSeries Data

## Introduction

### Supported Xively Services

The SDK currently supports using the following services:

 * Username/Password based authentication with Xively Identity Management(IdM) API
 * Querying organization or organization list from Blueprint
 * Querying device or device list from Blueprint
 * Querying end user or end user list from Blueprint
 * Updating end user in Blueprint
 * Updating device in Blueprint
 * Querying time series records of channels
 * Messaging through the publish-subscribe style (MQTT) interface

### Package structure:

* XivelySDKAndroid.aar - the Xively SDK archive
* [javadoc] - API documentation
* [XivelySDKAndroid] - the source code of the library
* [XivelySDKDemo] - visual demo application
* [XivelySDKE2E] - sample application showing all features, can be used for end-to-end tests
* CHANGELOG - release history
* COPYING - Copying and redistributing legal info
* LICENSE - License Agreement
* README.md - project readme document


## Getting Started

This section will help you get the Xively Android SDK setup in your environment.
 
### Prerequisites

#### SDK

The SDK is provided as an android archive library and as source.

#### Development Environment

This guide assumes that the developer using the Xively SDK is familiar with Android Studio and Android development basics.
The expected minimum Android API level of the Xively SDK is 15 (Android 4.0.3).

This guide assumes that you have a Xively account already setup, if you don't, [click here](https://www.xively.com/get-started).
It also assumes that you have 'account user' credentials as they will be needed to facilitate the basic connection.

### Install Xively SDK

#### Dependencies and Third Party Content

There are no external dependencies, however the library is based on and uses the following open source software:

* Paho Java client for MQTT - http://eclipse.org/paho/clients/android/
* Google gson - http://code.google.com/p/google-gson/
* Retrofit - http://square.github.io/retrofit/

#### 1. Download the SDK

If you haven't already check out the sdk or just download the aar file [from here](https://github.com/xively/xively-mobile-sdk-android).

#### 2. Import the Library

Copy the xively sdk library aar to your project (create an [aars] or any other folder in your project).
To import the library, open your application module settings (F4) and open the Dependencies tab. You may add the library as a file dependency, but you can also create a local maven repository for your library dependencies.

A typical build.gradle should look like this after the previous steps :

```gradle
repositories {
        flatDir {
            dirs 'aars'
        }
    }
    
dependencies {
    compile(name: 'xively-sdk-0.1.1', ext: 'aar')
    }
```

And the library is ready to use.

#### 3. Run XivelySDKE2E

If you checked out the full source you can quickly test the SDK in action with the XivelySDKE2E module. It has one activity, the E2EActivity class.
In E2EActivity's onCreate function you have to write in your Xively credentials :
```
String xivelyUsername = "your xively username";
String xivelyPassword = "your xively password";
String xivelyAccountId = "your xively account id";
```

After this you can build and run the project. It covers all API functionality, E2EActivity's source is a very good starting point for your own application.

#### 3. Run XivelySDKDemo

This module is a visual demo application for the Xively SDK. The only thing you need to do is adding your xively account id to MainActivity's xivelyAccountId member :
```
private String xivelyAccountId = "your xively account id";
```
After building and running the module you will see a possible usage scenario for the SDK.


## API Reference

The API Reference is included in the [javadoc] folder of the downloaded package. This is an autogenerated document in html format. Look for the *index.html* to open the document's Overview page.

## Tutorials

This section provides brief code snippets that would help someone add the desired Xively functionality - such as message publish and subscribe, authentication etc. - to their existing application.

### Using the Android SDK

Before starting to use any Xively SDK APIs, the embedding application is expected to perform authentication.
The authentication service is the single entry point of the Xively SDK. It can  be instantiated using the *XiAuthenticationFactory* factory methods.
It is mandatory to pass the embedding application's Context object to the authentication service factory methods:

```java
 XiAuthentication auth = XiAuthenticationFactory.createAuthenticationService(context);
```

The application context is required to initialize the SDK and request Authentication.

Optionally you can also specify a custom configuration if you would like to customize connection related properties or the internal log level of the Xively SDK. For details see the corresponding javadoc.
By creating a new instance of the *XiSdkConfig*, the object's properties will reflect the current values of the SDK configuration.
All the fields are public and can be modified, however the values set in your custom configuration object will become effective only if you pass it to the SDK in your authentication service request:

```java
XiSdkConfig customConfig = new XiSdkConfig();
//Set custom configuration values
customConfig.mqttDisconnectTimeout = 2000;
customConfig.logLevel = XiSdkConfig.LogLevel.OFF;
//...

//pass the custom configuration object to the authentication service
XiAuthentication auth =
        XiAuthenticationFactory.createAuthenticationService(context, customConfig);
```

### Login

Everything in the Xively Android SDK happens in the context of a user.
The first thing that must be done is to initializethe library and call the login function.

Before you can access the Xively SDK services a user must be authenticated.
Authentication is handled by the *com.xively.auth.XiAuthentication* service which can be instantiated using the *XiAuthenticationFactory*.

End users and Account users can authenticate using their Xively credentials and the implementing application has to provide their organization's Xively Account Id.
Authentication results are returned asynchronously using the *XiAuthenticationCallback* instance passed to the authentication request.

```java
 auth.requestAuth("endUser@email.com", "user password", "Xively Account Id",
		 new XiAuthenticationCallback() {
			 @Override
			 public void sessionCreated(XiSession xiSession) {
				 //Access Xively services using the xiSession session object
				 ...
			 }

			 @Override
			 public void authenticationFailed() {
				 showAlert("Login error", "Authentication failed.");
			 }
		 });
```

Once you are logged in your user context and JWT is stored and handled by the Xively SDK.

### List Devices

The SDK provides a service to retrieve a list of devices accesible with your credentials and their details (device id, device name, channels etc.).
The service can be accessed through the *requestXiDeviceInfoList* API of the XiSession object.

```java
session.requestXiDeviceInfoList(new XiDeviceInfoCallback() {
            @Override
            public void onDeviceInfoListReceived(List<XiDeviceInfo> list) {
                //process the list as required
                for (XiDeviceInfo device : list){
                    myDevices.add(device.deviceName, deviceInfo.deviceId);
					//...
                }
            }

            @Override
            public void onDeviceInfoListFailed() {
                //device listing failed
            }
        });
```

### List Organizations

The SDK provides a service to retrieve a list of organizations accesible with your credentials and their details.
The service can be accessed through the *requestXiOrganizationList* API of the XiSession object.

```java
session.requestXiOrganizationList(new XiOrganizationListCallback() {
            @Override
            public void onOrganizationListReceived(List<XiOrganizationInfo> list) {
                //process the list as required
                for (XiOrganizationInfo org : list){
					//...
                }
            }

            @Override
            public void onOrganizationListFailed() {
                //org listing failed
            }
        });
```

### List End Users

The SDK provides a service to retrieve a list of end users accesible with your credentials and their details.
The service can be accessed through the *requestXiEndUserList* API of the XiSession object.

```java
session.requestXiEndUserList(new XiEndUserListCallback() {
            @Override
            public void onEndUserListReceived(List<XiEndUserInfo> list) {
                //process the list as required
                for (XiEndUserInfo org : list){
					//...
                }
            }

            @Override
            public void onEndUserListFailed() {
                //end user listing failed
            }
        });
```

### Get End User

The SDK provides a service to retrieve one specific end user accesible with your credentials and their details.
The service can be accessed through the *requestXiEndUser* API of the XiSession object.

```java
session.requestXiEndUser(new XiEndUserCallback() {
            @Override
            public void onEndUserReceived(XiEndUserInfo info) {
                //process the info as required
            }

            @Override
            public void onEndUserFailed() {
                //end user info failed
            }
        });
```

### Get Device

The SDK provides a service to retrieve one specific device accesible with your credentials and their details.
The service can be accessed through the *requestXiDeviceInfo* API of the XiSession object.

```java
session.requestXiDeviceInfo(new XiDeviceInfoCallback() {
            @Override
            public void onDeviceInfoReceived(XiDeviceInfo deviceInfo) {
                //process the info as required
            }

            @Override
            public void onDeviceInfoFailed() {
                //end user info failed
            }
        });
```

### Update End User

The SDK provides a service to update one specific end user accesible with your credentials and their details.
The service can be accessed through the *requestXiEndUserUpdate* API of the XiSession object.
You have to provide the userId, the version tag of the user data retrieved by requestDeviceInfo or requestDeviceInfoList functions and a HashMap containing the new fields or custom fields you want to modify.

```java
session.requestXiEndUserUpdate(userId, version, data, new XiEndUserUpdateCallback() {
            @Override
            public void onEndUserUpdateSuccess(XiEndUserInfo info) {
                //process the info as required
            }

            @Override
            public void onEndUserUpdateFailed() {
                //end user update failed
            }
        });
```

### Update Device

The SDK provides a service to update one specific device accesible with your credentials and their details.
The service can be accessed through the *requestXiDeviceUpdate* API of the XiSession object.
You have to provide the userId, the version tag of the user data retrieved by requestDeviceInfo or requestDeviceInfoList functions and a HashMap containing the new fields or custom fields you want to modify.

```java
session.requestXiDeviceUpdate(userId, version, data, new XiEndUserUpdateCallback() {
            @Override
            public void onDevicUpdateSuccess(XiDeviceInfo info) {
                //process the info as required
            }

            @Override
            public void onDeviceUpdateFailed() {
                //device update failed
            }
        });
```

### Send Message

The Xively message bus makes it easy to communicate back and forth with your devices.
If you want to send messages to your device the first thing you have to do is connect, then you can publish and receive messages.

#### 1. Connect to Message Bus

Messaging can be accessed once you have a valid XiSession. The method *requestMessaging* can return the Creator object to asynchronously create an *XiMessaging* instance.
The method *requestMessaging* can be provided clean session and last will parameters. 
If the Messaging is created with clean session on, all subscriptions are canceled if the connection is closed or lost. If the clean session is off, all subscriptions remain alive on connection close or lost. This property is turned on by default. 
Last wills are user defined messages on user defined channels that are sent in case the the user's connection is closed or lost. There is no last will defined by default.
The Messaging instance will be returned as a parameter of the *serviceCreated* event of the XiServiceCreatorCallback instances registered in the Creator. To create a Messaging instance, the Creator can be used as follows:

```java
XiMessaging xiMessaging;
final XiMessagingCreator xiMessagingCreator = xiSession.requestMessaging();

//add a ServiceCreator callback
xiMessagingCreator.addServiceCreatorCallback(new XiServiceCreator.XiServiceCreatorCallback<XiMessaging>() {
    @Override
    public void onServiceCreated(XiMessaging service) {
        //it is not required, but it is good practice to remove all callbacks from the creator
        //when you don't need it any more
        xiMessagingCreator.removeAllCallbacks();

        //The XiMessaging ready for usage. You can expect it to be fully setup, in
        // a Running state, ready  for all operations.
        xiMessaging = service;
    }

    @Override
    public void onServiceCreateFailed() {
        xiMessagingCreator.removeAllCallbacks();
        //... handle failed event
    }
});

xiMessagingCreator.createXiMessaging();
```

#### 2. Messaging service state changes
The state listener interface provides callbacks about the connection status.

```java
xiMessaging.addStateListener(new XiMessagingStateListener() {
								@Override
								public void onStateChanged(XiMessaging.State state) {
									//Implement your actions based on the current state of the connection:
									//Running, Reconnecting, Closed, Error
								}

								@Override
								public void onError() {
									//The messaging connection has ended with an error
								}
							});
```

#### 3. Publish a Message
The following subscribe and publish operations are always available when the service is in the *Running* state:

```java
		//create a simple string message
        byte[] byteArrayMessage = "Hello Xively World!".getBytes();

        //note that you may not be subscribed to a channel in order to publish on it
        byte[] byteArrayMessage = "String command".getBytes();
        xiMessaging.publish("My IoT Device's channel", byteArrayMessage, XiMessaging.XiMessagingQoS.AtMostOnce);

        //retained publish
        xiMessaging.publish("My IoT Device's channel", byteArrayMessage, true, XiMessaging.XiMessagingQoS.AtLeastOnce);

        xiMessaging.unsubscribe("My IOT Device's channel");
```

### Receive Messages

Xively uses a publish / subscribe model. This means you can setup a subscription on a specific channel and be notified anytime a new message arrives.
Once a Messaging object instance is created you can register listeners for its events.
Note that most of the operations may throw an XiException if the operation fails or it is not available:

```java
try {
	 //dataListener defines an interface to receive messages
	 xiMessaging.addDataListener(new XiMessagingDataListener() {
                                    @Override
                                    public void onDataReceived(byte[] bytes, String s) {
                                        String newStringMessage = new String(bytes, Charset.defaultCharset());
										//process or display the message
                                    }

                                    @Override
                                    public void onDataSent(int i) {
										//The message with this id has been sent to the message broker
                                    }
                                });
		
	 //subscription listener provides callback about subscribe/unsubscribe success
     xiMessaging.addSubscriptionListener(new XiMessagingSubscriptionListener() {
            @Override
            public void onSubscribed(String s) {
                //subscribe to channel `s` was successful
            }

            @Override
            public void onSubscribeFailed(String s) {
                //subscribe to channel `s` failed
            }

            @Override
            public void onUnsubscribed(String s) {
				//unsubscribe from channel `s` was successful
            }

            @Override
            public void onUnsubscribeFailed(String s) {
				//unsubscribe from channel `s` failed
            }
        });
	 
    } catch (XiException e) {
        //this can be a XiException.NotConnectedException if meanwhile the connection was lost
        //or XiException.ConnectionException for general connection errors
    }
```

If you don't need the Messaging service any more you should remove all your listeners and invoke *xiMessaging.close()*.

### Query TimeSeries Data

Xively provides TimeSeries data storage to persist the data of specific device channels (e.g. sensor data). 


```java
//The TimeSeries APIs can be accessed from the session object
XiTimeSeries timeSeries = xiSession.timeSeries();

//start date will be one week before before the current time 
Date start = new Date(System.currentTimeMillis() - (168 * 60 * 60 * 1000) );

//end date is the current date for this example
Date end = new Date(System.currentTimeMillis());

//TimeSeries optionally can filter a specific category of keys for your request
String category = "airQuality";

timeSeries.requestTimeSeriesItemsForChannel(sensorChannel, start, end, category, new XiTimeSeriesCallback() {
    @Override
    public void onTimeSeriesItemsRetrieved(ArrayList<TimeSeriesItem> items) {
         //we can process the received data
         for (TimeSeriesItem tsData : items) {
            addToGraph(tsData.numericValue);
         }
    }

    @Override
    public void onFinishedWithError(XiTimeSeriesError error) {
         showError("Failed to load data...");
    }

    @Override
    public void onCancelled() {
         showError("Failed to load data...");
    }
}
```