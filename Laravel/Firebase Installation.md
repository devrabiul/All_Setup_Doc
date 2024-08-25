Install Kreait Firebase SDK in Laravel

```bash
composer require kreait/firebase-php
```

Step 2: Configure Firebase in Laravel
Create a Firebase Project in the Firebase Console and add a new Web App to get the configuration file (google-services.json).

Add the Firebase service account credentials:

Download the service-account-file.json from the Firebase Console.
Save this file in your Laravel project, preferably in the storage/app/ directory.
Add Firebase configuration in config/services.php:

```bash
return [
    // Other service configurations...

    'firebase' => [
        'credentials' => storage_path('app/service-account-file.json'),
    ],
];
```

Step 3: Create a Service Provider for Firebase
Create a Firebase service provider:

```bash
php artisan make:provider FirebaseServiceProvider
```

Register Firebase in the service provider:
```bash
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Kreait\Firebase\Factory;

class FirebaseServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(\Kreait\Firebase\Factory::class, function ($app) {
            return (new Factory)
                ->withServiceAccount(config('services.firebase.credentials'));
        });

        $this->app->singleton(\Kreait\Firebase\Auth::class, function ($app) {
            return $app->make(\Kreait\Firebase\Factory::class)->createAuth();
        });

        $this->app->singleton(\Kreait\Firebase\Messaging::class, function ($app) {
            return $app->make(\Kreait\Firebase\Factory::class)->createMessaging();
        });
    }

    /**
     * Bootstrap services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }
}
```

Register the provider in config/app.php:
```bash
'providers' => [
    // Other service providers...
    App\Providers\FirebaseServiceProvider::class,
],
```

Step 4: Example Usage
Sending a Notification:
```bash
use Kreait\Firebase\Messaging\CloudMessage;
use Kreait\Firebase\Messaging\Notification;
use Kreait\Firebase\Messaging;

$messaging = app(Messaging::class);

$message = CloudMessage::withTarget('token', $deviceToken)
    ->withNotification(Notification::create('Title', 'Body'));

$messaging->send($message);
```

Subscribing a user to a topic:
```bash
$messaging = app(Messaging::class);

$topic = 'news';
$deviceToken = 'your-device-token';

$messaging->subscribeToTopic($topic, $deviceToken);
```

Sending a message to a topic:
```bash
use Kreait\Firebase\Messaging\CloudMessage;
use Kreait\Firebase\Messaging\Notification;
use Kreait\Firebase\Messaging;

$messaging = app(Messaging::class);

$message = CloudMessage::withTarget('topic', 'news')
    ->withNotification(Notification::create('News Update', 'New content available!'));

$messaging->send($message);
```

tep 5: Testing and Debugging
Test the Firebase integration by creating routes or controller actions that trigger the Firebase functionality.

Log errors and inspect Firebase responses to ensure that everything is working correctly.

Step 6: Optional Enhancements
Create a custom Facade for easier access to Firebase services.
Implement dependency injection in your controllers to manage Firebase services efficiently.
Use environment variables for sensitive data like service account paths.
Summary
You've now successfully integrated Kreait Firebase PHP SDK into your Laravel project, allowing you to send notifications, manage topics, and interact with Firebase features programmatically.

Let me know if you need further assistance!




Create File
```bash
firebase-messaging-sw.js
```

```bash
importScripts('https://www.gstatic.com/firebasejs/8.3.2/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/8.3.2/firebase-messaging.js');

firebase.initializeApp({
    apiKey: "",
    authDomain: "",
    projectId: "",
    storageBucket: "",
    messagingSenderId: "",
    appId: "",
    measurementId: ""
});

const messaging = firebase.messaging();
messaging.setBackgroundMessageHandler(function (payload) {
    return self.registration.showNotification(payload.data.title, {
        body: payload.data.body ? payload.data.body : '',
        icon: payload.data.icon ? payload.data.icon : ''
    });
});
```


```bash
@php($fcm_credentials = \App\CentralLogics\Helpers::get_business_settings('fcm_credentials'))
var firebaseConfig = {
    apiKey: "{{isset($fcm_credentials['apiKey']) ? $fcm_credentials['apiKey'] : ''}}",
    authDomain: "{{isset($fcm_credentials['authDomain']) ? $fcm_credentials['authDomain'] : ''}}",
    projectId: "{{isset($fcm_credentials['projectId']) ? $fcm_credentials['projectId'] : ''}}",
    storageBucket: "{{isset($fcm_credentials['storageBucket']) ? $fcm_credentials['storageBucket'] : ''}}",
    messagingSenderId: "{{isset($fcm_credentials['messagingSenderId']) ? $fcm_credentials['messagingSenderId'] : ''}}",
    appId: "{{isset($fcm_credentials['appId']) ? $fcm_credentials['appId'] : ''}}",
    measurementId: "{{isset($fcm_credentials['measurementId']) ? $fcm_credentials['measurementId'] : ''}}"
};
firebase.initializeApp(firebaseConfig);
const messaging = firebase.messaging();
function startFCM() {
    messaging
        .requestPermission()
        .then(function() {
            return messaging.getToken();
        })
        .then(function(token) {
            subscribeTokenToBackend(token, 'admin_message');
        }).catch(function(error) {
            console.error('Error getting permission or token:', error);
    });
}
function subscribeTokenToBackend(token, topic) {
    fetch('{{url('/')}}/subscribeToTopic', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRF-TOKEN': '{{ csrf_token() }}'
        },
        body: JSON.stringify({ token: token, topic: topic })
    }).then(response => {
        if (response.status < 200 || response.status >= 400) {
            return response.text().then(text => {
                throw new Error(`Error subscribing to topic: ${response.status} - ${text}`);
            });
        }
        console.log(`Subscribed to "${topic}"`);
    }).catch(error => {
        console.error('Subscription error:', error);
    });
}
messaging.onMessage(function(payload) {
    console.log(payload.data);
    if(payload.data.order_id && payload.data.type == "order_request"){
        playAudio();
        $('#popup-modal').appendTo("body").modal('show');
    }
});
startFCM();
```

```bash
app/Providers/FirebaseServiceProvider.php
```

```bash
<?php

namespace App\Providers;

use App\CentralLogics\Helpers;
use Illuminate\Support\ServiceProvider;
use Kreait\Firebase\Factory;

class FirebaseServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        $this->app->singleton('firebase.firestore', function ($app) {
            $serviceAccountKey = Helpers::get_business_settings('push_notification_service_file_content')??[];
            if(count($serviceAccountKey)>0){
                $serviceAccount = $serviceAccountKey;
                return (new Factory)
                    ->withServiceAccount($serviceAccount)
                    ->createMessaging();
            }
            return false;
        });

        $this->app->singleton('firebase.messaging', function ($app) {
            $serviceAccountKey = \App\CentralLogics\Helpers::get_business_settings('push_notification_service_file_content')??[];
            if(count($serviceAccountKey)>0){
                $serviceAccount = $serviceAccountKey;
                return (new Factory)
                    ->withServiceAccount($serviceAccount)
                    ->createMessaging();
            }
            return false;
        });
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        //
    }
}
```
