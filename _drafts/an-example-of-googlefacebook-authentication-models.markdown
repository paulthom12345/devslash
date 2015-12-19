---
layout: post
title: Facebook and Google Authentication with PickAxe
---

Facebook and Google provide fantastic interfaces for authentication. This allows Android developers to offload the responsibility of authentication to another provider. This has many obvious advantages, the major two being:

- You have no need to create your own authentication scheme that is secure
- In the event of a data breach you do not leak a password

With this is  mind there are several examples of apps that have achieved great success in their engagement through the use of Google/Facebook authentication. **The Guardian's** Android App sees 41% of their user base come through their Google+ login system.

Unfortunatley it's not an entirely simple process to integrate the various options of the login providers. For example, *Facebook* provides a fantasic interface that will automatically deal with re-logging in without the developer even calling a single method. Google/Facebok also both have completely different requirements for using API Tokens server side.

**PickAxe** enables the developer to not have to concern themselves with the specific implementation of the third part provider.

Before going too far, **PickAxe** is currently in Early-Access and should not be consdered stable. *The version number is an issue and hopefully will be reverted to a sensible number before too long to reflect the project status*. 

## Using Easy Auth

Easy Auth provides an extreemely simple interface between your app and Facebook/Google. To get started, add the following to your `build.gradle`. 
```
compile net.devslash.easyauth:1.0.+
```

In each Activity/Fragment that you're wanting to know details of the user that is logged in you'll need to add an `AuthenticationProvider`.

```java
class MainActivity extends Activity implements CallbackListener{
    
    AuthenticationProvider provider;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        
        provider = new AuthenticationProvider(this)
    
    
```
Now till this point we haven't actually got a way to know when the user has actually been confirmed as having had logged in. For this reason we will have to make sure that we register as a `AuthenticationListener`. This will cause the activity to get callbacks for each event of Logging in/out. 

## Profile Providers
When a user has logged in you will receive a ProfileProvider through the callback interface. When initially registering for callbacks you can opt to receive a `ProfileProvider` instantly if the user has already logged in. This `ProfileProvider` is just an abstraction on top of the providers that Facebook/Google do provide. 

**Drawbacks**

There are several reasons that you might not want to use **PickAxe**. At the moment most of them come from how ProfileProviders work. 

* You cannot get custom details about a user, you can only receive details that are in the ProfileProvider interface
* You cannot use request additional permissions to use with the API key later on. It's only the details that are already required by **PickAxe**.