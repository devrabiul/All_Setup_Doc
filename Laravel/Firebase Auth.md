```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Firebase OTP Authentication</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <script src="{{ asset('assets/firebase.min.js') }}"></script>
    <script src="https://www.gstatic.com/firebasejs/8.3.2/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.3.2/firebase-auth.js"></script>
    <script>
        // Your web app's Firebase configuration
        const firebaseConfig = {
            apiKey: "AIzaSyAhMz6lRGLf-aoPhf4KE9raM87Y4s_Aa1s",
            authDomain: "endpoint-2050.firebaseapp.com",
            projectId: "endpoint-2050",
            storageBucket: "endpoint-2050.appspot.com",
            messagingSenderId: "541387409103",
            appId: "1:541387409103:web:997dc0b2a1886abebbd9ba",
            measurementId: "G-LX3X2MT9ZN"
        };
        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
    </script>
</head>

<body>
    <div class="container mt-5 d-flex justify-content-center">
        <!-- Card for OTP Verification -->
        <div class="card shadow-lg" style="max-width: 400px; width: 100%;">
            <div class="card-header text-center bg-primary text-white">
                <h5 class="mb-0">Phone OTP Verification</h5>
            </div>
            <div class="card-body">
                <!-- Recaptcha container -->
                <div id="recaptcha-container" class="mb-3"></div>

                <!-- Phone Number Input -->
                <div class="form-group mb-2">
                    <label for="phoneNumber" class="mb-1"><i class="fas fa-phone-alt"></i> Enter Phone Number</label>
                    <input type="text" class="form-control" id="phoneNumber" placeholder="+1 234 567 890">
                </div>

                <!-- Send OTP Button -->
                <div class="d-grid mb-2">
                    <button onclick="sendVerificationCode()" class="btn btn-primary btn-block">Send OTP</button>
                </div>

                <!-- Verification Code Input -->
                <div class="form-group mb-2">
                    <label for="verificationCode" class="mb-1"><i class="fas fa-key"></i> Enter OTP</label>
                    <input type="text" class="form-control" id="verificationCode" placeholder="123456">
                </div>

                <!-- Verify Code Button -->
                <div class="d-grid">
                    <button onclick="verifyCode()" class="btn btn-success btn-block">Verify Code</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        function sendVerificationCode() {
            const phoneNumber = document.getElementById('phoneNumber').value;
            const recaptchaVerifier = new firebase.auth.RecaptchaVerifier('recaptcha-container');
            firebase.auth().signInWithPhoneNumber(phoneNumber, recaptchaVerifier)
                .then((confirmationResult) => {
                    window.confirmationResult = confirmationResult;
                    alert('OTP sent!');
                }).catch((error) => {
                    console.error('Error during signInWithPhoneNumber', error);
                });
        }

        function verifyCode() {
            const code = document.getElementById('verificationCode').value;
            confirmationResult.confirm(code)
                .then((result) => {
                    const user = result.user;
                    alert('User signed in successfully');
                }).catch((error) => {
                    console.error('Error during confirm', error);
                });
        }

    </script>
</body>

</html>

```
