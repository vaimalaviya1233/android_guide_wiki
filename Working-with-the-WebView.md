## Overview

If you want to deliver a web application (or just a web page) as a part of a client application, you can do it using [WebView](https://developer.android.com/reference/android/webkit/WebView.html). The `WebView` class is an extension of Android's `View` class that allows you to display web pages as a part of your activity layout. Since Android 4.4, it is based on the Chrome on Android v33.0.0 according to this [reference](https://developer.chrome.com/multidevice/webview/overview).

This document shows you how to get started with WebView and how to do some additional things, such as handle page navigation and bind JavaScript from your web page to client-side code in your Android application. See [the official WebView](https://developer.android.com/guide/webapps/webview.html) docs for a more detailed look.

An alternative for using WebViews is [Chrome Custom Tabs](https://developer.chrome.com/multidevice/android/customtabs), which provides more flexibility in terms of customizing the toolbar, adding animations, or warming up the browser ahead of time.  Chrome Custom Tabs only works if Chrome on Android is installed on the browser.  For more information, see this [[guide|Chrome-Custom-Tabs]].

## Usage

### Load External Pages

To get Internet access, request the INTERNET permission in your manifest file. For example:

```xml
<manifest ... >
    <uses-permission android:name="android.permission.INTERNET" />
    ...
</manifest>
```

To add a `WebView` to your Application, simply include the `<WebView>` element in your activity layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

First, we need to configure the `WebView` to behave with reasonable defaults using [WebSettings](https://developer.android.com/reference/android/webkit/WebSettings.html) and creating a [WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient.html):

```java
public class MainActivity extends Activity {
   private WebView myWebView;

   @Override		
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      myWebView = (WebView) findViewById(R.id.webview);
      // Configure related browser settings
      myWebView.getSettings().setLoadsImagesAutomatically(true);
      myWebView.getSettings().setJavaScriptEnabled(true);
      myWebView.setScrollBarStyle(View.SCROLLBARS_INSIDE_OVERLAY);
      // Configure the client to use when opening URLs
      myWebView.setWebViewClient(new WebViewClient());
      // Load the initial URL
      myWebView.loadUrl("https://www.example.com");
   }
}
```

```kotlin
class MainActivity : Activity() {
    private lateinit var myWebView: WebView
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val myWebView = findViewById<View>(R.id.webview) as WebView

        myWebView.apply {
            // Configure related browser settings
            this.settings.loadsImagesAutomatically = true
            this.settings.javaScriptEnabled = true
            myWebView.scrollBarStyle = View.SCROLLBARS_INSIDE_OVERLAY
            // Configure the client to use when opening URLs
            myWebView.webViewClient = WebViewClient()
            // Load the initial URL
            myWebView.loadUrl("https://www.example.com")
        }
    }
}
```

You can attach javascript functions and use them within the mobile webpages as described [here in the official docs](https://developer.android.com/guide/webapps/webview.html#UsingJavaScript).

#### Handling responsive layouts

By default, the `WebView` does not account for the default scale size if HTML pages include [viewport](https://www.w3schools.com/css/css_rwd_viewport.asp) metadata. If you wish to enable the page to load with responsive layouts, you need to set it explicitly:

```java
// Enable responsive layout
myWebView.getSettings().setUseWideViewPort(true);
// Zoom out if the content width is greater than the width of the viewport
myWebView.getSettings().setLoadWithOverviewMode(true);
```

```kotlin
// Enable responsive layout
myWebView.getSettings().setUseWideViewPort(true)
// Zoom out if the content width is greater than the width of the viewport
myWebView.getSettings().setLoadWithOverviewMode(true)
```

You can also enable the ability to zoom-in controls on the page:

```java
myWebView.getSettings().setSupportZoom(true);
myWebView.getSettings().setBuiltInZoomControls(true); // allow pinch to zooom
myWebView.getSettings().setDisplayZoomControls(false); // disable the default zoom controls on the page
```

```kotlin
myWebView.getSettings().setSupportZoom(true)
myWebView.getSettings().setBuiltInZoomControls(true) // allow pinch to zooom
myWebView.getSettings().setDisplayZoomControls(false) // disable the default zoom controls on the page
```

### Loading Local Pages

In case you want to store a copy of a webpage locally to be loaded into a `WebView`, you can put it in the android `assets` folder. If you do not find one under your `main/` directory, then you can create one. Place the html, css, js, etc in this folder. 

For example, say I wanted to load a page entitled `index.html`. I would create my file under `{ProjectName}/app/src/main/assets/index.html`

then, in your activity or fragment you can use the code

```java
public class MainActivity extends Activity {
   private WebView myWebView;

   @Override		
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      myWebView = (WebView) findViewById(R.id.webview);
      myWebView.getSettings().setJavaScriptEnabled(true);
      myWebView.setWebViewClient(new WebViewClient());
      String path = Uri.parse("file:///android_asset/index.html").toString();
      myWebView.loadUrl(path);
   }
}
```

```kotlin
class MainActivity : Activity() {
    private lateinit var myWebView: WebView
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val myWebView = findViewById<View>(R.id.webview) as WebView

        myWebView.apply {
            // Configure related browser settings
            this.settings.loadsImagesAutomatically = true
            this.settings.javaScriptEnabled = true
            myWebView.scrollBarStyle = View.SCROLLBARS_INSIDE_OVERLAY
            // Configure the client to use when opening URLs
            myWebView.webViewClient = WebViewClient()
            // Load the initial URL
            myWebView.loadUrl("https://www.example.com")
            val path: String = Uri.parse("file:///android_asset/index.html").toString()
            myWebView.loadUrl(path)

        }
    }

}
```

#### Displaying a ProgressDialog in a WebView

Create your ```ProgressDialog``` in the ```setWebViewClient``` method

You can dismiss the dialog in ```onPageFinished``` method or in ```onPageCommitVisible```. Setting ```dismis``` in  the latter is convenient since your user won't have to wait for the whole page to load to proceed.

```java
webView.setWebViewClient(new WebViewClient() {
            ProgressDialog progressDialog = new ProgressDialog(Context);

            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                progressDialog.setTitle("Loading...");
                progressDialog.setMessage("Please wait...");
                progressDialog.setCancelable(false);
                progressDialog.show();
            }

            @Override
            public void onPageCommitVisible(WebView view, String url) {
                super.onPageCommitVisible(view, url);
                if (progressDialog != null){
                    progressDialog.dismiss();
                }
            }

        });
```        

```kotlin
webView.setWebViewClient(object : WebViewClient() {
            val progressDialog: ProgressDialog? = ProgressDialog(Context)
            override fun onPageStarted(
                view: WebView,
                url: String,
                favicon: Bitmap
            ) {
                super.onPageStarted(view, url, favicon)
                progressDialog.setTitle("Loading...")
                progressDialog.setMessage("Please wait...")
                progressDialog.setCancelable(false)
                progressDialog.show()
            }

            override fun onPageCommitVisible(
                view: WebView,
                url: String
            ) {
                super.onPageCommitVisible(view, url)
                if (progressDialog != null) {
                    progressDialog.dismiss()
                }
            }
        })
```

### Sharing cookies between WebViews and networking clients

WebViews currently use their own cookie manager, which means that any network requests you make outside of these web views are usually stored separately.  This can cause problems when trying to retain the same cookies (i.e. for authentication or cross-site script forgery (CSRF) headers).  The simplest approach as proposed in this [Stack Overflow article](https://stackoverflow.com/questions/18057624/two-way-sync-for-cookies-between-httpurlconnection-java-net-cookiemanager-and) is to implement a cookie handler that forwards all requests to the WebView cookie store.  See this [gist](https://gist.github.com/rogerhu/5e2fa5725487d3ce0529) for an example.
 
## References

* <https://developer.android.com/guide/webapps/webview.html>
* <https://developer.android.com/guide/webapps/best-practices.html>
* <https://www.javacodegeeks.com/2013/06/fragment-in-android-tutorial-with-example-using-webview.html>
* <https://www.tutorialspoint.com/android/android_webview_layout.htm>
* <https://www.mkyong.com/android/android-webview-example/>
* <https://stackoverflow.com/questions/16849347/adding-a-progress-dialog-in-a-webview>
* <https://www.androidcoding.in/2016/03/17/android-tutorial-webview-beginners/>