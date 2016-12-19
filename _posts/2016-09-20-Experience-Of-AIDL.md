---
layout: post
title: Experience Of AIDL
date: 2016-09-20 15:25:48
tags: Android
categories: post
---
At first, I must learn Bound Services,and then I could know Android AIDL.

## Bound Services

A bound service allows components(activities) to bind to the service, send requests,receive responses, and even perform interprocess communication(IPC). A bound service typically lives only while it serves another application component and does not run in the background indefinitely.
<!--more-->
## Creating a Bound Service

When creating a service that provides binding, you must provide an IBinder that provides the programming interface that clients can use to interact with the service. There are three ways you can define the interface:

###Extending the Binder class

Example:

```java
public class LocalService extends Service {
    // Binder given to clients
    private final IBinder mBinder = new LocalBinder();
    // Random number generator
    private final Random mGenerator = new Random();

    /**
     * Class used for the client Binder.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    public class LocalBinder extends Binder {
        LocalService getService() {
            // Return this instance of LocalService so clients can call public methods
            return LocalService.this;
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    /** method for clients */
    public int getRandomNumber() {
      return mGenerator.nextInt(100);
    }
}
```

The LocalBinder provides the getService() method for clients to retrieve the current instance of LocalService. This allows clients to call public methods in the service.

Here's an activity that binds to LocalService and calls getRandomNumber() when a button is clicked:

```java
public class BindingActivity extends Activity {
    LocalService mService;
    boolean mBound = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Bind to LocalService
        Intent intent = new Intent(this, LocalService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }

    /** Called when a button is clicked (the button in the layout file attaches to
      * this method with the android:onClick attribute) */
    public void onButtonClick(View v) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call were something that might hang, then this request should
            // occur in a separate thread to avoid slowing down the activity performance.
            int num = mService.getRandomNumber();
            Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
        }
    }

    /** Defines callbacks for service binding, passed to bindService() */
    private ServiceConnection mConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            LocalBinder binder = (LocalBinder) service;
            mService = binder.getService();
            mBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            mBound = false;
        }
    };
}
```

The above sample shows how the client binds to the service using an implementation of ServiceConnection and the onServiceConnected() callback. The next section provides more information about this process of binding to the service.

### Using a Messenger

If you need your service to communicate with remote processes.then you can use a Messenger to provide the interface for your service . This technique allows you to perform interprocess communication(IPC) without the need to use AIDL.

Here's a simple example service that uses a Messenger interface:

```java
public class MessengerService extends Service {
    /** Command to the service to display a message */
    static final int MSG_SAY_HELLO = 1;

    /**
     * Handler of incoming messages from clients.
     */
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_SAY_HELLO:
                    Toast.makeText(getApplicationContext(), "hello!", Toast.LENGTH_SHORT).show();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    final Messenger mMessenger = new Messenger(new IncomingHandler());

    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */
    @Override
    public IBinder onBind(Intent intent) {
        Toast.makeText(getApplicationContext(), "binding", Toast.LENGTH_SHORT).show();
        return mMessenger.getBinder();
    }
}
```
Notice that the handleMessage() method in the Handler is where the service receives the incoming Message and decides what to do, based on the what member.

All that client needs to do is create a Messenger based on the IBinder returned by the service and send a message using send(). For example, this activity that binds to the service and delivers the MSG_SAY_HELLO message to the service:

```java
public class ActivityMessenger extends Activity {
    /** Messenger for communicating with the service. */
    Messenger mService = null;

    /** Flag indicating whether we have called bind on the service. */
    boolean mBound;

    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the object we can use to
            // interact with the service.  We are communicating with the
            // service using a Messenger, so here we get a client-side
            // representation of that from the raw IBinder object.
            mService = new Messenger(service);
            mBound = true;
        }

        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mBound = false;
        }
    };

    public void sayHello(View v) {
        if (!mBound) return;
        // Create and send a message to the service, using a supported 'what' value
        Message msg = Message.obtain(null, MessengerService.MSG_SAY_HELLO, 0, 0);
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Bind to the service
        bindService(new Intent(this, MessengerService.class), mConnection,
            Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }
}
```

So as long, You can see these example that provide two way bind Service and send message.

### Defining an AIDL Interface

#### 1.Create the .aidl file


![aidl_create1](/img/aidl1.png)

here is aidl file,server and client have the same packge name and file name.

![aidl_create2](/img/aidl2.png)


```java
// IRemoteService.aidl
package com.knjin.aidl;
interface IRemoteService {
    int add(int num1,int num2);
}
```

When defining your service interface, be aware that:

Methods can take zero or more parameters, and return a value or void.

* All non-primitive parameters require a directional tag indicating which way the data goes. Either in, out, or inout (see the example below).
* Primitives are in by default, and cannot be otherwise.
Caution: You should limit the direction to what is truly needed, because marshalling parameters is expensive.
* All code comments included in the .aidl file are included in the generated IBinder interface (except for comments before the import and package statements).
* Only methods are supported; you cannot expose static fields in AIDL.

#### 2. Implement the interface

create a AddService.class at Server application.

```java
public class AddService extends Service {

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return stub;
    }

    IRemoteService.Stub stub = new IRemoteService.Stub() {
        @Override
        public int add(int num1, int num2) throws RemoteException {
            return num1+num2;
        }
    };
}

```

xml in AndroidManifest.xml

```xml
<service android:name=".AddService">
            <intent-filter>
                <action android:name="android.intent.action.AddService"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </service>
```

Now the **stub** is an instance of the **Stub** class, which defines the RPC interface for the service.In the next step, this instance is exposed to clients so they can interact with the service.

There are a few rules you should be aware of when implementing your AIDL interface:

* Incoming calls are not guaranteed to be executed on the main thread, so you need to think about multithreading from the start and properly build your service to be thread-safe.
* By default,RPC calls are synchronous. If you know that the service takes more than a few milliseconds to complete a request,you should not call it from the activity's main thread,because it might hang the application(Android might display an "Applicatiion is Not Responding" dialog)-you should usually call them from a separate thread in the client.
* No exceptions that you throw are sent back to the caller.

#### 3.Expose the interface to clients


Once you've implemented the interface for your service,you need to expose it to clients so they can bind to it.
When the client receives the **IBinder** in the **onServiceConnected()** callback,it must call   *YourServiceInterface.Stub.asInterface(service)*  to cast the returned parameter to *YourServiceInterface* type.

For example:

```java
/**
 * client
 */
public class MainActivity extends AppCompatActivity {
    EditText num1,num2;
    TextView res;
    Button add_btn;
    private IRemoteService mService;
    Handler mHandler = new Handler();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        num1 = (EditText) findViewById(R.id.num1);
        num2 = (EditText) findViewById(R.id.num2);
        res  = (TextView) findViewById(R.id.res);
        add_btn = (Button) findViewById(R.id.add_btn);

        add_btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("android.intent.action.AddService");
                bindService(intent,conn, Context.BIND_AUTO_CREATE);
                if (!TextUtils.isEmpty(num1.getText().toString())&&!TextUtils.isEmpty(num2.getText().toString())) {
                    int a = Integer.valueOf(num1.getText().toString());
                    int b = Integer.valueOf(num2.getText().toString());
                    try {
                        final int result = mService.add(a,b);
                        mHandler.post(new Runnable() {
                            @Override
                            public void run() {
                                res.setText(String.valueOf(result));
                            }
                        });
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }

            }
        });
    }

    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService = IRemoteService.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mService = null;
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(conn);
    }
}
```

### Passing Objects over IPC

For example, here is a Rect.aidl file to create a Rect class that's parcelable:

```java
package android.graphics;

// Declare Rect so AIDL can find it and knows that it implements
// the parcelable protocol.
parcelable Rect;
```

And here is an example of how the Rect class implements the Parcelable protocol.

```java
import android.os.Parcel;
import android.os.Parcelable;

public final class Rect implements Parcelable {
    public int left;
    public int top;
    public int right;
    public int bottom;

    public static final Parcelable.Creator<Rect> CREATOR = new
Parcelable.Creator<Rect>() {
        public Rect createFromParcel(Parcel in) {
            return new Rect(in);
        }

        public Rect[] newArray(int size) {
            return new Rect[size];
        }
    };

    public Rect() {
    }

    private Rect(Parcel in) {
        readFromParcel(in);
    }

    public void writeToParcel(Parcel out) {
        out.writeInt(left);
        out.writeInt(top);
        out.writeInt(right);
        out.writeInt(bottom);
    }

    public void readFromParcel(Parcel in) {
        left = in.readInt();
        top = in.readInt();
        right = in.readInt();
        bottom = in.readInt();
    }
}
```

#### Error

`finished with non-zero exit value 1`

in .aidl parameter have some error,You should limit the direction to what is truly needed, because marshalling parameters is expensive.

**solve** this question.
buildToolsVersion version is too high.
