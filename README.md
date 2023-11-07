# MBFUniversalSDK

[![Maven Central](https://img.shields.io/maven-central/v/vn.mobifone/universal-sdk.svg?label=Maven%20Central)](https://mvnrepository.com/artifact/vn.mobifone.adhub/MBFUniversalSDK)

Ads and Analytics Framework

## Prerequisites

- `minSdkVersion` of 21 or higher
- `compileSdkVersion` of 33 or higher

## Installation

MBFUniversalSDK is available through MVN. To install it, add mavenCentral to your repositories if it's not added yet.

```javascript
repositories {
    mavenCentral()
}
```

Import MBFUniversalSDK dependency

```javascript
dependencies {
    implementation 'vn.mobifone.adhub:MBFUniversalSDK:x.x.x'
}
```

Add this code to app manifest, wrapped in `<application>` tag and replace FILL_YOUR_WRITE_KEY_HERE

```xml
<meta-data android:name="vn.mobifone.sdk.WRITE_KEY" android:value="FILL_YOUR_WRITE_KEY_HERE" />
```

By default, we use same WRITE_KEY for both frameworks. If you would like use WRITE_KEY for AdNetwork, please add new one meta-data for it.

```xml
<meta-data android:name="vn.mobifone.sdk.WRITE_KEY_ADNETWORK" android:value="FILL_YOUR_WRITE_KEY_HERE" />
```

## Usage

### AdNetwork

#### Banner Ad

```javascript
// Create AdView with size and type is banner
val adView = AdView(requireContext())
adView.apply {
    id = View.generateViewId()
    size = AdSize.MEDIUM_RECTANGLE
    unitId = YOUR_INVENTORY_ID
}

// Add AdView to your layout
val parent = binding.root as ConstraintLayout
parent.addView(adView)

// Perform loadAd with a request
val adRequest = AdRequest.Builder().build()
adView.loadAd(adRequest)
```

**Predefined sizes:**

|      AdSize      | Width x Height |
| :--------------: | :------------: |
|      BANNER      |     320x50     |
|   FULL_BANNER    |     468x60     |
|   LARGE_BANNER   |    320x100     |
|    RECTANGLE     |    250x250     |
| MEDIUM_RECTANGLE |    300x250     |
|      VIDEO       |    480x360     |

Or custom any size in format `widthxheight`. Example: 640x480, 300x500...

#### Adaptive Banner Ad

Create AdView with size and type is banner programmatically

```javascript
val adView = AdView(requireContext())
adView.apply {
    id = View.generateViewId()
    size = AdSize.MEDIUM_RECTANGLE
    unitId = YOUR_INVENTORY_ID
}
```

Or create in XML, it's `layout_width` & `layout_height` must be `wrap_content`

```xml
<vn.mobifone.sdk.adnetwork.ads.AdView
    android:id="@+id/adBanner"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

Make AdView full width

```javascript
val maxWidth = resources.displayMetrics.widthPixels

adView.adaptiveSize(maxWidth) // Or any dimension in pixel unit
```

#### Video Ad

##### Load Ad VAST Tag Url

```javascript
// Create VideoAdLoader with size and type is video
val videoAdLoader = VideoAdLoader.Builder(
    requireContext(),
    adUnitID = YOUR_INVENTORY_ID,
    adSize = AdSize.VIDEO
).build()

// Set listener to retrieve data later
videoAdLoader.listener = this

// Perform loadAd with a request
val adRequest = AdRequest.Builder().build()
videoAdLoader.loadAd(adRequest)
```

##### Using Native Player to player Content and Ad

Please make sure exclude ExoPlayer module in MBFUniversalSDK and use it from your dependency instead of

```javascript
implementation ('vn.mobifone:universal-sdk:x.x.x') {
    exclude group: 'androidx.media3', module: 'media3-exoplayer-ima'
}
```

Define your player in layout

```xml
<com.google.android.exoplayer2.ui.PlayerView
    android:id="@+id/player_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"/>
```

Declare Player and AdsLoader

```javascript
private lateinit var imaAdsLoader: ImaAdsLoader
private var player: Player? = null
```

Init player

```javascript
private fun initializePlayer(vastTagUrl: String) {
    val dataSourceFactory = DefaultDataSource.Factory(requireContext())
    val mediaSourceFactory = DefaultMediaSourceFactory(dataSourceFactory).apply {
        setAdsLoaderProvider { imaAdsLoader }
        setAdViewProvider { binding.playerView }
    }

    player = ExoPlayer.Builder(requireContext()).apply {
        setMediaSourceFactory(mediaSourceFactory)
    }.build()
    binding.playerView.player = player
    imaAdsLoader.setPlayer(player)

    val contentUri = Uri.parse("<<<YOUR CONTENT URL>>>")
    val adTagUri = Uri.parse(vastTagUrl)

    val adsConfiguration = MediaItem.AdsConfiguration.Builder(adTagUri).build()
    val mediaItem = MediaItem.Builder().apply {
        setUri(contentUri)
        setAdsConfiguration(adsConfiguration)
    }.build()

    player?.apply {
        setMediaItem(mediaItem)
        prepare()
        playWhenReady = false
    }
}
```

Listen event video Ad loaded and perform playing

```javascript
videoAdLoader.listener = object : VideoAdListener {
    override fun onVideoAdLoaded(adUnitID: Int, bid: Bid, vastTagUrl: String) {
        // Call init with vast tag url
        initializePlayer(vastTagUrl)

        // Play Video Ad
        binding.playerView.onResume()
    }

    override fun onVideoAdFailedToLoad(adUnitID: Int, error: String) {
        Log.d("VideoAdFragment","Video Ad did fail to load with error:: $error")
    }
}
```

Setup listener for events

```javascript
playerView.listener = object : IMAPlayerView.PlayerViewListener {
    override fun onImaAdsLoaded(adUnitID: Int, bid: Bid, vastTagUrl: String) {
        Log.d("onImaAdsLoaded","Video Ad Content URL: $vastTagUrl")
    }

    override fun onImaAdsFailedToLoad(adUnitID: Int, error: String) {
        Log.d("onImaAdsFailedToLoad","Video Ad did fail to load with error: $error")
    }

    override fun onAdEvent(adEventName: String?) {
        Log.d("VideoAdFragment","Ad Event: $adEventName")
    }

    override fun onAdErrorEvent(errorTypeName: String?) {
        Log.d("VideoAdFragment","Ad Error Event: $errorTypeName")
    }
}
```

Don't forget release to prevent leak memory when screen is destroyed

```javascript
playerView.releasePlayer();
```

#### Native Ad

##### Using forNativeAd callback

NativeAdLoader will fetch nativeAd data and then binding to your views manually

```javascript
val nativeAdLoader = NativeAdLoader.Builder(requireContext(), YOUR_INVENTORY_ID)
    .forNativeAd { nativeAd ->
        populateNativeAdView(nativeAd)
    }
    .build()


// Perform load Ad

nativeAdLoader.loadAd(AdRequest.Builder().build())
```

| NativeAd Properties |  Type  |
| :-----------------: | :----: |
|        title        | String |
|     description     | String |
|       iconUrl       | String |
|    mainImageUrl     | String |
|   mainImageRatio    | Float  |
|   ctaDescription    | String |
|      sponsored      | String |

Example

```javascript
private fun populateNativeAdView(nativeAd: NativeAd) {

    nativeAd.title?.let { title ->
        tvTitle.text = title
    }

    nativeAd.description?.let { description ->
        tvDescription.text = description
    }

    nativeAd.mainImageUrl?.let { imageUrl ->
        imgMain.load(imageUrl)
    }

    nativeAd.iconUrl?.let { iconUrl ->
        imgIcon.load(iconUrl)
    }

    nativeAd.ctaDescription?.let { ctaDescription ->
        btnCTA.text = ctaDescription
        btnCTA.setOnClickListener {
            // Perform Ad Clicked
            nativeAd.performAdClicked(requireContext()) { uri ->
                return@performAdClicked true // If you'd like handle uri, please return false
            }
        }
    }
}
```

##### Using bindingIDs method

NativeAdLoader will fetch nativeAd data and binding to your view automatically. Please make sure your view id is matching NativeAD.ID constants.

```javascript
val nativeAdLoader = NativeAdLoader.Builder(requireContext(), YOUR_INVENTORY_ID)
    .bindingIDs(
        mutableMapOf(
            NativeAd.ID_TITLE               to R.id.tv_title,
            NativeAd.ID_DESCRIPTION         to R.id.tv_body,
            NativeAd.ID_CTA_DESCRIPTION     to R.id.btn_cta,
            NativeAd.ID_IMAGE_ICON          to R.id.img_icon,
            NativeAd.ID_IMAGE_MAIN          to R.id.img_main
        )
    )
    .build()

// Perform load Ad

adLoader.loadAd(AdRequest.Builder().build())
```

For listening Ad Click event, register AdListener to AdLoader

Example

```javascript
nativeAdLoader.adListener = object : AdListener {
    override fun onAdLoaded(adUnitID: Int) {
        Log.d("AdLoader", "Ad Loaded")
    }

    override fun onAdFailedToLoad(adUnitID: Int, error: String) {
        Log.d("AdLoader", "Ad did fail to load with error: $error")

    }

    override fun callToActionPerformed(uri: Uri?): Boolean {
        Log.d("AdLoader", "callToActionPerformed: $uri")
        return true
    }

}
```

##### Using Native Ad templates

MBFUniversalSDK provides three types of template: DEFAULT, MEDIUM, ARTICLE

Define NativeAdTemplateView in your layout xml file

```xml
<vn.mobifone.sdk.adnetwork.ads.NativeAdTemplateView
    android:id="@+id/native_ad_view_default"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:template="DEFAULT" />
```

In Kotlin code, using NativeAdLoader to fetch NativeAd data then binding `withTemplateView` method

```javascript
val nativeAdView = findViewById<NativeAdTemplateView>(R.id.native_ad_view_default)

val nativeAdLoader = NativeAdLoader.Builder(requireContext(), YOUR_INVENTORY_ID)
    .withTemplateView(nativeAdView)
    .build()


nativeAdLoader.loadAd(AdRequest.Builder().build())
```

Override Ad Clicked event if needed

```javascript

nativeAdView.setOnAdClickListener { _, uri ->
    Log.d("NativeAdView", "Ad Clicked: $uri")
    false
}
```

#### Using AdRequest

```javascript
// Create AdView
// ...
// Perform loadAd with a request
val screenContext = HashMap<String, String>()
screenContext["keywords"] = arrayOf("talents", "show", "tv-show").joinToString()
screenContext["title"] = "HomeScreen"

val adRequest = AdRequest.Builder().apply {
    context = screenContext
}.build()
```

#### Custom AdSize

##### Banner

```javascript
val adView = AdView(requireContext())
adView.apply {
    id = View.generateViewId()
    size = AdSize.forCustomBanner(300, 200)
    unitId = YOUR_INVENTORY_ID
}
```

##### Video

```javascript
// Create PlayerView...

playerView.requestAd(
  YOUR_INVENTORY_ID,
  adRequest,
  AdSize.forCustomVideo(480, 360),
);
```

```javascript
// Create VideoAdLoader
val videoAdLoader = VideoAdLoader.Builder(
    requireContext(),
    adUnitID = YOUR_INVENTORY_ID,
    adSize = AdSize.forCustomVideo(480, 360)
).build()
```

#### Listen Ad Events

##### For Banner Ad - AdListener

```javascript
adView.adListener = object: AdListener {
    override fun onAdLoaded(adUnitID: Int, bid: Bid) {
        Log.d("AdView", "Ad Loaded with bid id: ${bid.id}")
    }

    override fun onAdFailedToLoad(adUnitID: Int, error: String) {
        Log.d("AdView", "Ad did fail to load with error:: $error")
    }

    override fun shouldOverrideOnAdClicked(view: BaseAdView, uri: Uri?): Boolean {
        Log.d("AdView", "Click on Ad")
        return super.shouldOverrideOnAdClicked(view, uri)
    }
}
```

##### For Video Ad - VideoAdListener

```javascript
videoAdLoader.listener = object : VideoAdListener {
    override fun onVideoAdLoaded(adUnitID: Int, bid: Bid, vastTagUrl: String) {
        Log.d("VideoAdFragment","Video Ad Content URL: $vastTagUrl")
        binding.textAdContentUrl.text = vastTagUrl
    }

    override fun onVideoAdFailedToLoad(adUnitID: Int, error: String) {
        Log.d("VideoAdFragment","Video Ad did fail to load with error:: $error")
    }

    override fun onAdEvent(adEventName: String?) {
        Log.d("VideoAdFragment","Ad Event: $adEventName")
    }

    override fun onAdErrorEvent(errorTypeName: String?) {
        Log.d("VideoAdFragment","Ad Error Event: $errorTypeName")
    }
}
```

#### Remove Ad

##### For Banner Ad

```javascript
// Remove AdView when loading Ad failed

binding.root.removeView(adView);
```

### Analytics

Analytics will be initialized automatically and collect data for you. You can also manually track events with the following methods:

#### Track Events

```javascript
MBF.track(context: Context, event: String, properties: Properties?)
```

To track an event, you need to call the `MBF.track()` method and pass in three parameters: `context`, `name` and `properties`.

- The `context` parameter is the current context of your app, for example: `this` or `applicationContext`.
- The `name` parameter is a string to name the event, for example: "Post view", "Sign up", "Purchase", etc.
- The `properties` parameter is a `Properties` object to contain detailed information about the event, for example: post title, product type, order value, etc.

```javascript
// Create a Properties object and set the properties of the event
val postViewEventProperties = Properties().apply {
    put("title", "Post Title")
    put("category", "Category 1, Category 2")
    put("keyword", "Keyword 1, Keyword 2, ...")
    ...
}

// Track the event "Post view" with the properties created
MBF.track(context, "Post view", postViewEventProperties)
```

#### Identify Events

```javascript
MBF.identify(context: Context, userId: String, traits: Traits?)
```

To identify a user, you need to call the `MBF.identify()` method and pass in two parameters: `context` and `userId`.

- The `context` parameter is the current context of your app, for example: `this` or `applicationContext`.
- The `userId` parameter is a string to identify your user, for example: "PartnerUserID-01".
- You can also pass in another `Traits` object to contain additional information about the user, for example: user name, user type, etc.

```javascript
// Create a Traits object and set the properties of the user
val userTraits = Traits().apply {
    put("userName", "Username 1")
    put("userType", "Normal")
    ...
}

// Identify user with user ID and properties created
MBF.identify(context, "PartnerUserID-01", userTraits)
```

#### Screen Events

```javascript
MBF.screen(context: Context, name: String, properties: Properties?)
```

To track a screen, you need to call the `MBF.screen()` method and pass in two parameters: `context` and `title`.

- The `context` parameter is the current context of your app, for example: `this` or `applicationContext`.
- The `title` parameter is a string to name the screen, for example: "Login Screen", "Home Screen", etc.
- You can also pass in another `Properties` object to contain detailed information about the screen, for example: login method, number of posts, etc.

```javascript
// Create a Properties object and set the properties of the screen
val loginScreenProperties = Properties().apply {
    put("loginMethod", "FACEBOOK/GOOGLE/OTP/QR")
    ...
}

// Track screen with screen name and properties created
MBF.screen(context, "LoginScreen", loginScreenProperties)
```

## Author

AiACTIV TECH, tech@aiactiv.io

## License

MBFUniversalSDK is available under the MIT license. See the LICENSE file for more info.
