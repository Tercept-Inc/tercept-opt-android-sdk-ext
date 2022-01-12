For External Android packages
 
## Tercept Android SDK for Tercept Analytics & Optimization
 
 
NOTE: minimum API level is **21**.


Integrating Tercept library in your android project

1. Go to settings.gradle of your app and inside your repositories, include our maven github repo like this
```
      google()
      mavenCentral() 
//        jcenter() // Warning: this repository is going to shut down soon
      maven {
          name = "GitHubPackages"
          url = uri("https://maven.pkg.github.com/tercept-inc/tercept-opt-android-sdk-kotlin-pub")
          credentials {
               username = “Your Github username”
               password = “Your Personal Access Token with read package permission at the least”        
          }
      }
```
2. Inside your build.gradle for the app, in dependencies, include
```
    implementation 'com.tercept.sdk:tercept-android-sdk:4.1.0'
```
  Resync your gradle to get the tercept library  

  In case you are unable to implement the above process, one can also download the aar file from this
  [link](https://github.com/Tercept-Inc/tercept-opt-android-sdk-pub) and going inside the **_Packages_**
  section 


 
 
 
## Tercept SDK Usage and Functions
 
 
### Initialize the SDK by calling init(Activity activity, String GAID, JSONObject params).
```
    import com.tercept.sdk.TerceptOptimization;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        TerceptOptimization terceptOpt = new TerceptOptimization(networkCode);
        terceptOpt.init(this, GAID, params);
    	  //your code
    }
```
  `networkCode` => DFP network code.  
  `this` => Reference to the activity instance.  
  `String GAID` => Google Advertising ID for the user.  
  `JSONObject params` => (optional) A JSON object of custom parameters. Here we should send any parameters which might help us do a better targeting for ad unit optimisations like web_url, age or any other parameters which are non-standard  
  
  If no custom parameters are to be passed, the following call can also be used.
```
    terceptOpt.init(this, GAID);
```
 
  If you work with different networkCodes in the same app then one has to do initialization for all the different networkCodes
  For example:
```
    TerceptOptimization terceptOpt1 = new TerceptOptimization(networkCode1)
    terceptOpt1.init(this, GAID, params)

    TerceptOptimization terceptOpt2 = new TerceptOptimization(networkCode2)
    terceptOpt2.init(this, GAID, params)
                          .
                          . 
                          .
                          .
```
  All the methods described below need to be called for each instantiation of the class. 
  At every page where adunits are integrated with Tercept  




### For fetching key values for adunits from tercept server
```
    terceptOpt.fetch(adunits);
```

  `fetch(adunits)` => Initiates an asynchronous HTTPS GET request which obtains ad-unit specific configuration. 

  `ArrayList<String> adunits` => ArrayList of strings representing list of unique ad-unit names. 

  Ideally, we should send in all the ad units at once specific to a particular page which user is viewing.  




### For getting key values for building the ad request
  Before building each ad request

```
    Map<String, String> keys = terceptOpt.getCustomTargetingKeys(adunit);
```

  `keys` => a map containing custom targeting key names and values.

  `String adunit` => A string representing the unique name of the adunit.

  And then iterate over the keys map and add custom targeting key values.

```
    PublisherAdRequest.Builder builder = new PublisherAdRequest.Builder();
    for (Map.Entry<String,String> keyValue : keys.entrySet()) {
        builder.addCustomTargeting(keyValue.getKey(),keyValue.getValue())
    }

```



### For Logging gam events data 
  Listen to the following ad events:   
    _onAdClicked, onAdClosed, onAdFailedToLoad, onAdImpression, onAdLeftApplication, onAdLoaded, onAdOpened_

  And for IMA ads, there are some additional events:  
    _onFirstQuartile, onMidpoint, onThirdQuartile, onStarted, onSkipped_

  There is an enum class called `GAMEVENTS` which exposes all these events to the client while logging the events to Tercept.
 
  In the handlers for each of those events:

```
    terceptOpt.logEvent(adunit, eventName);
```

  `String adunit` => A string representing the unique name of the adunit.

  `GAMEVENTS eventName` => An enum field representing the unique name of the event from the following list of event names only: 

    onAdClicked, onAdClosed, onAdFailedToLoad, onAdImpression, onAdLeftApplication, onAdLoaded, onAdOpened, onFirstQuartile, onMidPoint, onThirdQuartile, onStarted, onSkipped

  Ideally, all the events must be logged. Tercept’s SDK will decide which events to actually capture to do advertisement optimizations.

  This function does not return anything.

  It can be used in conjunction with custom parameters as part of it to send custom parameters for the logging events.

  We can also pass custom event parameters to each event getting logged using this flavour of log event.

```
    terceptOpt.logEvent(adunit, eventName, customEventParams);
```  




### To check what events are logged currently - FOR DEBUG on client side
  We can check the events which have been currently logged by tercept to be sent to tercept’s server. This method is mostly for debugging purposes.

```
    terceptOpt.getEventsData()
```
  Returns a JSON object of all the events logged to be sent to tercept.   



### To send the logged events to tercept server
  At the end of each user session:

```
    terceptOpt.sendEventsData();
```  


### To set custom parameters explicitly
  Additionally, if custom parameters cannot be sent in UI context, we can set them explicitly as well. This will be sent to the tercept server whenever events data is sent as part of `sendEventsData()`.

```
    terceptOpt.setCustomParameters(params);
```

  `Params` => list of JSON object key values to be sent as custom parameters for tercept. Ideally, the init method should be preferred over this to maintain the sync of ad units and custom parameters.
 

