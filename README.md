#How to Add an Android Header in a Calling App (With User Location)

This tutorial demonstrates how to make a Sinch app-to-app call with a header. In this app, I'll send the location of the person calling so the recipient can see where the other user is calling from. 

![screenshot of finished app](images/finished-app.png)

The app is built on top of the **sinch-rtc-sample-calling** sample included in the Android SDK, and the finished code can be found on [our GitHub](https://www.github.com/sinch/android-app-app-calling-headers).

##Setup

1. Create a free [Sinch developer account](https://www.sinch.com/signup)
2. Create an app in the [Developer Dashboard](https://www.sinch.com/dashboard/#/apps)
3. Download the [Sinch Android SDK](https://www.sinch.com/downloads)
4. Add your app key and secret to the **sinch-rtc-sample-calling** app included in the SDK (in **SinchService.java**)

##Update SinchServiceInterface

Sinch allows you to make an app-to-app call and send along a `Map<String, String>` of headers. First, extend the `SinchServiceInterface` in **SinchService.java** to support this method:

    public Call callUser(String userId, Map<String, String> headers) {
        return mSinchClient.getCallClient().callUser(userId, headers);
    }
    
##Get location and make the call

You'll need the following permission to access the device's location:

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    
Now, in `callButtonClicked` in **PlaceCallActivity.java**, you can get the latitude and longitude of the last known location:

    LocationManager locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
    Location lastLoc = locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER);
    Double longitude = lastLoc.getLongitude();
    Double latitude = lastLoc.getLatitude();
    
**Note:** This is definitely not a production-ready way of getting an accurate location. First of all, your app will crash if the user doesn't have any "last location." You could also use the ACCESS_COARSE_LOCATION permission, since you don't need to know their *exact* location. Regardless, there are several location strategies and I suggest doing some research if you want to send a current location in your production app.
    
Use a Geocoder object to turn the latitude and and longitude into a human-readable address:

    Geocoder geocoder = new Geocoder(this, Locale.getDefault());
    List<Address> addresses = null;
    try {
        addresses = geocoder.getFromLocation(latitude, longitude, 1);
    } catch (IOException e) {
        e.printStackTrace();
    }
    
I chose to send the city/state/zipcode of the location as the header:

    Map<String, String> headers = new HashMap<String, String>();
    headers.put("location", addresses.get(0).getAddressLine(1));
    
You can add as many items as you like to the headers like so:

    headers.put("key", "value");
    
Finally, change this line that makes the call:

    Call call = getSinchServiceInterface().callUser(userName);
    
to this:

    Call call = getSinchServiceInterface().callUser(userName, headers);
    
##Display header value on incoming call

In `SinchCallClientListener#onIncomingCall` in **SinchService.java**, get the location value from the headers, and pass it along in the intent:

    intent.putExtra("location", call.getHeaders().get("location"));
    
Then, in **IncomingCallScreenActivity.java**, you can get the location from the intent like so:

    String location = getIntent().getStringExtra(SinchService.LOCATION);
    
I chose to create a TextView with ID remoteUserLocation in **incoming.xml**, so I could set it to the location from the header in `onServiceConnected` (right where the remoteUserId TextView is set).

    TextView remoteUserLocation = (TextView) findViewById(R.id.remoteUserLocation);
    remoteUserLocation.setText("Calling from " + mCallLocation);
    
That's all! Rinse and repeat for any data you want to send along with an app-to-app call.
