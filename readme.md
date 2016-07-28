# Android Pay on Chrome

[See live code samples](https://paymentrequestguide.firebaseapp.com)

The following guide outlines how to implement Android Pay

#### Review Requirements
- Android Pay is available to customers in the following countries: US / UK / AU / SG
- Review the Android Pay detailed [content policies](https://support.google.com/payments/merchant/answer/75724?payments_to_biz=&rd=1) to make sure your specific goods or services are supported.
- Your [payment processor](https://developers.google.com/android-pay/#processors) needs to support Android Pay tokens.
- For testing, you need to [add a card](https://support.google.com/androidpay/answer/6289372?hl=en&ref_topic=6224823) to Android Pay on your test device.

#### Getting Started
- Download the latest version of [Chrome Dev](https://play.google.com/store/apps/details?id=com.chrome.dev) for Android.
- Enable [this Chrome flag](chrome://flags/#enable-experimental-web-platform-features) to test the PaymentRequest API.
- Because Android Pay in Chrome utilizes the PaymentRequest API, it is essential that you familiarize yourself with the [integration guide](https://developers.google.com/web/fundamentals/primers/payment-request/?hl=en) prior to continuing.
- Work through the demo to get acquainted with the PaymentRequest API.
- Even if you are not an Android developer, it will be useful to acquaint yourself with the [Android Pay in-app APIs](https://developers.google.com/android-pay/android/tutorial).  Because the responses returned by Android Pay are the same on Android & Chrome, the information on response handling is useful.

#### Get started with Android Pay in Test Mode

Any site can begin testing with Android Pay in test mode.

1. We'll start with the (basic implementation of PaymentRequest)[https://github.com/asieke/prguide/blob/master/public/credit_cards.html] that accepts credit cards
2. We’ll be modifying the supportedInstruments object within the PaymentRequest in order to let Chrome know that the website supports Android Pay 

	```javascript
	  var supportedInstruments = [{
	      supportedMethods: ['amex', 'discover','mastercard','visa']
	    }
	  ];
	```

3. Add “android.com/pay” to the above supportedInstruments object

	```javascript
	  var supportedInstruments = [
	    {
	      supportedMethods: ['amex', 'discover','mastercard','visa']
	    },
	    //add android pay instrument details
	    {
	      supportedMethods: ['https://android.com/pay'],
	      data: {
	        environment: 'TEST',
	        allowedCardNetworks: ['AMEX', 'MASTERCARD', 'VISA', 'DISCOVER'],
	        paymentMethodTokenizationParameters: {
	          tokenizationType: 'NETWORK_TOKEN',
	          parameters: {
	            'publicKey': 'BC9u7amr4kFD8qsdxnEfWV7RPDR9v4gLLkx3jfyaGOvxBoEuLZKE0Tt5O/2jMMxJ9axHpAZD2Jhi4E74nqxr944='
	          }
	        }
	      }
	    }
	  ];
	```

4. Let’s walk through the above parameters in a bit more details
   - supportedMethods ['https://android.com/pay'] - represents Android Pay’s payment app identifier; this must be specified exactly in order for Android Pay to work
   - environment: ‘TEST’ - specifies that you will be calling the Android Pay test environment
   - allowedCardNetworks: ['...'] - specifies which card networks constitute a valid Android Pay response
   - paymentMethodTokenizationParameters: {} - for now leave this as NETWORK_TOKEN and add your own public key to encrypt the response.  (See: [How to generate encryption keys](https://developers.google.com/android-pay/integration/gateway-processor-integration#retrieving-the-encrypted-payload)).
5. Once you add the Android Pay object, Chrome can now request Android Pay tokens on your behalf.  The completed code should produce the following UI:
   - The response from PaymentRequest will contain the shipping and contact information as in the examples outlined in the [PaymentRequest integration guide](https://developers.google.com/web/fundamentals/primers/payment-request/?hl=en) but now includes an additional response from Android Pay containing
   - Billing address information
   - Information about the payment instrument
   - Details about the Payment token
6. Putting it all together - [see the completed code example](https://github.com/asieke/prguide/blob/master/public/tutorial.html)

#### Request a Network Token

Now that we have a working prototype that utilizes the Android Pay test environment, it is just a few changes to start requesting network tokens.  Requesting a Network token requires 2 pieces of information to be included in the PaymentRequest.

 - A Merchant ID that maps to your origin
 - A public key used to encrypt the response from Android Pay
 
1. Sign up for a merchant ID from Android Pay - 
   - Add your company, site origin and a company email to the [following form](https://goo.gl/forms/SiKd7GAESCPNg9H83)
   - Google will provide a merchant ID within 24 hours.
2. Acquire a key-pair used to encrypt the response from Android Pay
   - Google recommends you work with your payment processor to obtain a public key.  This simplifies the process as your processor will be able to handle decryption of the Android Pay Payload.  Contact your payment processor for more information.
   - If you want to handle encryption yourself, Please refer to the following guide for generating a base64 encoded Elliptic Curve Integrated Encryption key
3. Modify the TEST environment request to request an Android Pay token

	```javascript
	  var supportedInstruments = [
	    {
	      supportedMethods: ['amex', 'discover','mastercard','visa']
	    },
	    {
	      supportedMethods: ['https://android.com/pay'],
	      data: {
	        //merchant ID obtained from Google that maps to your origin
	        merchantId: '02510116604241796260',
	        allowedCardNetworks: ['AMEX', 'MASTERCARD', 'VISA', 'DISCOVER'],
	        paymentMethodTokenizationParameters: {
	          tokenizationType: 'NETWORK_TOKEN',
	          parameters: {
	            //public key to encrypt response from Android Pay
	            'publicKey': 'BC9u7amr4kFD8qsdxnEfWV7RPDR9v4gLLkx3jfyaGOvxBoEuLZKE0Tt5O/2jMMxJ9axHpAZD2Jhi4E74nqxr944='
	          }
	        }
	      }
	    }
	  ];
	```

4. The UI and response should have the same format as in the TEST mode example
5. Putting it all together - [see the completed code example](https://github.com/asieke/prguide/blob/master/public/android_pay_network.html)

#### Request a Gateway Token

The following example outlines how to request a token directly from your payment gateway.  In this example we outline how to request a stripe token.  If you use Braintree or Vantiv please contact your processor for more detailed instructions.

In requesting a gateway token, you eliminate a network call to your gateway as in the Network model above.  Instead, Android Pay makes a call to your processor on your behalf and returns a chargeable gateway token.

1. Use the same [Android Pay Merchant ID](https://goo.gl/forms/SiKd7GAESCPNg9H83)
2. You do not need to utilize encryption in order to request a Gateway token.
3. Modify the Android Pay request to include your gateway parameters

	```javascript
	  var supportedInstruments = [
	    {
	      supportedMethods: ['amex', 'discover','mastercard','visa']
	    },
	    {
	      supportedMethods: ['https://android.com/pay'],
	      data: {
	        //merchant ID obtained from Google that maps to your origin
	        merchantId: '02510116604241796260',
	        allowedCardNetworks: ['AMEX', 'MASTERCARD', 'VISA', 'DISCOVER'],
	        paymentMethodTokenizationParameters: {
	          tokenizationType: 'GATEWAY_TOKEN',
	          parameters: {
	            'gateway': 'stripe',
	            // Place your own Stripe publishable key here.
	            'stripe:publishableKey': 'pk_live_fD7ggZCtrB0vJNApRX5TyJ9T',
	            'stripe:version': '2016-07-06'
	          }
	        }
	      }
	    }
	  ];
	 ```
	 
4. The response from Chrome/Android Pay now includes a chargeable gateway token in addition to the same billing and contact information provided in the previous examples.
5. Putting it all together - [see the completed code example](https://github.com/asieke/prguide/blob/master/public/android_pay_gateway.html)


