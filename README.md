# Parse Starter Project Android From Scratch

The following steps will enable you to create a Parse starter project from scratch for Android. Instructions for creating a Parse starter project for [iOS (and Heroku) are here](https://github.com/plaudev/ParseStarterProjectFromScratch). For setting up your custom Parse server & push notification on Heroku, [see here](https://github.com/plaudev/ParseHerokuPushNotification). I will leave the server side stuff largely out in this post. While written with hosting Parse on the cloud in mind, these instructions should work with a local set up as well.

## 1 Download Android Parse SDK Directly from Parse

Download the Android Parse SDK [directly from Parse here](https://parse.com/apps/quickstart#parse_data/mobile/android/native/existing). We will reference & copy specific lines of code from it without importing those files directly. Just download & unzip this SDK so you can refer to it in the steps below.

## 2 Create a New Project in Android Studio

Create your own project your usual way in Android Studio giving it any name you want. We will edit this project to make it talk to Parse.

## 3 Edit strings.xml

Insert the following lines into `strings.xml`:
```
<string name="parse_app_id">__YOUR_APP_ID__</string>
<string name="parse_client_key"></string>
<string name="parse_server">__SERVER_URL__</string>
```
You will need to insert your own `appId` & `serverUrl` of course. Notice `clientKey` is blank; this is not a mistake. [Client keys are not longer necessary with Parse server](https://github.com/ParsePlatform/parse-server). To be clear, `clientKey` is **NOT** the same thing as `masterKey` or `fileKey`. The `serverUrl` should include `http://` or `https://` as the case may be; and also include or exclude `/parse` (or `/parse/`) as the case may be depending on:

* which cloud vendor is hosting your Parse server, or
* how you manually set up your Parse server.

## 4 Edit AndroidManifest.xml

Insert these lines to give proper permissions:
```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```
Insert the `name` variable somewhere withing the `<application>` tag like this. Note that we are going to add a new class called `StarterApplication` in a few moments:
```
<applicatiomn ...
  android:name=".StarterApplication"
  ... />
```
Insert the `name` & `label` variables somewhere witin the `<activity>` tag like this:
```
<activity ...
  android:name=".MainActivity"
  android:label="@string/app_name"
  ... />
```
Insert the following `meta` tags into the `<application>` block. These will pick up the definitions in `strings.xml`:
```
<meta-data
  android:name="com.parse.APPLICATION_ID"
  android:value="@string/parse_app_id" />
<meta-data
  android:name="com.parse.CLIENT_KEY"
  android:value="@string/parse_client_key" />
```

## 5 Edit build.gradle (modeule)

Insert the following into the `dependencies { ... }` block:
```
compile 'com.parse.bolts:bolts-android:1.+'
compile 'com.parse:parse-android:1.+'
```

## 6 Add StarterApplication Class

Do `File` -> `New` -> `Java Class` to add a class named `StarterApplication` and replace the class definition with the following. Again the code will pick up the definitions in `strings.xml`. Turning on auto insertion of import statements might save you a little bit of time.
```
public class StarterApplication extends Application {

    String appId;
    String clientKey;
    String serverUrl;

    @Override
    public void onCreate() {
        super.onCreate();

        appId = getResources().getString(R.string.parse_app_id);
        clientKey = getResources().getString(R.string.parse_client_key); // should be blank
        serverUrl = getResources().getString(R.string.parse_server);
        Log.i("StarterApplication", "appId=" + appId + ", clientKey=" + clientKey + ", serverUrl=" + serverUrl);

        // Enable Local Datastore.
        Parse.enableLocalDatastore(this);

        // OPTIONAL https://github.com/ParsePlatform/ParseInterceptors-Android/wiki/ParseLogInterceptor
        //Parse.addParseNetworkInterceptor(new ParseLogInterceptor());

        // Add your initialization code here
        Parse.initialize(new Parse.Configuration.Builder(getApplicationContext())
                .applicationId(appId)
                .clientKey(clientKey) // can leave out since it is blank
                .server(serverUrl)
                .build()
        );

        ParseUser.enableAutomaticUser();
        ParseACL defaultACL = new ParseACL();
        // Optionally enable public read access.
        //defaultACL.setPublicReadAccess(true);
        //defaultACL.setPublicWriteAccess(true);
        ParseACL.setDefaultACL(defaultACL, true);

        // object = table 
        // feel free to enter your own test values here
        ParseObject parseObject = new ParseObject("Test");
        parseObject.put("pasta", "cannelloni");
        parseObject.put("price", 4.99);

        Log.i("StarterApplication", "Attempting to save...");
        parseObject.saveInBackground(new SaveCallback() {
            @Override
            public void done(ParseException e) {
                if (e == null) {
                    Log.i("saveInBackground", "Success... congrats!");
                } else {
                    Log.i("saveInBackground", "Ooops... " + e.toString());
                }
            }
        });
    }
}
```

## 7 Edit MainActivity.java

Insert this line into `onCreate()`:
```
ParseAnalytics.trackAppOpenedInBackground(getIntent());
```

## 8 Et Voila

If you have done everything above correctly, you should see *Success... congrats!* in the logcat. From the Parse dash-board, you should also see `cannelloni` costing $`4.99` added to the `Test` table (or whatever custom entries you had made earlier) in your Parse database.

## 9 Debug

Should you unfortunately see *Ooops... `error message`* in the logcat, the test entry failed to be written into Parse. You can verify that the Parse server is actually set up correctly by running a cURL command like this one:
```
curl -X POST -H "X-Parse-Application-Id: __YOUR_APP_ID__" -H "Content-Type: application/json" -d '{"pasta":"farfalle","price":3.99}' __SERVER_URL__/parse/classes/Test
```
and get a result like this:
```
{"objectId":"pV6NI9yrqY","createdAt":"2016-11-24T21:22:03.576Z"}
```
If you do, then the Parse server is running properly and the problem likely lies in your settings for Parse on your cloud vendor. In my case, after I did all the above it did not work. My problem turned out to be in the security group settings. Having set up my Parse server [following instructions here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html) among others, I had only set up `SSH` connection and removed `HTTP` & `HTTPS`. The error I was getting from the logcat with the *Ooops...* line was: 
```
com.parse.ParseRequest$ParseRequestException: i/o failure
```
Reinserting an entry for `HTTP` got me *Success... congrats!* and `farfalle` costing $`3.99` in my `Test` table on Parse.

## 10 Follow Up

Please comment here with feedback & questions and/or follow me [@PLauDev](https://twitter.com/plaudev).
