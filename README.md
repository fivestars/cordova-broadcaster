# Cordova Broadcaster

Cordova Plugin to allow global message exchange between javascript and native (and viceversa). Based (forked off of) on [bsorrentino's cordova broadcaster](https://github.com/bsorrentino/cordova-broadcaster)

[![npm](https://img.shields.io/npm/v/cordova-plugin-broadcaster.svg)](https://www.npmjs.com/package/cordova-plugin-broadcaster) [![Join the chat at https://gitter.im/bsorrentino/cordova-broadcaster](https://badges.gitter.im/bsorrentino/cordova-broadcaster.svg)](https://gitter.im/bsorrentino/cordova-broadcaster?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


## Ingredient Technologies

Broadcaster plugin providing bridge for the following native technologies:

  target OS | Native Technology
 ----|----
 IOS | **[NotificationCenter](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/index.html#//apple_ref/occ/instm/NSNotificationCenter/addObserverForName%3aobject%3aqueue%3ausingBlock%3a)**
Android | **[LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)**

## Usage:

### From Native to Javascript (global broadcast)

#### Javascript
```javascript
window.broadcaster.addEventListener(“INTENT_NAME”, sample_func)
var sample_func = function (intent_extras) {
  var sample_extra = intent_extras['EXAMPLE_KEY'];
  console.log(sample_extra); // "EXAMPLE_DATA"
}
```

#### ANDROID

```Java
val sample_intent = Intent(“INTENT_NAME”)
sample_intent.putExtra("EXAMPLE_KEY", "EXAMPLE_DATA")
applicationContext.sendBroadcast(sample_intent)
```

### From Javascript to Native (global broadcast)

#### Javascript

```javascript
window.broadcaster.sendGlobalBroadcast(“INTENT_NAME”, “INTENT_ACTION”, {“example_key”: “example_data”})
 ```

#### ANDROID

```Java
// First create an inner class (non-static) that can be registered dynamically in the code.
// (non-static -> register dynamically programmatically; static -> register in manifest)
inner class MTabBroadcastReceiver : BroadcastReceiver() {
  override fun onReceive(context: Context?, intent: Intent?) {
  // TO GET INTENT_EXTRAS
  val extra = intent?.extras.get(“example_key”) // “example_data”
    when (intent?.action) {
      “INTENT_ACTION” -> {}
      else -> {}
    }
  }
}

// Then, register the receiver in onCreate() or onServiceConnected()
mtabReceiver = MTabBroadcastReceiver()
val mtabIntent = IntentFilter(“INTENT_NAME”) // must match or else it won’t be received
mtabIntent.addAction(“INTENT_ACTION”) // must match or else it won’t be received
registerReceiver(mtabReceiver, mtabIntent)

// Remember to unregister the receiver in onDestroy() to prevent a leak
unregister(mtabReceiver)
```

### From Native to Javascript (locally)

#### Javascript
```javascript
    console.log( "register didShow received!" );

    var listener = function( e ) {
      //log: didShow received! userInfo: {"data":"test"}
      console.log( "didShow received! userInfo: " + JSON.stringify(e)  );
    }

    window.broadcaster.addEventListener( "didShow", listener);
```

#### ANDROID

```Java
final Intent intent = new Intent("didShow");

Bundle b = new Bundle();
b.putString( "data", "test" );
intent.putExtras( b);

LocalBroadcastManager.getInstance(this).sendBroadcastSync(intent);
```

#### IOS

##### Objective-C
```Objective-C
[[NSNotificationCenter defaultCenter] postNotificationName:@"didShow"
                                                    object:nil
                                                  userInfo:@{ @"data":@"test"}];
```

##### Swift
```swift
let nc = NSNotificationCenter.default
nc.post(name:"didShow", object: nil, userInfo: ["data":"test"])
```

#### BROWSER

```javascript

let event = new CustomEvent("didShow", { detail: { data:"test"} } );
document.dispatchEvent( event )

```
### From Javascript to Native (locally)

#### Javascript

```javascript
  window.broadcaster.fireNativeEvent( "test.event", { item:'test data' }, function() {
    console.log( "event fired!" );
    } );
 ```

#### ANDROID

```Java
final BroadcastReceiver receiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String data = intent.getExtras().getString("data");

        Log.d("CDVBroadcaster",
                String.format("Native event [%s] received with data [%s]", intent.getAction(), data));

    }
};

LocalBroadcastManager.getInstance(this)
            .registerReceiver(receiver, new IntentFilter("test.event"));
}
```

#### IOS

##### Objective-C

```Objective-C
[[NSNotificationCenter defaultCenter] addObserverForName:@"test.event"
                                                  object:nil
                                                   queue:[NSOperationQueue mainQueue]
                                              usingBlock:^(NSNotification *notification) {
                                                      NSLog(@"Handled 'test.event' [%@]", notification.userInfo[@"item"]);
                                                    }];
```

##### Swift 3.0

```swift
let nc = NotificationCenter.default
nc.addObserver(forName:Notification.Name(rawValue:"test.event"),
               object:nil, queue:nil) {
  notification in
  print( "\(notification.userInfo)")
}
```

#### BROWSER

```javascript

document.addEventListener( "test.event", ( ev:Event ) => {
  console.log( "test event", ev.detail );
});

```