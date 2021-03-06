## Overview

Intent is a powerful concept within the Android universe. An intent is a message that can be thought of as a request that is given to either an activity within your own app, an external application, or a built-in Android service.

Think of an intent as a way for an Activity to communicate with the outside Android world. A few key tasks that an intent might be used for within your apps:

 * Take the user to another screen (activity) within your application
 * Take the user to a particular URL within the Android web browser 
 * Take the user to the camera to have them take a picture
 * Initiate a call for the user to a given number

As you can see, the Intent is a core part of user flows in Android development. The [Intent](http://developer.android.com/reference/android/content/Intent.html) object itself is a class that represents a particular "request" including the topic of the request and any request "parameters" which are called the [Bundle](http://developer.android.com/reference/android/os/Bundle.html)..

## Explicit Intents

An "explicit" intent is used to launch other activities within your application. For example, if you the user presses the "compose" button and you want to bring up an activity for them to compose a message, you would launch that second activity using an explicit intent.

Using an intent is as simple as constructing the [Intent](http://developer.android.com/reference/android/content/Intent.html) with the correct parameters and then invoking that intent using the `startActivity` method:

```java
// ActivityOne.java
public void launchComposeView() {
  // first parameter is the context, second is the class of the activity to launch
  Intent i = new Intent(ActivityOne.this, ActivityTwo.class);
  startActivity(i); // brings up the second activity
}
```

```kotlin
// ActivityOne.kt
fun launchComposeView() {
  // first parameter is the context, second is the class of the activity to launch
  val i = Intent(this@ActivityOne, ActivityTwo::class.java)
  startActivity(i) // brings up the second activity
}
```
Now, in the launched second activity, the user can go back to the first screen by hitting "back" or if the developer wants to trigger the second activity to close, we need only call the [`finish` method](http://developer.android.com/reference/android/app/Activity.html#finish\(\)):

```java
// ActivityTwo.java
public void onSubmit(View v) {
  // closes the activity and returns to first screen
  this.finish(); 
}
```
```kotlin
// ActivityTwo.kt
fun onSubmit(v: View) {
  // closes the activity and returns to first screen
  this.finish();
}
```

**Note:** The first argument of the Intent constructor used above is a [Context](http://developer.android.com/reference/android/content/Context.html) which at the moment is just the current Activity in scope.

### Passing Data to Launched Activities

In addition to specifying the activity that we want to display, an intent can also pass key-value data between activities. Think of this as specifying the "request parameters" for an HTTP Request. You can specify the parameters by putting key-value pairs into the intent bundle:

```java
// ActivityOne.java
public void launchComposeView() {
  // first parameter is the context, second is the class of the activity to launch
  Intent i = new Intent(ActivityOne.this, ActivityTwo.class);
  // put "extras" into the bundle for access in the second activity
  i.putExtra("username", "foobar"); 
  i.putExtra("in_reply_to", "george"); 
  i.putExtra("code", 400);
  // brings up the second activity
  startActivity(i); 
}
```
```kotlin
// ActivityOne.kt
fun launchComposeView() {
  // first parameter is the context, second is the class of the activity to launch
  val i = Intent(this@ActivityOne, ActivityTwo::class.java)
  // put "extras" into the bundle for access in the second activity
  i.putExtra("username", "foobar")
  i.putExtra("in_reply_to", "george") 
  i.putExtra("code", 400)
  // brings up the second activity
  startActivity(i)
}
```

Once you have added data into the bundle, you can easily access that data within the launched activity:

```java
// ActivityTwo.java (subactivity) can access any extras passed in
protected void onCreate(Bundle savedInstanceState) {
   String username = getIntent().getStringExtra("username");
   String inReplyTo = getIntent().getStringExtra("in_reply_to");
   int code = getIntent().getIntExtra("code", 0);
}
```
```kotlin
// ActivityTwo.kt (subactivity) can access any extras passed in
fun onCreate(savedInstanceState: Bundle) {
   val username = getIntent().getStringExtra("username")
   val inReplyTo = getIntent().getStringExtra("in_reply_to")
   val code = getIntent().getIntExtra("code", 0)
}
```

And using this system the intent can pass useful data across activities.

### Returning Data Result to Parent Activity

In the typical case of using `startActivity`, the activity is launched and added to the navigation stack and no result is expected. If the user wants to close the activity, the user can simply hit "back" and the parent activity is displayed. 

However, in other cases the parent activity may want the launched activity to return a result back when it is finished. In this case, we use an `ActivityResultLauncher`, which launches an Activity and expects a `Result` to come back.

```java
// ActivityOne.java
ActivityResultLauncher<Intent> editActivityResultLauncher = registerForActivityResult(
        new ActivityResultContracts.StartActivityForResult(),
        new ActivityResultCallback<ActivityResult>() {
            @Override
            public void onActivityResult(ActivityResult result) {
                // If the user comes back to this activity from EditActivity
                // with no error or cancellation
                if (result.getResultCode() == Activity.RESULT_OK) {
                    Intent data = result.getData();
                    // Get the data passed from EditActivity
                    String editedString = data.getExtras().getString("newString");
                }
            }
        });

public void startEditActivity() {
    Intent intent = new Intent(this, EditActivity.class);
    intent.putExtra("stringToEdit", "CodePath")
    editActivityResultLauncher.launch(intent);
}
```
```kotlin
// ActivityOne.kt
var editActivityResultLauncher: ActivityResultLauncher<Intent> = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    // If the user comes back to this activity from EditActivity
    // with no error or cancellation
    if (result.resultCode == Activity.RESULT_OK) {
        val data = result.data
        // Get the data passed from EditActivity
        if (data != null) {
            val editedString = data.extras!!.getString("newString")
        }
    }
}

fun startEditActivity() {
    val intent = Intent(this, MainActivity::class.java)
    intent.putExtra("stringToEdit", "CodePath")
    editActivityResultLauncher.launch(intent)
}
```

This will launch the subactivity, and when the subactivity is complete then it can return the result to the parent:

```java
// EditActivity.java -- launched for a result
public void onSubmit(View v) {
  EditText et = (EditText) findViewById(R.id.textToEdit);
  // Prepare data intent 
  Intent data = new Intent();
  // Pass relevant data back as a result
  data.putExtra("newString", et.getText().toString());
  // Activity finished ok, return the data
  setResult(RESULT_OK, data); // set result code and bundle data for response
  finish(); // closes the activity, pass data to parent
} 
```
```kotlin
fun onSubmit(v: View) {
    val et = findViewById<View>(R.id.textToEdit) as EditText
    // Prepare data intent
    val data = Intent()
    // Pass relevant data back as a result
    data.putExtra("newString", et.text.toString())
    // Activity finished ok, return the data
    setResult(RESULT_OK, data) // set result code and bundle data for response
    finish() // closes the activity, pass data to parent
}
```

Once the sub-activity finishes, the 'onActivityResult' callback that was implemented in `editActivityResultLauncher` is invoked.

And using that process you can communicate data freely between different activities in your application.

### Passing Complex Data in a Bundle

Bundles can pass complex Java objects into Intents through the use of serialization. Simple types such as integers and strings can be automatically passed as an extra but to pass Java objects, they need to be `Serializable` or `Parcelable`. 

```java
public class User implements Serializable {
	private static final long serialVersionUID = 5177222050535318633L;
	private String firstName;
	private String lastName;
	private int age;
}
```
```kotlin
class User : Serializable {
	val serialVersionUID: Long = 5177222050535318633L
        var firstName: String
	var String lastName: String
	var int age: String
}
```
Then you can pass arbitrary user objects into an intent as an extra:

```java
User u = new User("John", "Smith", 45);
Intent  i =  new Intent(ActivityOne.this, SecondActivity.class);
i.putExtra("user", u);
startActivity(i);
```
```kotlin
val u = User("John", "Smith", 45)
val i = Intent(this@ActivityOne, SecondActivity::class.java)
i.putExtra("user", u)
startActivity(i)
```

and access the user data in the launched intent with `getSerializableExtra`:

```java
// SecondActivity.java
User u = (User) getIntent().getSerializableExtra("user");
TextView tvUser = (TextView) findViewById(R.id.tvUser);
tvUser.setText(u.getFirstName() + " " + u.getLastName());
```
```kotlin
// SecondActivity.kt
val u: User? = intent.getSerializableExtra("user") as User?
val tvUser = findViewById<View>(R.id.tvUser) as TextView
tvUser.setText(u.getFirstName() + " " + u.getLastName())
```

A faster and more reliable approach is to use [Parcelable as a replacement](http://guides.codepath.com/android/Using-Parcelable) for Serializable. Parcelable is about 10x faster than Serializable and thanks to libraries like [Parceler](http://guides.codepath.com/android/Using-Parceler) not very hard to implement.

## Implicit Intents

Implicit Intents are requests to perform an action based on a desired action and target data. This is in contrast to an explicit intent that targets a specific activity. For example, if I want to make a phone call for the user, that can be done with this intent:

```java
Intent callIntent = new Intent(Intent.ACTION_CALL);
callIntent.setData(Uri.parse("tel:3777789888"));
startActivity(callIntent);
```
```kotlin
val callIntent = Intent(Intent.ACTION_CALL)
callIntent.data = Uri.parse("tel:3777789888")
startActivity(callIntent)
```

If I want to launch a website in the phone's browser, I might do this:

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));
startActivity(browserIntent);
```
```kotlin
val browserIntent = Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"))
startActivity(browserIntent)
```

You can see a list of other [[common implicit intents|Common-Implicit-Intents]].

### Receiving Implicit Intents

If we wish to receive implicit intents, you need to associate intent filters with your activity.  You can also enable links within a view page to launch your app using [deep links](https://developer.android.com/training/app-indexing/deep-linking.html).  You can also create custom URL schemes (i.e. instead of http:// you can create example://) patterns that are primarily meant to launch your app as well.  

Make sure to specify `android.intent.category.DEFAULT` category to declare that the activity should receive implicit intents.   Otherwise, the activity can only respond to explicit intents.  You also need to declare what type of implicit intent action to which it responds with the `<action>` tag.  The list of possible actions 
are shown in the `Standard Activity Actions` section [here](https://developer.android.com/reference/android/content/Intent.html).

If you wish to receive a share intent from Chrome, for instance, you have to declare an activity will be responsible for processing the data in your `AndroidManifest.xml` file:

```xml
<activity
    android:name="com.codepath.ReceiveIntentDataActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"></data>
    </intent-filter>
```

Receiving the data entails receiving the intent data:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  // Get intent, action and MIME type
  Intent intent = getIntent();
  String action = intent.getAction();
  String type = intent.getType();

  if (Intent.ACTION_SEND.equals(action) && type != null) {
    if ("text/plain".equals(type)) {

      // Make sure to check whether returned data will be null.
      String titleOfPage = intent.getStringExtra(Intent.EXTRA_SUBJECT);
      String urlOfPage = intent.getStringExtra(Intent.EXTRA_TEXT);
      Uri imageUriOfPage = (Uri) intent.getParcelableExtra(Intent.EXTRA_STREAM);
    }   
  }
```
```kotlin
override fun onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  val intent = intent
  val action = intent.action
  val type = intent.type

  if (Intent.ACTION_SEND == action && type != null) {
     if ("text/plain" == type) {

     // Make sure to check whether returned data will be null.
     val titleOfPage = intent.getStringExtra(Intent.EXTRA_SUBJECT)
     val urlOfPage = intent.getStringExtra(Intent.EXTRA_TEXT)
     val imageUriOfPage: Uri? = intent.getParcelableExtra<Parcelable>(Intent.EXTRA_STREAM) as Uri?
     }
  }
```

See [this article](https://paul.kinlan.me/parsing-screenshot-from-Chrome-for-Android-send-intent/) for more details about parsing data from Chrome. **_This link was invalid_**

If you wish for a web page link to be able to launch an activity, make sure to also specify the BROWSABLE category as well.  You should also specify the `<data>` tag to specify what URL pattern to scan for:

```xml
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.BROWSABLE" />
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="http"
              android:host="www.codepath.com"
              android:pathPrefix="/example" />
    </intent-filter>
```

Receiving the specific URL being requested entails calling `getData()` on the intent:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    Intent intent = getIntent();
    String action = intent.getAction();
    Uri data = intent.getData();
}
```
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
   setContentView(R.layout.main)
   val intent = intent
   val action = intent.action
   val data: Uri? = intent.data
}
```

You can also receive more complex data types as shown in this [guide](https://developer.android.com/training/sharing/receive.html).

You can also use the [DeepLinkDispatch](https://github.com/airbnb/DeepLinkDispatch) library to makes it easy to annotate your activities with specific URL patterns.

### Default link handling

If you wish for your app to be considered the default handler for a link (only works for Android Marshmallow devices and higher), you can leverage the new feature by setting `android:autoVerify` on the intent filter:

```xml
<intent-filter android:autoVerify="true">

</intent>
```

You would then need to host an `assetlinks.json` file at `https://domain[:optional_port]/.well-known/assetlinks.json` that uses the following format:

```javascript
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.mycompany.app1",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```

The `sha256_cert_fingerprints` comes from the app signing certificate by typing `keytool -list -v -keystore my-release-key.keystore`.  The package name should correspond to your app's package name.  See [this guide](https://developer.android.com/training/app-links/index.html) for more details.

You can check to see if this file is hosted properly by checking this URL:

```
https://digitalassetlinks.googleapis.com/v1/statements:list?
   source.web.site=https://<domain1>:<port>&
   relation=delegate_permission/common.handle_all_urls
```

## References

 * <http://developer.android.com/guide/components/intents-filters.html>
 * <http://developer.android.com/reference/android/content/Intent.html>
 * <http://www.vogella.com/articles/AndroidIntent/article.html>
 * [[Common Implicit Intents|Common-Implicit-Intents]]
 * <https://developer.android.com/training/app-links/index.html>