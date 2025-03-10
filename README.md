# @react-native-seoul/naver-login

[![npm version](http://img.shields.io/npm/v/@react-native-seoul/naver-login.svg?style=flat-square)](https://npmjs.org/package/@react-native-seoul/naver-login)
[![downloads](http://img.shields.io/npm/dm/@react-native-seoul/naver-login.svg?style=flat-square)](https://npmjs.org/package/@react-native-seoul/naver-login)
[![license](http://img.shields.io/npm/l/@react-native-seoul/naver-login.svg?style=flat-square)](https://npmjs.org/package/@react-native-seoul/naver-login)

React Native 네이버 로그인 라이브러리 입니다.

### Supported platforms

![Android](https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-000000?style=for-the-badge&logo=ios&logoColor=white)

### Supported typing

- TypeScript
- Flow

<img src="https://user-images.githubusercontent.com/33388801/196834333-69841305-ebd2-4b59-b02b-b079aafd7523.gif" width=400 />

## Installation

❗️ `2.x` 버전은 [2.x branch](https://github.com/react-native-seoul/react-native-naver-login/tree/2.x) 의 설치 가이드와 사용법을 따라주세요.


```shell
# npm
npm install @react-native-seoul/naver-login --save

# yarn
yarn add @react-native-seoul/naver-login
```

### RN version >= `0.60` 

- Auto Linking 이 적용됩니다.
- iOS의 경우 추가적으로 Cocoapods 설치가 필요합니다.

```shell
cd ios && pod install
```

### RN version < `0.60`

- `0.60` 미만의 React Native를 사용중이시라면 [Manual Linking Guide](./README-manual-linking.md)를 참고해주세요.


### 추가 작업 - iOS ❗️Important

#### 1. Launch Service Queries Schemes 추가

로그인 시에 네이버 앱을 실행시키기 위해 [Launch Services Queries Schemes](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html) 를 등록해주어야 합니다.

`Info.plist` 파일안에 다음과 같은 항목을 추가합니다. 

- `naversearchapp`
- `naversearchthirdlogin`

이미 `LSApplicationQueriesSchemes` 가 항목으로 추가되어 있다면, `<array>` 안에 두 가지만 더 추가해주세요.

```
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>naversearchapp</string>
  <string>naversearchthirdlogin</string>
</array>
```

![image](https://user-images.githubusercontent.com/33388801/196834997-40ca2368-50e6-45cb-9b68-2cf5307fd792.png)

#### 2. custom URL scheme 추가

네이버 로그인이 완료된 뒤 다시 우리의 앱으로 돌아오기 위해 `URL Scheme`를 `Info.plist` 에 정의해주어야 합니다.

아래 코드들에서 `{{ CUSTOM URL SCHEME }}`는 커스텀하게 정의할 우리 앱에 사용될 URL scheme라고 생각하시면 됩니다.

주의할 점은 다음과 같습니다.

- 네이버 개발자 콘솔에 기입한 `URL Scheme`와 동일해야 합니다.
- `login` 함수 호출시에 `serviceUrlScheme` 로 동일하게 전달해주어야 합니다.
- TODO 설명 및 이미지 추가

대략 다음과 같이 `Info.plist`에 입력되게 됩니다.
```
<key>CFBundleURLTypes</key>
<array>
	<dict>
		<key>CFBundleTypeRole</key>
		<string>Editor</string>
		<key>CFBundleURLName</key>
		<string>naver</string>
		<key>CFBundleURLSchemes</key>
		<array>
			<string>{{ CUSTOM URL SCHEME }}</string>
		</array>
	</dict>
	...
</array>
```

![image](https://user-images.githubusercontent.com/33388801/196835050-3d887f3c-1d07-4be3-a2cf-27ed18a48691.png)

#### 3. `AppDelegate`의 `application:openURL:options` 에서 URL 핸들링 로직 추가

네이버 로그인이 성공한 후 우리앱으로 다시 돌아와 URL을 처리하기 위해 필요한 과정입니다.

```objc
#import <NaverThirdPartyLogin/NaverThirdPartyLoginConnection.h>
...

// 다른 URL 핸들링 로직이 없는경우
- (BOOL)application:(UIApplication *)application
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
   return [[NaverThirdPartyLoginConnection getSharedInstance] application:app openURL:url options:options];
}

// 다른 URL 핸들링 로직이 같이 있는 경우
- (BOOL)application:(UIApplication *)application
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
  // naver
  if ([url.scheme isEqualToString:@"{{ CUSTOM URL SCHEME }}"]) {
    return [[NaverThirdPartyLoginConnection getSharedInstance] application:application openURL:url options:options];
  }
  
  // kakao
  if([RNKakaoLogins isKakaoTalkLoginUrl:url]) {
    return [RNKakaoLogins handleOpenUrl: url];
  }
  
  ...
}
```


### 추가 작업 - Android ❗️Important

#### 1. Proguard

만약 Release build에서 R8 컴파일러를 이용해 code obfuscating을 하신다면, app/build.gradle 설정에 `minifyEnabled`이 `true`로 설정이 되어있을 것입니다.

그 경우 다음과 같은 Proguard 규칙이 필요합니다.

만약 그렇지 않다면 별도의 설정이 필요하지 않습니다.

```text
-keep public class com.nhn.android.naverlogin.** {
       public protected *;
}
```

## API

| Func        |        Param        |            Return             | Description                        |
|:------------|:-------------------:|:-----------------------------:|:-----------------------------------|
| login       | `NaverLoginRequest` | `Promise<NaverLoginResponse>` | 로그인, 반환되는 `Promise`는 항상 resolve된다. |
| getProfile  |      `String`       | `Promise<GetProfileResponse>` | 프로필 불러오기                           |
| logout      |                     |        `Promise<void>`        | 로그아웃                               |
| deleteToken |                     |        `Promise<void>`        | 네이버 앱 연동 삭제                        |
| deleteTokenWithInit |`NaverLoginRequest`|`Promise<void>`.         | (Android) deleteToken 이전에 SDK initialize 선행   |

### Type

**NaverLoginRequest**
```typescript
export interface NaverLoginRequest {
  consumerKey: string;
  consumerSecret: string;
  appName: string;
  disableNaverAppAuth?: boolean;
  /** Only for iOS */
  serviceUrlScheme?: string;
}
```

**NaverLoginResponse**
```typescript
export interface NaverLoginResponse {
  isSuccess: boolean;
  /** isSuccess가 true일 때 존재합니다. */
  successResponse?: {
    accessToken: string;
    refreshToken: string;
    expiresAtUnixSecondString: string;
    tokenType: string;
  };
  /** isSuccess가 false일 때 존재합니다. */
  failureResponse?: {
    message: string;
    isCancel: boolean;

    /** Android Only */
    lastErrorCodeFromNaverSDK?: string;
    /** Android Only */
    lastErrorDescriptionFromNaverSDK?: string;
  };
}
```

**GetProfileResponse**
```typescript
export interface GetProfileResponse {
  resultcode: string;
  message: string;
  response: {
    id: string;
    profile_image: string | null;
    email: string;
    name: string;
    birthday: string | null;
    age: string | null;
    birthyear: number | null;
    gender: string | null;
    mobile: string | null;
    mobile_e164: string | null;
    nickname: string | null;
  };
}
```


## Usage

- 자세한 예제는 [예제 프로젝트](./NaverLoginExample)를 참고해주세요

```typescript jsx
import React, {useState} from 'react';
import {SafeAreaView, Button, View, Text, ScrollView} from 'react-native';
import NaverLogin, {
  NaverLoginResponse,
  GetProfileResponse,
} from '@react-native-seoul/naver-login';

const consumerKey = '';
const consumerSecret = '';
const appName = 'Hello';
const serviceUrlScheme = 'navertest';

const App = () => {
  const [success, setSuccessResponse] =
    useState<NaverLoginResponse['successResponse']>();
  const [failure, setFailureResponse] =
    useState<NaverLoginResponse['failureResponse']>();
  const [getProfileRes, setGetProfileRes] = useState<GetProfileResponse>();

  const login = async () => {
    const {failureResponse, successResponse} = await NaverLogin.login({
      appName,
      consumerKey,
      consumerSecret,
      serviceUrlScheme,
    });
    setSuccessResponse(successResponse);
    setFailureResponse(failureResponse);
  };

  const logout = async () => {
    try {
      await NaverLogin.logout();
      setSuccessResponse(undefined);
      setFailureResponse(undefined);
      setGetProfileRes(undefined);
    } catch (e) {
      console.error(e);
    }
  };

  const getProfile = async () => {
    try {
      const profileResult = await NaverLogin.getProfile(success!.accessToken);
      setGetProfileRes(profileResult);
    } catch (e) {
      setGetProfileRes(undefined);
    }
  };

  const deleteToken = async () => {
    try {
      await NaverLogin.deleteToken();
      setSuccessResponse(undefined);
      setFailureResponse(undefined);
      setGetProfileRes(undefined);
    } catch (e) {
      console.error(e);
    }
  };

  return (
    <SafeAreaView
      style={{alignItems: 'center', justifyContent: 'center', flex: 1}}>
      <ScrollView
        style={{flex: 1}}
        contentContainerStyle={{flexGrow: 1, padding: 24}}>
        <Button title={'Login'} onPress={login} />
        <Gap />
        <Button title={'Logout'} onPress={logout} />
        <Gap />
        {success ? (
          <>
            <Button title="Get Profile" onPress={getProfile} />
            <Gap />
          </>
        ) : null}
        {success ? (
          <View>
            <Button title="Delete Token" onPress={deleteToken} />
            <Gap />
            <ResponseJsonText name={'Success'} json={success} />
          </View>
        ) : null}
        <Gap />
        {failure ? <ResponseJsonText name={'Failure'} json={failure} /> : null}
        <Gap />
        {getProfileRes ? (
          <ResponseJsonText name={'GetProfile'} json={getProfileRes} />
        ) : null}
      </ScrollView>
    </SafeAreaView>
  );
};
const Gap = () => <View style={{marginTop: 24}} />;
const ResponseJsonText = ({json = {}, name}: {json?: object; name: string}) => (
  <View
    style={{
      padding: 12,
      borderRadius: 16,
      borderWidth: 1,
      backgroundColor: '#242c3d',
    }}>
    <Text style={{fontSize: 20, fontWeight: 'bold', color: 'white'}}>
      {name}
    </Text>
    <Text style={{color: 'white', fontSize: 13, lineHeight: 20}}>
      {JSON.stringify(json, null, 4)}
    </Text>
  </View>
);

export default App;
```

## Troubleshooting
<a href='https://github.com/crossplatformkorea/react-native-naver-login/issues/163'>issue #163</a> 에서 언급한대로, RN에서 발급받은 Access Token을 받아 외부에서 이 토큰을 사용해 인증하려고하면 invalid token error가 발생하는 이슈가 있었습니다. 여기에 대한 대응으로 근본적인 해결책은 아니지만 deleteToken을 호출해 token을 무효화 한 이후에 login을 진행하면 정상작동하는 것을 확인했습니다.

다만 <a href='https://github.com/crossplatformkorea/react-native-naver-login/issues/156'>issue #156</a>에서도 언급했듯이, android에서 deleteToken을 호출하면 앱이 crash가 나는 현상이 있습니다. 이 문제는 NaverIdLoginSDK가 초기화되기 전에 deleteToken이 먼저 실행되면서 생기는 문제로, deleteToken 대신에 deleteTokenWithInit를 사용하여 해결할 수 있습니다.

```typescript

const login = async () => {
    const {failureResponse, successResponse} = await NaverLogin.login({
      appName,
      consumerKey,
      consumerSecret,
      serviceUrlScheme,
    });
    setSuccessResponse(successResponse);
    setFailureResponse(failureResponse);
  };

const loginAfterDeleteToken = async () => {
    NaverLogin.deleteTokenWithInit({
      appName,
      consumerKey,
      consumerSecret,
    });
    setTimeout(login, 200);
  };

```
deleteTokenWithInit은 Android only이며, ios에서 실행될 경우 deleteToken과 동일하게 동작합니다.
