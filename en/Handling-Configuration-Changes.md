## Overview

There are various situations such as when the screen orientation is rotated where the Activity can actually be destroyed and removed from memory and then re-created from scratch again. In these situations, the best practice is to prepare for cases where the Activity is re-created by properly saving and restoring the state.

## Saving and Restoring Activity State

As your activity begins to stop, the system calls `onSaveInstanceState()` so your activity can save state information with a collection of key-value pairs. The default implementation of this method **automatically saves** information about the state of the activity's **view hierarchy**, such as the text in an `EditText` widget or the scroll position of a `ListView`.

To save additional state information for your activity, you must implement `onSaveInstanceState()` and add key-value pairs to the Bundle object. For example:

```java
public class MainActivity extends Activity {
    static final String SOME_VALUE = "int_value";
    static final String SOME_OTHER_VALUE = "string_value";

    @Override
    protected void onSaveInstanceState(Bundle savedInstanceState) {
        // Save custom values into the bundle
        savedInstanceState.putInt(SOME_VALUE, someIntValue);
        savedInstanceState.putString(SOME_OTHER_VALUE, someStringValue);
        // Always call the superclass so it can save the view hierarchy state
        super.onSaveInstanceState(savedInstanceState);
    }
}
```

The system will call that method before an Activity is destroyed. Then later the system will call `onRestoreInstanceState` where we can restore state from the bundle:

```java
@Override
protected void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);
    // Restore state members from saved instance
    someIntValue = savedInstanceState.getInt(SOME_VALUE);
    someStringValue = savedInstanceState.getString(SOME_OTHER_VALUE);
}
```

Instance state can also be restored in the standard `Activity#onCreate` method but it is convenient to do it in `onRestoreInstanceState` which ensures all of the initialization has been done and allows subclasses to decide whether to use the default implementation. Read [this stackoverflow post](http://stackoverflow.com/a/14676555/313399) for details.

Note that `onSaveInstanceState` and `onRestoreInstanceState` are not guaranteed to be called together. Android invokes `onSaveInstanceState()` when there's a chance the activity might be destroyed. However, there are cases where `onSaveInstanceState` is called but the activity is not destroyed and as a result `onRestoreInstanceState` is not invoked.

Read more on the [Recreating an Activity](http://developer.android.com/training/basics/activity-lifecycle/recreating.html) guide.

## Saving and Restoring Fragment State

Fragments also have a `onSaveInstanceState()` method which is called when their state needs to be saved:

```java
public class MySimpleFragment extends Fragment {
    private int someStateValue;
    private final String SOME_VALUE_KEY = "someValueToSave";
   
    // Fires when a configuration change occurs and fragment needs to save state
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        outState.putInt(SOME_VALUE_KEY, someStateValue);
        super.onSaveInstanceState(outState);
    }
}
```

Then we can pull data out of this saved state in `onCreateView`:

```java
public class MySimpleFragment extends Fragment {
   // ...

   // Inflate the view for the fragment based on layout XML
   @Override
   public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.my_simple_fragment, container, false);
        if (savedInstanceState != null) {
            someStateValue = savedInstanceState.getInt(SOME_VALUE_KEY);
            // Do something with value if needed
        }
        return view;
   }
}
```

For the fragment state to be saved properly, we need to be sure that we aren't **unnecessarily recreating the fragment** on configuration changes. This means being careful not to reinitialize existing fragments when they already exist. Any fragments being initialized in an Activity need to be **looked up by tag** after a configuration change:

```java
public class ParentActivity extends AppCompatActivity {
    private MySimpleFragment fragmentSimple;
    private final String SIMPLE_FRAGMENT_TAG = "myfragmenttag";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        if (savedInstanceState != null) { // saved instance state, fragment may exist
           // look up the instance that already exists by tag
           fragmentSimple = (MySimpleFragment)  
              getSupportFragmentManager().findFragmentByTag(SIMPLE_FRAGMENT_TAG);
        } else if (fragmentSimple == null) { 
           // only create fragment if they haven't been instantiated already
           fragmentSimple = new MySimpleFragment();
        }
    }
}
```

This requires us to be careful to **include a tag for lookup** whenever putting a fragment into the activity within a transaction:

```java
public class ParentActivity extends AppCompatActivity {
    private MySimpleFragment fragmentSimple;
    private final String SIMPLE_FRAGMENT_TAG = "myfragmenttag";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ... fragment lookup or instantation from above...
        // Always add a tag to a fragment being inserted into container
        if (!fragmentSimple.isInLayout()) {
            getSupportFragmentManager()
                .beginTransaction()
                .replace(R.id.container, fragmentSimple, SIMPLE_FRAGMENT_TAG)
                .commit();
        }
    }
}
```

With this simple pattern, we can properly re-use fragments and restore their state across configuration changes.

## Retaining Fragments

In many cases, we can avoid problems when an Activity is re-created by simply using fragments. If your views and state are within a fragment, we can easily have the fragment be retained when the activity is re-created:

```java
public class RetainedFragment extends Fragment {
    // data object we want to retain
    private MyDataObject data;

    // this method is only called once for this fragment
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // retain this fragment when activity is re-initialized
        setRetainInstance(true);
    }

    public void setData(MyDataObject data) {
        this.data = data;
    }

    public MyDataObject getData() {
        return data;
    }
}
```

This approach keeps the fragment from being destroyed during the activity lifecycle.  They are instead retained inside the Fragment Manager.  See the Android official docs for more [information](http://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject).  

Now you can **check to see if the fragment already exists by tag** before creating one and the fragment will retain it's state across configuration changes. See the [Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject) guide for more details.

## Properly Handling List State

### ListView

Often when you rotate the screen, the app will lose the scroll position and other state of any lists on screen. To properly retain this state for `ListView`, you can store the instance state `onPause` and restore `onViewCreated` as shown below:

```java
// YourActivity.java
private static final String LIST_STATE = "listState";
private Parcelable mListState = null;

// Write list state to bundle
@Override
protected void onSaveInstanceState(Bundle state) {
    super.onSaveInstanceState(state);
    mListState = getListView().onSaveInstanceState();
    state.putParcelable(LIST_STATE, mListState);
}

// Restore list state from bundle
@Override
protected void onRestoreInstanceState(Bundle state) {
    super.onRestoreInstanceState(state);
    mListState = state.getParcelable(LIST_STATE);
}


@Override
protected void onResume() {
    super.onResume();
    loadData(); // make sure data has been reloaded into adapter first
    // ONLY call this part once the data items have been loaded back into the adapter
    // for example, inside a success callback from the network
    if (mListState != null) {
        myListView.onRestoreInstanceState(mListState);
        mListState = null;
    }
}
```

Check out this [blog post](https://futurestud.io/tutorials/how-to-save-and-restore-the-scroll-position-and-state-of-a-android-listview) and [stackoverflow post](http://stackoverflow.com/a/5688490) for more details. 

Note that **you must load the items back into the adapter first** before calling `onRestoreInstanceState`. In other words, don't call `onRestoreInstanceState` on the ListView until after the items are loaded back in from the network or the database. 

### RecyclerView

Often when you rotate the screen, the app will lose the scroll position and other state of any lists on screen. To properly retain this state for `RecyclerView`, you can store the instance state `onPause` and restore `onViewCreated` as shown below:

```java
// YourActivity.java
public final static String LIST_STATE_KEY = "recycler_list_state";
Parcelable listState;

protected void onSaveInstanceState(Bundle state) {
     super.onSaveInstanceState(state);
     // Save list state
     listState = mLayoutManager.onSaveInstanceState();
     state.putParcelable(LIST_STATE_KEY, listState);
}

protected void onRestoreInstanceState(Bundle state) {
    super.onRestoreInstanceState(state);
    // Retrieve list state and list/item positions
    if(state != null)
        listState = state.getParcelable(LIST_STATE_KEY);
}

@Override
protected void onResume() {
    super.onResume();
    if (listState != null) {
        mLayoutManager.onRestoreInstanceState(listState);
    }
}
```

Check out this [blog post](http://panavtec.me/retain-restore-recycler-view-scroll-position) and [stackoverflow post](http://stackoverflow.com/a/28262885) for more details. 

## Locking Screen Orientation

If you want to lock the screen orientation change of any screen (activity) of your android application you just need to set the `android:screenOrientation` property of an `<activity>` within the `AndroidManifest.xml`:

```xml
<activity
    android:name="com.techblogon.screenorientationexample.MainActivity"
    android:screenOrientation="portrait"
    android:label="@string/app_name" >
    <!-- ... -->
</activity>
```

Now that activity is forced to always be displayed in "portrait" mode. 

## Manually Managing Configuration Changes

If your application doesn't need to update resources during a specific configuration change and you have a performance limitation that requires you to avoid the activity restart, then you can declare that your activity handles the configuration change itself, which prevents the system from restarting your activity. 

However, this technique should be considered **a last resort** when you must avoid restarts due to a configuration change and is not recommended for most applications. To take this approach, we must add the `android:configChanges` node to the activity within the `AndroidManifest.xml`:

```xml
<activity android:name=".MyActivity"
          android:configChanges="orientation|screenSize|keyboardHidden"
          android:label="@string/app_name">
```

Now, when one of these configurations change, the activity does not restart but instead receives a call to `onConfigurationChanged()`:

```java
// Within the activity which receives these changes
// Checks the current device orientation, and toasts accordingly
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);

    // Checks the orientation of the screen
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
        Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
    }
}
```

See the [Handling the Change](http://developer.android.com/guide/topics/resources/runtime-changes.html#HandlingTheChange) docs. For more about which configuration changes you can handle in your activity, see the [android:configChanges](http://developer.android.com/guide/topics/manifest/activity-element.html#config) documentation and the [Configuration](http://developer.android.com/reference/android/content/res/Configuration.html) class.

## References

* <http://developer.android.com/guide/topics/resources/runtime-changes.html>
* <http://developer.android.com/training/basics/activity-lifecycle/recreating.html>
* <http://www.vogella.com/tutorials/AndroidLifeCycle/article.html#configurationchange>
* <http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html>
* <http://www.intertech.com/Blog/saving-and-retrieving-android-instance-state-part-1/>
* <http://sunil-android.blogspot.com/2013/03/save-and-restore-instance-state.html>
* <https://medium.com/google-developers/activity-revival-and-the-case-of-the-rotating-device-167e34f9a30d#.nq3b23lxg>