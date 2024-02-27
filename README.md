# AuthGate

## Installation

### git URL

```
https://github.com/TinycellCorp/auth-gate.git?path=Packages/AuthGate#1.0.6

// dependencies
https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask
```

## Firebase

Firebase Unity Packages중에는 100M 이상인 파일들이 포함되어 있는데,

github의 경우 단일 파일 50M 이상은 경고, 100M 이상은 에러로 Push를 할 수 없다.

lfs사용을 회피할 것 이라면 tgz로 압축되어 있는 패키지를 사용해 설치하면 된다.

[Google APIs for Unity Archive](https://developers.google.com/unity/archive?hl=ko#firebase)에서 tgz 확장자 패키지를
다운받아서 `Add package from tarball` 메뉴로 패키지 추가.

기본적으로 아래 패키지들을 추가한다.

* [Unity용 외부 종속 항목 관리자](https://developers.google.com/unity/archive?hl=ko#tools)
* [Firebase 앱 (코어)](https://developers.google.com/unity/archive?hl=ko#firebase_app_core)

> **Note**  
> 패키지를 구현한 Firebase 버전  
> com.google.firebase.*-11.0.0

## TL;DR

```csharp
var result = await FirebaseApp.CheckAndFixDependenciesAsync();
if (result != DependencyStatus.Available) return;

var config = new FirebaseConfig();
// add support google signin
config.AddCredentialProvider(new GoogleProvider("webClientId"));
// add support apple signin
config.AddCredentialProvider(new AppleProvider());

var auth = FirebaseAuth.DefaultInstance;
IGate gate = new FirebaseGate(auth, config);

var user = UserInfo.Invalid;
try
{
    user = await GateManager.InitializeAsync(gate);
}
catch (InvalidCredentialException e)
{
    Debug.LogError(e.Reason.ToString());
}

if(user.IsValid())
{
    // success previous credentional signin
    return;
}

try
{
  // google signin
  user = await GateManager.SignInAsync(GoogleProvider.Id);
  
  // apple signin
  // user = await GateManager.SignInAsync(AppleProvider.Id);
}
catch(SignInFailedException e)
{
    Debug.LogError(e.Message);
}

if(user.IsValid())
{
    // success signin
}
```

## Auth flow

[//]: # (TODO: add diagram)

### GateManager.InitializeAsync

```csharp
var config = new FirebaseConfig();
config.AddCredentialProvider(new GoogleProvider("webClientId"));

var auth = FirebaseAuth.DefaultInstance;
IGate gate = new FirebaseGate(auth, config);

var user = await GateManager.InitializeAsync(gate);
```

FirebaseAuth 초기화 후 이전에 로그인 했었다면 FirebaseAuth.CurrentUser에 User인스턴스가 생성되어 있다.

> 해당 유저정보를 즉시 반환함

User가 생성되어 있더라도 해당 User에 연결된 인증상태가 해제되어 있을 수 있기 때문에  
해당 인증에 대한 유효성 검사를 시도한다.

> 유효성 검사에 실패하면 InvalidCredentialException

자격증명이 유효하지않으면 다시 SignIn을 진행하면 된다.

### GateManager.SignInAsync

```csharp
var user = await GateManager.SignInAsync(GoogleProvider.Id);
```

초기화 과정에서 유효한 User를 얻지 못하면 id provider로 추가인증을 거쳐 로그인한다. (SignIn)

> SignIn 중에 문제 발생 시 SignInFailedException

### GateManager.SignInAnonymousAsync

익명 유저 로그인.

## Plugins

### Sign in with Google

* [google-signin-unity (Google 공식 패키지)](https://github.com/googlesamples/google-signin-unity)

  구글 공식 패키지이지만 릴리즈가 2018이후로 없는 상태이기 때문에 그냥 적용하기에는 종속성이나 iOS빌드 버그 등  
  여러 이슈가 발생하는 경우가 있다.

* [google-signin-unity (CodeMasterYi)](https://github.com/CodeMasterYi/google-signin-unity)

  공식 패키지를 기반으로 발생하는 이슈를 다른 개발자(CodeMasterYi)가 수정한
  버전 [PR#205](https://github.com/googlesamples/google-signin-unity/pull/205)

* [google-signin-unity (TinycellCorp)](https://github.com/TinycellCorp/google-signin-unity)

  위 패키지에서 [PR#1](https://github.com/CodeMasterYi/google-signin-unity/pull/1)를 적용.  
  pods GoogleSignIn 7.0.0 릴리즈 이후 xcode로 플러그인 빌드 시 GIDAuthentication.h file not found가 발생하는 이슈를 수정한 버전.
   
  `https://github.com/TinycellCorp/google-signin-unity.git#1.0.4`

### Sign in with Apple

[apple-signin-unity](https://github.com/lupidan/apple-signin-unity)

## TroubleShooting

### Android Resolver: Gradle Resolve: Tool Extraction Failed

Scripting Backend: il2CPP

### Google Sign In Package

Minimum API Level 26 (Min Sdk)  
API Level 33 (Target Sdk)