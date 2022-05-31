[![en](https://img.shields.io/badge/lang-en-yellow.svg)](https://github.com/PayME-Tech/PayME-SDK-RN-Readme/blob/main/README_EN.md)

PayME SDK là bộ thư viện để các app có thể tương tác với PayME Platform. PayME SDK bao gồm các chức năng chính như sau:

- Hệ thống đăng ký, đăng nhập, eKYC thông qua tài khoản ví PayME
- Chức năng nạp rút chuyển tiền  từ ví PayME.
- Tích hợp các dịch vụ của PayME Platform.

**Một số thuật ngữ**

|      | Name    | Giải thích                                                   |
| :--- | :------ | ------------------------------------------------------------ |
| 1    | app     | Là app mobile iOS/Android hoặc web sẽ tích hợp SDK vào để thực hiện chức năng thanh toán ví PayME. |
| 2    | SDK     | Là bộ công cụ hỗ trợ tích hợp ví PayME vào hệ thống app.     |
| 3    | backend | Là hệ thống tích hợp hỗ trợ cho app, server hoặc api hỗ trợ  |
| 4    | AES     | Hàm mã hóa dữ liệu AES. [Tham khảo](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) |
| 5    | RSA     | Thuật toán mã hóa dữ liệu RSA.                               |
| 6    | IBN     | Instant Payment Notification , dùng để thông báo giữa hệ thống backend của app và backend của PayME |

##  Cài đặt

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

Nếu App Tích hợp có sử dụng payCode = VNPAY thì cấu hình thêm urlscheme vào Activity nhận kết quả thanh toán để khi thanh toán xong bên ví VNPAY có thể tự động quay lại app tích hợp

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

Cấp quyền truy cập danh bạ khi dùng chức năng nạp điện thoại và chuyển tiền

```xml
  ...
    <uses-permission android:name="android.permission.READ_CONTACTS" />
  ...
```

- **MainActivity.java**

⚠️⚠️⚠️ Đã xoá từ version 0.9.16 trở đi. 

Vui lòng xoá 2 function này nếu đã cài đặt, nếu chưa thì xin bỏ qua bước này.

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

Update file Info.plist của app với những key như sau (giá trị của string có thể thay đổi, đây là các message hiển thị khi yêu cầu người dùng cấp quyền tương ứng):

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

Nếu không sử dụng tính năng danh bạ thì thêm vào cuối podfile

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

Nếu như project iOS chưa cấu hình cho Swift thì có thể thêm file .swift rỗng tùy ý vào project từ XCode để tự động enable Bridging header.

1. Mở XCode Project
2. File -> New File -> Chọn .swift file -> Next -> Create -> Chọn Create Bridging Header
[![img](https://docs-assets.developer.apple.com/published/f86b29eb4b/757d8a73-4e94-4b68-8227-ae32e2ca1304.png)](https://github.com/PayME-Tech/PayME-SDK-Android-Example/blob/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)


## Usage

Hệ thống PayME sẽ cung cấp cho app tích hợp các thông tin sau:

- **PublicKey** : Dùng để mã hóa dữ liệu, app tích hợp cần truyền cho SDK để mã hóa.
- **AppToken** : AppToken cấp riêng định danh cho mỗi app, cần truyền cho SDK để mã hóa
- **SecretKey** : Dùng đã mã hóa và xác thực dữ liệu ở hệ thống backend cho app tích hợp.

Bên App sẽ cung cấp cho hệ thống PayME các thông tin sau:

- **AppPublicKey** : Sẽ gửi qua hệ thống backend của PayME dùng để mã hóa.
- **AppPrivateKey**: Sẽ truyền vào PayME SDK để thực hiện việc giải mã.

Chuẩn mã hóa: RSA-512bit.

### Khởi tạo thư viện

Trước khi sử dụng PayME SDK cần gọi phương thức khởi tạo một lần duy nhất để khởi tạo SDK.

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
| `ENV.SANDBOX`    | `enum` | Môi trường sandbox.    |
| `ENV.STAGING` | `enum` | Môi trường staging. |
| `ENV.PRODUCTION` | `enum` | Môi trường production. |

#### Parameters

| Property       | Type       | Description                                                  |
| -------------- | ---------- | ------------------------------------------------------------ |
| `appToken`     | `string`   | AppId cấp riêng định danh cho mỗi app, cần truyền cho SDK để mã hóa. |
| `publicKey`    | `string`   | Dùng để mã hóa dữ liệu, app tích hợp cần truyền cho SDK để mã hóa. Do hệ thống PayME cung cấp cho app tích hợp. |
| `connectToken` | `string`   | app cần truyền giá trị được cung cấp ở trên, xem cách tạo bên dưới. |
| `privateKey`   | `string`   | app cần truyền vào để giải mã dữ liệu. Bên app sẽ cung cấp cho hệ thống PayME.                        |
| `configColor`  | `string[]` | configColor : là tham số màu để có thể thay đổi màu sắc giao dịch ví PayME, kiểu dữ liệu là chuỗi với định dạng #rrggbb. Nếu như truyền 2 màu thì giao diện PayME sẽ gradient theo 2 màu truyền vào. |

configColor : là tham số màu để có thể thay đổi màu sắc giao dịch ví PayME, kiểu dữ liệu là chuỗi với định dạng #rrggbb. Nếu như truyền 2 màu thì giao diện PayME sẽ gradient theo 2 màu truyền vào.

[![img](https://github.com/PayME-Tech/PayME-SDK-Android-Example/raw/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)](https://github.com/PayME-Tech/PayME-SDK-Android-Example/blob/main/fe478f50-e3de-4c58-bd6d-9f77d46ce230.png?raw=true)


Cách tạo **connectToken**:

connectToken cần để truyền gọi api từ tới PayME và sẽ được tạo từ hệ thống backend của app tích hợp. Cấu trúc như sau:

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

Tạo connectToken bao gồm thông tin KYC (Dành cho các đối tác có hệ thống KYC riêng)

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

| **Tham số**   | **Bắt buộc** | **Giải thích**                                               |
| :------------ | :----------- | :----------------------------------------------------------- |
| **timestamp** | Yes          | Thời gian tạo ra connectToken theo định dạng iSO 8601 , Dùng để xác định thời gian timeout của connectToken. Ví dụ 2021-01-20T06:53:07.621Z |
| ***userId***  | Yes          | là giá trị cố định duy nhất tương ứng với mỗi tài khoản khách hàng ở dịch vụ, thường giá trị này do server hệ thống được tích hợp cấp cho PayME SDK |
| ***phone***   | No           | Số điện thoại của hệ thống tích hợp |

Trong đó ***AES*** là hàm mã hóa theo thuật toán AES. Tùy vào ngôn ngữ ở server mà bên hệ thống dùng thư viện tương ứng. Xem thêm tại đây https://en.wikipedia.org/wiki/Advanced_Encryption_Standard

Tham số KycInfo

| **Tham số**   | **Bắt buộc** | **Giải thích**                                               |
| :------------ | :----------- | :----------------------------------------------------------- |
| fullname | Yes          | Họ tên |
| gender  | Yes          | Giới tính ( MALE/FEMALE) |
| address   | Yes           | Địa chỉ |
| identifyType   | Yes           | Loại giấy tờ (CMND/CCCD) |
| identifyNumber   | Yes           | Số giấy tờ |
| issuedAt   | Yes           | Ngày đăng ký |
| placeOfIssue   | Yes           | Nơi cấp |
| video   | No           | đường dẫn tới video |
| face   | No           | đường dẫn tới ảnh chụp khuôn mặt |
| front   | No           | đường dẫn tới ảnh mặt trước giấy tờ |
| back   | No           | đường dẫn tới ảnh mặt sau giấy tờ |

### Mã lỗi của PayME SDK

| **Hằng số**   | **Mã lỗi** | **Giải thích**                                               |
| :------------ | :----------- | :----------------------------------------------------------- |
| <code>EXPIRED</code> | <code>401</code>          | ***token*** hết hạn sử dụng |
| <code>NETWORK</code>  | <code>-1</code>          | Kết nối mạng bị sự cố |
| <code>SYSTEM</code>   | <code>-2</code>           | Lỗi hệ thống |
| <code>LIMIT</code>   | <code>-3</code>           | Lỗi số dư không đủ để thực hiện giao dịch |
| <code>ACCOUNT_NOT_ACTIVATED</code>   | <code>-4</code>           | Lỗi tài khoản chưa kích hoạt |
| <code>ACCOUNT_NOT_KYC</code>   | <code>-5</code>           | Lỗi tài khoản chưa định danh |
| <code>PAYMENT_ERROR</code>   | <code>-6</code>          | Thanh toán thất bại |
| <code>ERROR_KEY_ENCODE</code>   | <code>-7</code>           | Lỗi mã hóa/giải mã dữ liệu |
| <code>USER_CANCELLED</code>   | <code>-8</code>          | Người dùng thao tác hủy |
| <code>ACCOUNT_NOT_LOGIN</code>   | <code>-9</code>           | Lỗi chưa đăng nhập tài khoản |
| <code>BALANCE_ERROR</code>   | <code>-10</code>           | Lỗi khi thanh toán bằng ví PayME mà số dư trong ví không đủ |
<code>PAYMENT_PENDING</code>   | <code>-11</code>          | Thanh toán đang chờ xử lý |
| <code>ACCOUNT_ERROR</code>   | <code>-12</code>           | Lỗi tài khoản bị khóa |

### Các chức năng của PayME SDK
#### login()

Có 2 trường hợp

- Dùng để login lần đầu tiên ngay sau khi khởi tạo PayME.
- Dùng khi accessToken hết hạn, khi gọi hàm của SDK mà trả về mã lỗi ERROR_CODE.EXPIRED, lúc này app cần gọi login lại để lấy accessToken dùng cho các chức năng khác.

Sau khi gọi login() thành công rồi thì mới gọi các chức năng khác của SDK ( openWallet, pay, ... )

```javascript
payME.login(
  onSuccess: (response: any) => {
    if (response.accountStatus === AccountStatus.NOT_ACTIVATED) {
      // Tài khoản chưa kích hoạt
      // gọi fun openWallet() để kích hoạt tài khoản
    }
    if (response.accountStatus === AccountStatus.NOT_KYC) {
      // Tài khoản chưa định danh
      // gọi fun openKYC() để định danh tài khoản
    }
    if (response.accountStatus === AccountStatus.KYC_REVIEW) {
      // Tài khoản đã gửi thông tin định danh, đang chờ duyệt
    }
    if (response.accountStatus === AccountStatus.KYC_REJECTED) {
      // Yêu cầu định danh bị từ chối
			// gọi fun openKYC() để định danh tài khoản
    }
    if (response.accountStatus === AccountStatus.KYC_APPROVED) {
      // Tài khoản đã định danh
    }
  },
  onError: (error: any) => void,
);
```

#### getAccountInfo()

App có thể dùng thuộc tính này sau khi khởi tạo SDK để biết được trạng thái liên kết tới ví PayME.

```javascript
payME.getAccountInfo(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### openKYC()

Hàm này được gọi khi từ app tích hợp khi muốn mở modal định danh tài khoản ( yêu cầu tài khoản phải chưa định danh).

```javascript
payME.openKYC(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```


#### openWallet() - Mở UI chức năng PayME tổng hợp

Hàm này được gọi khi từ app tích hợp khi muốn gọi 1 chức năng PayME bằng cách truyền vào tham số Action như trên.

```javascript
payME.openWallet(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### Tham số

| **Tham số**                                                  | **Bắt buộc** | **Giải thích**                                               |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes          | Dùng để bắt callback khi thực hiện giao dịch thành công từ PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes          | Dùng để bắt callback khi có lỗi xảy ra trong quá trình gọi PayME SDK |

#### openHistory() - Mở UI chức năng lịch sử giao dịch

```javascript
payME.openHistory(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### Tham số

| **Tham số**                                                  | **Bắt buộc** | **Giải thích**                                               |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes          | Dùng để bắt callback khi thao tác thành công từ PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes          | Dùng để bắt callback khi có lỗi xảy ra trong quá trình gọi PayME SDK |


#### deposit() - Nạp tiền

```javascript
payME.deposit(
      amount: Number,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

Hàm này có ý nghĩa giống như khi gọi openWallet với action Action.Deposit.

#### withdraw() - Rút tiền

```javascript
payME.withdraw(
      amount: Number,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

Hàm này có ý nghĩa giống như khi gọi openWallet với action Action.Withdraw.

#### transfer() - Chuyển tiền

```javascript
payME.transfer(
      amount: Number,
      description: String,
      closeWhenDone: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

| **Tham số** | **Bắt buộc** | **Giải thích** |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| [amount](https://www.notion.so/amount-34eb8b97a9d04453867a7e4d87482980) | Yes| Dùng trong trường hợp action là Deposit/Withdraw thì truyền vào số tiền |
| description | Yes | Nội dung chuyển tiền |
| closeWhenDone | Yes | true: Đóng SDK khi hoàn tất giao dịch |
| onSuccess | Yes | Dùng để bắt callback khi thực hiện giao dịch thành công từ PayME SDK |
| onError | Yes | Dùng để bắt callback khi có lỗi xảy ra trong quá trình gọi PayME SDK |

#### scanQR() - Mở chức năng quét mã QR để thanh toán

```javascript
payME.scanQR(
      payCode: String,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

| **Tham số** | **Bắt buộc** | **Giải thích** |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| payCode | Yes | [Danh sách phương thức thanh toán](#danh-sách-phương-thức-thanh-toán) |

Định dạng qr :
```javascript
 const qrString = "{$type}|${storeId}|${action}|${amount}|${note}|${orderId}|${userName}"
```

Ví dụ  :
```javascript
const qrString = "OPENEWALLET|54938607|PAYMENT|20000|Chuyentien|2445562323|taikhoan"
```

- action: loại giao dịch ( 'PAYMENT' => thanh toán)
- amount: số tiền thanh toán
- note: Mô tả giao dịch từ phía đối tác
- orderId: mã giao dịch của đối tác, cần duy nhất trên mỗi giao dịch. Tối đa 22 kí tự.
- storeId: ID của store phía công thanh toán thực hiên giao dịch thanh toán
- userName: Tên tài khoản
- type: OPENEWALLET

#### payQRCode() - thanh toán mã QR code
```javascript
payME.payQRCode(
      qr: String,
      payCode: String,
      isShowResultUI: Boolean,
      onSuccess: (response: any) => void,
      onError: (error: any) => void
);
```


| **Tham số** | **Bắt buộc** | **Giải thích** |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| qr | Yes | Mã QR để thanh toán  (Định dạng QR như hàm scanQR) |
| payCode | Yes | [Danh sách phương thức thanh toán](#danh-sách-phương-thức-thanh-toán) |
| isShowResultUI | Yes | Option hiển thị UI kết quả giao dịch |

#### getSupportedServices()

App có thể dùng thược tính này sau khi khởi tạo SDK để biết danh sách các dịch vụ mà PayME đang cung cấp

```javascript
payME.getSupportedServices(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### openService()

Hàm này được gọi khi từ app tích hợp khi muốn gọi 1dịch vụ mà PayME cũng cấp bằng cách truyền vào tham số service như trên

```javascript
payME.openService(
      service: Object,
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

#### pay() - Thanh toán

Hàm này được dùng khi app cần thanh toán 1 khoản tiền từ ví PayME đã được kích hoạt.

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

| **Tham số**                                                  | **Bắt buộc** | **Giải thích**                                               |
| :----------------------------------------------------------- | :----------- | :----------------------------------------------------------- |
| amount | Yes           | Số tiền cần thanh toán bên app truyền qua cho SDK. |
| note | No           | Mô tả giao dịch từ phía đối tác. |
| orderId | Yes           | Mã giao dịch của đối tác, cần duy nhất trên mỗi giao dịch. Tối đa 22 kí tự. |
| storeId | No           | ID của store phía công thanh toán thực hiên giao dịch thanh toán. |
| userName | No           | Tên tài khoản. |
| extractData | No           | Thông tin bổ sung (extraData) là một nội dung được định nghĩa theo dạng chuỗi, chứa thông tin bổ sung của một giao dịch mà đối tác muốn nhận về khi hoàn tất một giao dịch với PAYME. Nếu Merchant ko cần IPN thêm data custom của mình có thể bỏ qua. |
| isShowResultUI | Yes           | Option hiển thị UI kết quả thanh toán. |
| payCode | Yes           | [Danh sách phương thức thanh toán](#danh-sách-phương-thức-thanh-toán) |
| [onSuccess](https://www.notion.so/onSuccess-6e24a547a1ad46499c9d6413b5c02e81) | Yes          | Dùng để bắt callback khi thực hiện giao dịch thành công từ PayME SDK |
| [onError](https://www.notion.so/onError-25f94cb5a141484b8a70b9f1a2d7f33f) | Yes          | Dùng để bắt callback khi có lỗi xảy ra trong quá trình gọi PayME SDK |

Trong trường hợp app tích hợp cần lấy số dư để tự hiển thị lên UI trên app thì có thể dùng hàm, hàm này không hiển thị UI của PayME SDK.
- Khi thanh toán bằng ví PayME thì yêu cầu tài khoản đã kích hoạt, định danh và số dư trong ví phải lớn hơn số tiền thanh toán
- Thông tin tài khoản lấy qua hàm <code>getAccountInfo()</code>
- Thông tin số dư lấy qua hàm <code>getWalletInfo()</code>

#### getWalletInfo() - Lấy các thông tin của ví

```javascript
 payME.getWalletInfo(
      onSuccess: (response: any) => void,
      onError: (error: any) => void,
);
```

- Trong trường hợp lỗi thì hàm sẽ trả về message mỗi tại hàm onError , khi đó app có thể hiển thị balance là 0.
- Trong trường hợp thành công SDK trả về thông tin như sau:

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

*balance*: App tích hợp có thể sử dụng giá trị trong key balance để hiển thị, các field khác hiện tại chưa dùng.

*detail.cash*: Tiền có thể dùng.

*detail.lockCash*: Tiền bị lock.


#### close() - Đóng UI

Hàm này được dùng để app tích hợp đóng lại UI của SDK khi đang payment hoặc openWallet

```javascript
 payME.close();
```

### setLanguage()
Chuyển đổi ngôn ngữ của sdk
```javascript
 payME.setLanguage(
   language: LANGUAGES
 );
```
Ngôn ngữ hỗ trợ: VIETNAMESE, ENGLISH

Ví dụ  :
```javascript
payME.setLanguage(LANGUAGES.VIETNAMESE);
```

## Danh sách phương thức thanh toán
| **payCode** | **Phương thức thanh toán** |
| :------------| :-------------|
| PAYME  | Thanh toán ví PayME |
| ATM  | Thanh toán thẻ ATM Nội địa |
| MANUAL_BANK  | Thanh toán chuyển khoản ngân hàng |
| CREDIT  | Thanh toán thẻ tín dụng |

## Ghi chú

#### Làm việc với use_framework!

- react-native-permission: https://github.com/zoontek/react-native-permissions#workaround-for-use_frameworks-issues
- Google Map iOS Util: https://github.com/googlemaps/google-maps-ios-utils/blob/b721e95a500d0c9a4fd93738e83fc86c2a57ac89/Swift.md
## License

Copyright 2020 @ [PayME](payme.vn)

