// Initialize reCAPTCHA instance once
        var recaptchaVerifier;
        var recaptchaContainer = document.getElementById('recaptcha-container');
        if (recaptchaContainer && recaptchaContainer.innerHTML.trim() === "") {
            window.onload = function() {
                recaptchaVerifier = new firebase.auth.RecaptchaVerifier('recaptcha-container', {
                    // size: 'invisible', // ReCaptcha v3 doesn't need interaction
                    callback: function(response) {
                        // Log the response immediately after page load
                        console.log('reCAPTCHA v3 solved automatically:', response);
                        storeRecaptchaVerifierResponse(response);
                    },
                    'expired-callback': function() {
                        console.error('reCAPTCHA expired');
                    }
                });

                // Automatically trigger reCAPTCHA v3 and get the response
                recaptchaVerifier.render().then(function(widgetId) {
                    console.log('reCAPTCHA rendered and solving...');
                    // Here you are getting the token automatically after load
                    return recaptchaVerifier.verify().then(function(response) {
                        // Log the response token immediately after it is generated
                        console.log('reCAPTCHA v3 Response (page load):', response);
                        storeRecaptchaVerifierResponse(response); // Send the response to the server
                    }).catch(function(error) {
                        console.error('Error getting reCAPTCHA v3 response:', error);
                    });
                }).catch(function(error) {
                    console.error('Error rendering reCAPTCHA v3:', error);
                });
            };
        } else {
            console.log("reCAPTCHA container is either not found or already contains inner elements!");
        }

        // function storeRecaptchaVerifierResponse(response) {
        //     console.log('reCAPTCHA Response:', response);
        //     $.ajax({
        //         url: firebaseConfigurationConfig.data("recaptcha-store"),
        //         method: "POST",
        //         data: {
        //             _token: $('meta[name="_token"]').attr("content"),
        //             g_recaptcha_response: response,
        //         },
        //         beforeSend: function () {
        //         },
        //         success: function (response) {
        //             console.log('reCAPTCHA: ', response);
        //         },
        //         complete: function () {
        //
        //         },
        //     });
        // }

        let confirmationResult;
        function sendFirebasePhoneVerificationCode(phoneNumber, redirectURl = null) {
            console.log(phoneNumber)
            console.log(recaptchaVerifier)
            // Use the pre-initialized recaptchaVerifier
            firebase.auth().signInWithPhoneNumber(phoneNumber, recaptchaVerifier)
                .then((result) => {
                    confirmationResult = result;
                    console.log('Confirmation Result:', result);
                    console.log('Session Info:', result.verificationId);
                    window.confirmationResult = result;
                    callbackFirebasePhoneVerificationCode('success', phoneNumber, result.verificationId, null)
                    // return {
                    //     status: 'success',
                    //     phone: phoneNumber,
                    //     sessionInfo: result.verificationId,
                    //     error: null,
                    // };
                }).catch((error) => {
                console.error('Error during signInWithPhoneNumber', error);
                callbackFirebasePhoneVerificationCode('error', phoneNumber, null, error)
                return {
                    status: 'error',
                    phone: phoneNumber,
                    sessionInfo: null,
                    error: error,
                };
            });
        }
