Install Command
```bash
composer require "kreait/firebase-php:^7.0"
```

or

```bash
composer require kreait/laravel-firebase
```

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
