PayME SDK is a set of libraries for apps to interact with PayME Platform. PayME SDK includes the following main functions:

- Sign up, login, eKYC system through PayME wallet account.
- Deposit and withdrawal function from PayME wallet.
- Integration of services of PayME Platform.

**Some terms**

| | Name | Explanation |
| :--- | :----- | ---------------------------------------------------  |
| 1 | app | Is an iOS/Android mobile app or web that will integrate the SDK to perform the PayME wallet payment function. |
| 2 | SDK | Is a toolkit to support the integration of PayME wallet into the app system. |
| 3 | backend | An integrated system that supports an app, server or api. |
| 4 | AES | AES data encryption function. [Reference](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) |
| 5 | RSA | RSA data encryption algorithm. |
| 6 | IPN | Instant Payment Notification , used to notify between the app's backend and PayME's backend |

## Setting

- npm:

```shell
npm install react-native-payme-sdk
```

- yarn:

```shell
yarn add react-native-payme-sdk
```

#### Android:
- **Update file build.gradle project**
```kotlin
allprojects {
  repositories {
    google()
    jcenter()
    maven {
      url "https://plugins.gradle.org/m2/"
    }
    maven {
      url "https://jitpack.io"
    }
  }
}
```

- **Update file build.gradle Module**

```kotlin
android {
  ...
  packagingOptions {
    exclude "META-INF/DEPENDENCIES"
    exclude "META-INF/LICENSE"
    exclude "META-INF/LICENSE.txt"
    exclude "META-INF/license.txt"
    exclude "META-INF/NOTICE"
    exclude "META-INF/NOTICE.txt"
    exclude "META-INF/notice.txt"
    exclude "META-INF/ASL2.0"
  }
}
```

- **Update file Android Manifest**

If the integrated app uses payCode = VNPAY, please configure the urlscheme to Activity receive payment. When the payment is completed, the VNPAY app can automatically return to the integrated app.

```xml
 <activity
    android:launchMode="singleTask"
    android:windowSoftInputMode="adjustResize"
    android:configChanges="keyboard|keyboardHidden|orientation|screenSize|uiMode"
    android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="apptest"
            android:host="payment.vnpay.result"
            tools:ignore="AppLinkUrlError" />
    </intent-filter>
  </activity>
```

Allow access contact permission when using phone top-up and money transfer.

```xml
  ...
    <uses-permission android:name="android.permission.READ_CONTACTS" />
  ...
```

- **MainActivity.java**

⚠️⚠️⚠️ Removed from version 0.9.16 onwards.

Please remove these 2 functions if you've installed them into your app. If not, please skip this step.

```java
...
import android.content.Intent;
import com.facebook.react.ReactInstanceManager;
import com.reactnativepaymesdk.PaymeSdkModule;

public class MainActivity extends ReactActivity {
  ...
  @Override
  public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    PaymeSdkModule.Companion.onActivityResult(this,requestCode, resultCode, data);
  }

  @Override
  public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    PaymeSdkModule.Companion.onRequestPermissionsResult(requestCode, permissions, grantResults);
  }
}
```


#### iOS:

```shell
cd ios && pod install
```
- **Podfile**
```swift
target 'AppExample' do
  ...
  use_frameworks! :linkage => :static
  ...
end
```

- **Info.plist**

Update the app's Info.plist file with the following keys (the value of the string may change, here are the messages displayed when asking the user to grant the corresponding permission):

Update file Info.plist of the app with the following keys (the value of string may change, there are all messages displayed when asking user to allow the corresponding permission):

```swift
<key>NSCameraUsageDescription</key>
<string>Need to access your camera to capture a photo add and update profile picture.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Need to access your library to add a photo or video of kyc video</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Need to access your photo library to select a photo add and update profile picture</string>
<key>NSContactsUsageDescription</key>
<string>Need to access your contact</string>
```

If you don't use the contacts feature, add it at the end of the podfile

```swift
post_install do |installer|
   installer.pods_project.targets.each do |target|
       if target.name == 'PayMESDK'
           target.build_configurations.each do |config|
             config.build_settings['SWIFT_ACTIVE_COMPILATION_CONDITIONS'] ||= '$(inherited)'
             config.build_settings['SWIFT_ACTIVE_COMPILATION_CONDITIONS'] << 'IGNORE_CONTACT'
           end
       end
   end
end
```

If the iOS project is not configured for Swift, you can add an optional empty .swift file to the project from XCode to automatically enable Bridging header.

1. Open XCode Project
2. File -> New File -> Select .swift file -> Next -> Create -> Select Create Bridging Header

[![img](https://docs-assets.developer.apple.com/published/f86b29eb4b/757d8a73-4e94-4b68-8227-ae32e2ca1304.png)](https://github.com/PayME-Tech/PayME-SDK-Android-Example/blob/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)

## Usage

PayME system will provide the integrated app with the following information:

- **PublicKey** : Used to encrypt data, the integrated app needs to transmit to the SDK for encryption.
- **AppToken** : AppToken provides a unique identifier for each app, needs to transmit to the SDK for encryption.
- **SecretKey** : Used to encrypt and authenticate data in the backend system for the integrated app.

The App will provide the PayME system with the following information:

- **AppPublicKey** : It will sent through PayME's backend system for encryption.
- **AppPrivateKey**: It will transmit to PayME SDK to perform decryption.

Encryption standard: RSA-512bit.

### Initializing the Library

Necessary to call the initialization method only 1 time to initialize the SDK before using PayME SDK.


```javascript
import payME, { ENV, LANGUAGES } from 'react-native-payme-sdk'

payME.init(
      appToken,
      publicKey,
      connectToken,
      privateKey,
      configColor,
      LANGUAGES.VIETNAMESE,
      ENV.SANDBOX
    );
```
#### Constant

| Property           | Type   | Description            |
| ------------------ | ------ | ---------------------- |
`ENV.SANDBOX` | `enum` | Sandbox environment. |
| `ENV.STAGING` | `enum` | Staging environment. |
| `ENV.PRODUCTION` | `enum` | Production environment. |

#### Parameters

| Property | Type | Description |
| -------------- | ---------- | --------------------------------------------------- |
| `appToken` | `string` | AppId provides a unique identifier for each app, it needs to transmit for the SDK for encryption. |
| `publicKey` | `string` | For data encryption, the integrated app needs to transmit it to the SDK for encryption. PayME system provided for the integrated app. |
| `connectToken` | `string` | App needs to transmit the value above, see how to create it below. |
| `privateKey` | `string` | App needs to transmit into the app system for decode the data. The app side will provide to the PayME system. |
| `configColor` | `string[]` | configColor : The color parameter can change color of PayME wallet transactions. The data type is string with the format #rrggbb. If 2 colors are transmitted, the PayME interface will gradient according to the 2 input colors. |

configColor : The color parameter can change color of PayME wallet transactions. The data type is string with the format #rrggbb. If 2 colors are transmitted, the PayME interface will gradient according to the 2 input colors.

[![img](https://github.com/PayME-Tech/PayME-SDK-Android-Example/raw/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)](https://github.com/PayME-Tech/PayME-SDK-Android-Example/blob/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)


How to create **connectToken**:

connectToken is needed to transmit the api to PayME, and it will create from backend system of the integrated app. The structure is following below:

-  ***Nodejs Example***
```javascript
import crypto from 'crypto'

const data = {
  timestamp: "2021-01-20T06:53:07.621Z",
  userId : "ABC",
  phone : "0909998877"
}

const algorithm = `aes-256-cbc`
const ivbyte = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
const iv = Buffer.from(ivbyte)

const cipher = crypto.createCipheriv(algorithm, secretKey, iv)

const encrypted = cipher.update(JSON.stringify(data), 'utf8', 'base64')

const connectToken = encrypted + cipher.final('base64')
```

-  ***React native Example***
```javascript
import forge from 'node-forge'

function encryptAES(text, secretKey) {
  const ivbyte = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  const cipher = forge.cipher.createCipher('AES-CBC', secretKey)
  cipher.start({ iv: ivbyte })
  cipher.update(forge.util.createBuffer(text, 'utf8'))
  cipher.finish()
  return forge.util.encode64(cipher.output.getBytes())
}

const data = {
  timestamp: "2021-01-20T06:53:07.621Z",
  userId : "abc",
  phone : "0123456789"
};

const connectToken = encryptAES(JSON.stringify(data), appSecretkey)
```

Create connectToken including KYC information (For partners has independent KYC system)

```javascript
const data = {
  timestamp: "2021-01-20T06:53:07.621Z",
  userId : "abc",
  phone : "0123456789",
  kycInfo: {
    fullname: "Nguyễn Văn A",
    gender: "MALE",
    birthday: "1995-01-20T06:53:07.621Z",
    address: "15 Nguyễn cơ thạch",
    identifyType: "CMND",
    identifyNumber: "142744332",
    issuedAt: "2013-01-20T06:53:07.621Z",
    placeOfIssue: "Hồ Chí Minh",
    video: "https://file-examples-com.github.io/uploads/2017/04/file_example_MP4_480_1_5MG.mp4",
    face: "https://cdn.pixabay.com/photo/2015/04/23/22/00/tree-736885__480.jpg",
    image: {
      front: "https://cdn.pixabay.com/photo/2015/04/23/22/00/tree-736885__480.jpg",
      back: "https://cdn.pixabay.com/photo/2015/04/23/22/00/tree-736885__480.jpg"
    }
  }
};

const connectToken = encryptAES(JSON.stringify(data), appSecretkey)
```

| **Parameters** | **Required** | **Explanation** |
| :----------- | :---------- | :------------------------------------------------- |
| **timestamp** | Yes | Time of creating connectToken in the format iSO 8601. Used to determine the timeout of connectToken. Example 2021-01-20T06:53:07,621Z |
| ***userId*** | Yes | Is a unique fixed value corresponding to each customer account in the service, usually this value is provided by the integrated system server for the PayME SDK |
| ***phone*** | No | Phone number of the system integration |

In which ***AES*** is the encryption function according to the AES algorithm. Depending on the language on the server, the system side uses the corresponding library. See more: https://en.wikipedia.org/wiki/Advanced_Encryption_Standard

KycInfo Parameters

| **Parameters** | **Required** | **Explanation** |
| :----------- | :---------- | :------------------------------------------------- |
| fullname | Yes | Full name |
| gender | Yes | Gender ( MALE/FEMALE) |
| address | Yes | Address |
| identifyType | Yes | Type of document (ID/CCCD) |
| identifyNumber | Yes | Number of papers |
| issuedAt | Yes | Registration date |
| placeOfIssue | Yes | Place of issue |
| video | No | Video link |
| face | No | Link to face photo |
| front | No | Link to a photo of the front of the document |
| back | No | Link to photo of the back of the document |

### PayME SDK Error Code

| **Constant** | **Error Code** | **Explanation** |
| :----------- | :---------- | :------------------------------------------------- ---------- |
| EXPIRED | 401 | ***token*** expired |
| NETWORK | -1 | Network connection problem |
| SYSTEM | -2 | System Error |
| LIMIT | -3 | Error of insufficient balance to make a transaction |
| ACCOUNT_NOT_ACTIVATED | -4 | Account not activated error |
| ACCOUNT_NOT_KYC | -5 | Unknown account error |
| PAYMENT_ERROR | -6 | Payment failed |
| ERROR_KEY_ENCODE | -7 | Data encryption/decryption error |
| USER_CANCELLED | -8 | User cancels |
| ACCOUNT_NOT_LOGIN | -9 | Error not logged in account |
| BALANCE_ERROR | -10 | Error when paying with PayME wallet but the balance in the wallet is not enough |
PAYMENT_PENDING | -11 | Payment Pending |
| ACCOUNT_ERROR   | -12           | Account Locked |

### Functions of PayME SDK
#### login()

There are 2 cases

- Used to login for the first time after create PayME.
- Used when the accessToken expires. When calling the SDK function that returns the error code ERROR_CODE.EXPIRED, the app needs to re-call login to get the accessToken for other functions.

After calling login() successfully, then call other functions of the SDK (openWallet, pay, ...)

```javascript
payME.login(
onSuccess: (response: any) => {
if (response.accountStatus === AccountStatus.NOT_ACTIVATED) {
// Account not activated
// call fun openWallet() to activate the account
}
if (response.accountStatus === AccountStatus.NOT_KYC) {
// Anonymous account
// call fun openKYC() to identify the account
}
if (response.accountStatus === AccountStatus.KYC_REVIEW) {
// Account submitted identifier, pending approval
}
if (response.accountStatus === AccountStatus.KYC_REJECTED) {
// Identify request denied
// call fun openKYC() to identify account
}
if (response.accountStatus === AccountStatus.KYC_APPROVED) {
// Identified account
}
},
onError: (error: any) => void,
);
```

#### getAccountInfo()

App can use this attribute after create the SDK to know the link status to PayME wallet.

```javascript
payME.openKYC(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### openWallet() - Open UI for PayME general function 

This function is called when from the built-in app want to call a PayME function by transmit the Action parameter as above.

```javascript
payME.openWallet(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### Parameter

| **Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes | Used to catch callback when making a successful transaction from PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes | Used to catch callback when an error occurs during calling PayME SDK |

#### openHistory() - Open UI for transaction history function

```javascript
payME.openHistory(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### Parameter

| **Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes | Used to catch callback when successful operation from PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes | Used to catch callback when an error occurs during calling PayME SDK |


#### deposit() - Deposit

```javascript
payME.deposit(
      amount: Number,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

This function has the same meaning as calling openWallet with the Action.Deposit action.

#### withdraw() - Withdrawal

```javascript
payME.withdraw(
      amount: Number,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

This function has the same meaning as calling openWallet with Action.Withdraw action.

#### transfer() - Transfer money

```javascript
payME.transfer(
      amount: Number,
      description: String,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

**Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| [amount](https://www.notion.so/amount-34eb8b97a9d04453867a7e4d87482980) | Yes| Used in case the action is Deposit/Withdraw, then transmit the amount |
| description | Yes | Money transfer content |
| closeWhenDone | Yes | true: Close SDK when transaction completed |
| onSuccess | Yes | Used to catch callback when making a successful transaction from PayME SDK |
| onError | Yes | Used to catch callback when an error occurs during calling PayME SDK |

#### scanQR() - Open QR code scanning for payment

```javascript
payME.scanQR(
      payCode: String,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

**Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| payCode | Yes | [Payment Method List](#list-method-payment) |

qr format:
```javascript
const qrString = "{$type}|${storeId}|${action}|${amount}|${note}|${orderId}|${userName}"
```

Example:
```javascript
const qrString = "OPENEWALLET|54938607|PAYMENT|20000|Chuyentien|2445562323|taikhoan"
```

- action: transaction type ( 'PAYMENT' => payment)
- amount: payment amount
- note: Description of the transaction from the partner
- orderId: transaction code of partner, which needs to be unique on each transaction. Maximum 22 characters.
- storeId: ID of the payment gateway that made the payment transaction
- userName: Account name
- type: OPENEWALLET

#### payQRCode() - QR code payment
```javascript
payME.payQRCode(
      qr: String,
      payCode: String,
      isShowResultUI: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void
);
```

**Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| qr | Yes | QR code for payment (QR format like scanQR function) |
| payCode | Yes | [Payment Method List](#list-method-payment) |
| isShowResultUI | Yes | Option to UI display transaction results |

#### getSupportedServices()

App can use this property after create SDK for a list of services that PayME is providing

```javascript
payME.getSupportedServices(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### openService()

This function is called when the built-in app want to call a service that PayME also provides by transmit the service parameter as above.

```javascript
payME.openService(
      service: Object,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### pay() - Payment

This function is used when the app needs to pay an amount from the activated PayME wallet.

```javascript
payME.pay(
      amount: Number,
      note: String,
      orderId: String,
      storeId: Number?,
      userName: String?,
      extractData: String,
      true,
      payCode: String,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

**Parameters** | **Required** | **Explanation** |
| :------------------------------------------------- | :---------- | :------------------------------------------------- |
| amount | Yes | The payment amount in app is transmit to the SDK. |
| note | No | Description of the transaction from partner. |
| orderId | Yes | Partner transaction code, which needs to be unique on each transaction. Maximum 22 characters. |
| storeId | No | The ID of store that performed the payment transaction. |
| userName | No | Account name. |
| extractData | No | Additional Information (extraData) is a string defined content, containing additional information about a transaction that parrner wants to receive when completing a transaction with PAYME. If the Merchant does not need IPN to add his custom data, he can ignore it. |
| isShowResultUI | Yes | Option UI display the payment result |
| payCode | Yes | [Payment Method List](#list-method-payment) |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes | Used to catch callback when making a successful transaction from PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes | Used to catch callback when an error occurs during calling PayME SDK |

In case the built-in app needs to get the balance to display itself on the UI on the app, you can use the function, this function does not display the UI of the PayME SDK.
- When paying with PayME wallet, it required the activated account, identifier and balance in the wallet must be greater than the payment amount
- Account information is obtained through the getAccountInfo() function
- Balance information is obtained through the getWalletInfo function ()

#### getWalletInfo() - Get wallet information

```javascript
 payME.getWalletInfo(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

- In case of error, the function will return a message at the onError function, then the app can display the balance as 0.
- In the successful case, the SDK returns the following information:

```json
{
  "Wallet": {
    "balance": 111,
    "detail": {
      "cash": 1,
      "lockCash": 2
    }
  }
}
```

*balance*: The built-in app can use the value in the balance key to display, other fields are currently unused.

*detail.cash*: Money can be used.

*detail.lockCash*: Money is locked.


#### close() - Close UI

This function is used for the integrated app to close the SDK's UI during payment or openWallet

```javascript
payME.close();
```

### setLanguage()
Change the language of the sdk
```javascript
payME.setLanguage(
language: LANGUAGES
);
```
Languages ​​supported: VIETNAMESE, ENGLISH

Example :
```javascript
payME.setLanguage(LANGUAGES.VIETNAMESE);
```
## List of payment methods
| **payCode** | **Payment Method** |
| :-----------| :------------|
| PAYME | PayME wallet payment |
| ATM | Domestic ATM card payment |
| MANUAL_BANK | Bank transfer payment |
| CREDIT | Credit card payments |

## Notes

#### Working with use_framework!

- react-native-permission: https://github.com/zoontek/react-native-permissions#workaround-for-use_frameworks-issues
- Google Map iOS Util: https://github.com/googlemaps/google-maps-ios-utils/blob/b721e95a500d0c9a4fd93738e83fc86c2a57ac89/Swift.md
## License

Copyright 2020 @ [PayME](payme.vn)
