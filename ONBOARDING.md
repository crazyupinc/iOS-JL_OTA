# iOS JL_OTA 온보딩 가이드

iOS용 杰理(Jieli) 블루투스 OTA(Over-The-Air) 업그레이드 SDK에 오신 것을 환영합니다!

## 📋 목차

1. [프로젝트 소개](#프로젝트-소개)
2. [시작하기 전에](#시작하기-전에)
3. [개발 환경 설정](#개발-환경-설정)
4. [빠른 시작](#빠른-시작)
5. [SDK 통합 방법](#sdk-통합-방법)
6. [주요 기능](#주요-기능)
7. [문제 해결](#문제-해결)
8. [추가 리소스](#추가-리소스)

---

## 프로젝트 소개

### 🎯 프로젝트 개요

iOS JL_OTA는 杰理(Jieli) 블루투스 장치의 펌웨어 무선 업그레이드(OTA)를 지원하는 iOS SDK입니다. 이 프로젝트를 통해 다음과 같은 장치의 펌웨어 업데이트를 구현할 수 있습니다:

- **데이터 전송 장치**: AC695X, AC608N, AC897, AD697N, AD698N, AC630N, AC632N
- **스마트워치**: AC695X, JL701N, AC707N
- **스피커**: JL701N, AC897, AD697N, AD698N, 700N

### ✨ 주요 특징

- ✅ BLE 단일/이중 백업 업그레이드 지원
- ✅ 장치 인증 및 페어링
- ✅ 브라우저 및 PC에서 OTA 파일 전송 지원
- ✅ BLE 광고 패킷 필터링
- ✅ 자동 재연결 기능
- ✅ 상세한 업그레이드 진행 상태 콜백

### 📦 SDK 버전

**최신 버전**: V2.4.0 (2025년 10월 13일)

**앱 버전**: V3.5.1 (2025년 10월 13일)

---

## 시작하기 전에

### 필수 요구 사항

#### 하드웨어
- iOS 12.0 이상을 실행하는 iPhone 또는 iPad
- 블루투스 4.0(BLE) 지원 필요

#### 소프트웨어
- **macOS**: 최신 버전 권장
- **Xcode**: 14.0 이상
- **iOS SDK**: 12.0 이상
- **CocoaPods**: 의존성 관리 도구

#### 지식 요구 사항
- Swift 또는 Objective-C 기본 지식
- iOS 앱 개발 경험
- Core Bluetooth 프레임워크에 대한 기본 이해

### ⚠️ 중요 사항

1. **권한 설정 필수**: 앱에서 블루투스 사용을 위한 권한을 반드시 설정해야 합니다.
2. **RCSP 프로토콜**: 모든 지원 장치는 RCSP 프로토콜을 지원해야 합니다.
3. **공개 기술만 사용**: 본 프로젝트는 공개 기술 정보 또는 자체 혁신 설계만 사용합니다.

---

## 개발 환경 설정

### 1단계: 저장소 클론

```bash
git clone https://github.com/Jieli-Tech/iOS-JL_OTA.git
cd iOS-JL_OTA
```

### 2단계: 의존성 설치

프로젝트는 CocoaPods를 사용합니다:

```bash
cd code/JL_OTA
pod install
```

### 3단계: 프로젝트 열기

```bash
open JL_OTA.xcworkspace
```

⚠️ **주의**: `.xcworkspace` 파일을 열어야 합니다. `.xcodeproj` 파일이 아닙니다!

### 4단계: 필수 프레임워크 추가

프로젝트에 다음 프레임워크가 포함되어 있는지 확인하세요:

#### 필수 프레임워크
- `JL_OTALib.framework` - OTA 업그레이드 핵심 라이브러리
- `JL_AdvParse.framework` - 블루투스 광고 패킷 파싱
- `JL_HashPair.framework` - 장치 인증
- `JLLogHelper.framework` - 로그 수집 및 관리

#### 선택적 프레임워크
- `JL_BLEKit.framework` - 杰理 통합 블루투스 라이브러리 (자체 블루투스 관리를 사용하는 경우 선택 사항)

### 5단계: Info.plist 권한 설정

`Info.plist` 파일에 다음 권한을 추가하세요:

```xml
<key>NSBluetoothPeripheralUsageDescription</key>
<string>이 앱은 블루투스 장치와 통신하기 위해 블루투스 권한이 필요합니다.</string>

<key>NSBluetoothAlwaysUsageDescription</key>
<string>이 앱은 장치 업그레이드를 위해 블루투스 권한이 필요합니다.</string>
```

---

## 빠른 시작

### 기본 OTA 업그레이드 흐름

```
1. 블루투스 장치 스캔
   ↓
2. 장치 연결
   ↓
3. 장치 정보 가져오기
   ↓
4. 업그레이드 필요 여부 확인
   ↓
5. OTA 파일 선택
   ↓
6. 업그레이드 시작
   ↓
7. 진행 상태 모니터링
   ↓
8. 완료 또는 오류 처리
```

### 30초 예제 (Objective-C)

```objective-c
// 1. OTA 매니저 초기화
JL_OTAManager *otaManager = [[JL_OTAManager alloc] init];
otaManager.delegate = self;

// 2. 장치 연결 알림
[otaManager noteEntityConnected];

// 3. 장치 정보 가져오기
[otaManager cmdTargetFeature];

// 4. OTA 업그레이드 시작
NSData *otaData = [NSData dataWithContentsOfFile:@"firmware.bfu"];
[otaManager cmdOTAData:otaData Result:^(JL_OTAResult result, float progress) {
    if (result == JL_OTAResultUpgrading) {
        NSLog(@"업그레이드 진행 중: %.2f%%", progress * 100);
    } else if (result == JL_OTAResultSuccess) {
        NSLog(@"업그레이드 성공!");
    }
}];
```

---

## SDK 통합 방법

SDK를 사용하는 두 가지 방법이 있습니다:

### 방법 1: 자체 블루투스 관리 (권장)

자체 블루투스 연결 로직을 구현하고 SDK는 OTA 데이터 파싱만 담당합니다.

**장점**:
- 완전한 제어 권한
- 기존 블루투스 코드와 통합 용이
- 더 유연한 커스터마이징

**사용 예시**: `BleManager` 폴더 참조

#### 주요 단계:

1. **초기화**
```objective-c
@interface JLBleManager() <JL_OTAManagerDelegate, JLHashHandlerDelegate>
@property (strong, nonatomic) JL_OTAManager *otaManager;
@property (strong, nonatomic) JLHashHandler *pairHash;
@end

- (instancetype)init {
    self = [super init];
    if (self) {
        _otaManager = [[JL_OTAManager alloc] init];
        _otaManager.delegate = self;
        
        self.pairHash = [[JLHashHandler alloc] init];
        self.pairHash.delegate = self;
        
        [JL_OTAManager logSDKVersion];
    }
    return self;
}
```

2. **BLE 서비스 및 특성 설정**
- 서비스 UUID: `AE00`
- 쓰기 특성: `AE01`
- 읽기/알림 특성: `AE02`

3. **데이터 수신 처리**
```objective-c
- (void)peripheral:(CBPeripheral *)peripheral 
  didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic 
  error:(NSError *)error {
    [_otaManager cmdOtaDataReceive:characteristic.value];
}
```

4. **데이터 전송 구현**
```objective-c
- (void)otaDataSend:(NSData *)data {
    // MTU 크기에 맞춰 데이터 분할 전송
    [self writeDataByCbp:data];
}
```

### 방법 2: JL_BLEKit 프레임워크 사용

杰理가 제공하는 통합 블루투스 라이브러리를 사용합니다.

**장점**:
- 빠른 구현
- 모든 블루투스 로직 자동 처리
- 테스트 완료

**사용 예시**: `SDKBleManager` 폴더 참조

#### 주요 단계:

1. **JL_BLEMultiple 초기화**
```objective-c
self.mBleMultiple = [[JL_BLEMultiple alloc] init];
self.mBleMultiple.BLE_FILTER_ENABLE = YES;  // 杰理 장치만 필터링
self.mBleMultiple.BLE_PAIR_ENABLE = YES;    // 페어링 활성화
self.mBleMultiple.BLE_TIMEOUT = 7;          // 연결 타임아웃
```

2. **장치 스캔 시작**
```objective-c
[[JL_RunSDK sharedInstance].mBleMultiple scanStart];
```

3. **장치 연결**
```objective-c
[[JL_RunSDK sharedInstance].mBleMultiple connectEntity:entity Result:^(JL_EntityM_STATUS status) {
    if (status == JL_EntityM_StatusConnected) {
        NSLog(@"연결 성공!");
    }
}];
```

---

## 주요 기능

### 1. 장치 스캔 및 연결

```objective-c
// 스캔 시작
[[JLBleManager sharedInstance] startScanBLE];

// 알림 수신
[[NSNotificationCenter defaultCenter] addObserver:self
    selector:@selector(onDeviceFound:)
    name:kFLT_BLE_FOUND
    object:nil];
```

### 2. 장치 인증

```objective-c
[_pairHash bluetoothPairingKey:self.pairKey Result:^(BOOL ret) {
    if (ret) {
        NSLog(@"페어링 성공");
    } else {
        NSLog(@"페어링 실패");
    }
}];
```

### 3. 장치 정보 가져오기

```objective-c
[_otaManager cmdTargetFeature];

// 델리게이트에서 응답 받기
- (void)otaFeatureResult:(JL_OTAManager *)manager {
    NSLog(@"펌웨어 버전: %@", manager.versionFirmware);
    NSLog(@"OTA 상태: %d", manager.otaStatus);
    
    if (manager.otaStatus == JL_OtaStatusForce) {
        NSLog(@"강제 업그레이드 필요!");
    }
}
```

### 4. OTA 업그레이드

```objective-c
NSData *otaData = [NSData dataWithContentsOfFile:otaFilePath];

[_otaManager cmdOTAData:otaData Result:^(JL_OTAResult result, float progress) {
    switch (result) {
        case JL_OTAResultPreparing:
            NSLog(@"파일 검증 중...");
            break;
            
        case JL_OTAResultUpgrading:
            NSLog(@"업그레이드 진행: %.1f%%", progress * 100);
            break;
            
        case JL_OTAResultReconnect:
            NSLog(@"장치 재연결 중...");
            // 자동으로 재연결 시도
            break;
            
        case JL_OTAResultSuccess:
            NSLog(@"업그레이드 성공!");
            break;
            
        case JL_OTAResultFail:
            NSLog(@"업그레이드 실패");
            break;
            
        default:
            break;
    }
}];
```

### 5. 업그레이드 취소

```objective-c
[_otaManager cmdOTACancelResult:^(uint8_t status, uint8_t sn, NSData *data) {
    NSLog(@"업그레이드 취소됨");
}];
```

### 6. 로그 관리

```objective-c
// 로그 활성화
[JLLogManager setLog:YES IsMore:NO Level:JLLOG_COMPLETE];
[JLLogManager saveLogAsFile:YES];

// 로그 비활성화
[JLLogManager setLog:NO IsMore:NO Level:JLLOG_COMPLETE];
[JLLogManager saveLogAsFile:NO];

// 로그 파일 경로 변경
NSString *logPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"ota.log"];
[JLLogManager redirectLogPath:logPath];

// 로그 수집
[JLLogManager collectLog:^(NSString *str) {
    NSLog(@"로그: %@", str);
}];
```

---

## 문제 해결

### 일반적인 문제

#### 1. 장치가 스캔되지 않음

**원인**:
- 블루투스 권한이 거부됨
- 장치가 광고 모드가 아님
- BLE 필터가 너무 제한적임

**해결 방법**:
```objective-c
// 블루투스 상태 확인
if (centralManager.state != CBManagerStatePoweredOn) {
    NSLog(@"블루투스가 꺼져 있습니다");
}

// 필터 비활성화 시도
self.mBleMultiple.BLE_FILTER_ENABLE = NO;
```

#### 2. 연결 실패

**원인**:
- 신호가 약함
- 장치가 다른 기기와 연결됨
- 타임아웃

**해결 방법**:
```objective-c
// 타임아웃 증가
self.mBleMultiple.BLE_TIMEOUT = 15;

// 재시도
[[JLBleManager sharedInstance] connectPeripheralWithUUID:deviceUUID];
```

#### 3. 페어링 실패

**원인**:
- 잘못된 페어링 키
- 장치가 이미 페어링됨

**해결 방법**:
```objective-c
// 페어링 초기화
[_pairHash hashResetPair];

// 올바른 키 사용
self.mAssist.mPairKey = nil; // 기본 키 사용
```

#### 4. OTA 업그레이드 실패

**원인**:
- 잘못된 펌웨어 파일
- 배터리 부족
- 연결 끊김

**해결 방법**:
```objective-c
// 배터리 확인
if (result == JL_OTAResultLowPower) {
    NSLog(@"배터리를 충전하세요");
    return;
}

// 파일 검증
if (result == JL_OTAResultFailErrorFile) {
    NSLog(@"펌웨어 파일이 손상되었습니다");
    return;
}

// 재연결 시도
if (result == JL_OTAResultReconnect) {
    [[JLBleManager sharedInstance] connectPeripheralWithUUID:lastUUID];
}
```

#### 5. MTU 크기 문제

**원인**:
- 일부 기기는 작은 MTU만 지원

**해결 방법**:
```objective-c
// MTU 크기 가져오기
NSUInteger mtu = [peripheral maximumWriteValueLengthForType:CBCharacteristicWriteWithoutResponse];
NSLog(@"MTU: %lu", (unsigned long)mtu);

// 데이터 분할 전송
- (void)writeDataByCbp:(NSData *)data {
    NSInteger len = data.length;
    while (len > 0) {
        if (len <= _bleMtu) {
            [_mBlePeripheral writeValue:data 
                      forCharacteristic:self.mRcspWrite 
                                   type:CBCharacteristicWriteWithoutResponse];
            break;
        } else {
            NSData *chunk = [data subdataWithRange:NSMakeRange(0, _bleMtu)];
            [_mBlePeripheral writeValue:chunk 
                      forCharacteristic:self.mRcspWrite 
                                   type:CBCharacteristicWriteWithoutResponse];
            data = [data subdataWithRange:NSMakeRange(_bleMtu, len - _bleMtu)];
            len -= _bleMtu;
        }
    }
}
```

### 디버깅 팁

1. **로그 활성화**: 항상 상세 로그를 활성화하여 문제를 추적하세요.

```objective-c
[JLLogManager setLog:YES IsMore:YES Level:JLLOG_COMPLETE];
```

2. **SDK 버전 확인**: 최신 버전을 사용하고 있는지 확인하세요.

```objective-c
[JL_OTAManager logSDKVersion];
[JLHashHandler sdkVersion];
```

3. **오류 코드 참조**: 각 `JL_OTAResult` 값의 의미를 확인하세요.

4. **알림 모니터링**: 모든 SDK 알림을 관찰하여 상태를 추적하세요.

```objective-c
[[NSNotificationCenter defaultCenter] addObserver:self
    selector:@selector(onBLEConnected:)
    name:kFLT_BLE_PAIRED
    object:nil];
    
[[NSNotificationCenter defaultCenter] addObserver:self
    selector:@selector(onBLEDisconnected:)
    name:kFLT_BLE_DISCONNECTED
    object:nil];
```

---

## 추가 리소스

### 📚 문서

- [README.md](README.md) - 프로젝트 개요 및 버전 정보 (중국어)
- [API 문서](doc/API%20说明.md) - 상세 API 레퍼런스 (중국어)
- [공식 문서](https://doc.zh-jieli.com/Apps/iOS/ota/zh-cn/master/index.html) - 온라인 문서 (중국어)

### 🔗 링크

- **GitHub 저장소**: [https://github.com/Jieli-Tech/iOS-JL_OTA](https://github.com/Jieli-Tech/iOS-JL_OTA)
- **앱 스토어**: "杰理OTA" 검색

### 📁 프로젝트 구조

```
iOS-JL_OTA/
├── code/
│   └── JL_OTA/                 # 메인 프로젝트
│       ├── Frameworks/         # 프레임워크 파일
│       ├── JL_OTA/             # 앱 소스 코드
│       ├── Sources/            # 추가 소스
│       └── OTAFiles/           # 샘플 OTA 파일
├── doc/                        # 문서
│   ├── API 说明.md             # API 문서
│   ├── Release_V2.3.1/         # 이전 릴리스
│   └── Release_V2.4.0/         # 최신 릴리스
├── libs/                       # 라이브러리
├── README.md                   # 프로젝트 README
└── ONBOARDING.md              # 이 파일
```

### 🎓 학습 경로

1. **기초**: Core Bluetooth 프레임워크 이해
2. **중급**: 데모 앱 실행 및 코드 탐색
3. **고급**: 자체 앱에 SDK 통합
4. **전문가**: OTA 프로세스 커스터마이징

### 💡 모범 사례

1. **항상 에러 처리**: 모든 OTA 결과를 적절히 처리하세요.
2. **사용자 피드백 제공**: 진행 상태를 명확하게 표시하세요.
3. **연결 안정성 테스트**: 약한 신호 조건에서 테스트하세요.
4. **배터리 확인**: 업그레이드 전 배터리 상태를 확인하세요.
5. **파일 검증**: OTA 파일의 무결성을 확인하세요.

### ❓ 도움이 필요하신가요?

- **이슈 리포트**: [GitHub Issues](https://github.com/Jieli-Tech/iOS-JL_OTA/issues)
- **풀 리퀘스트**: 기여를 환영합니다!
- **코드 예제**: `code/JL_OTA` 폴더의 데모 앱 참조

---

## 시작하기

이제 모든 준비가 완료되었습니다! 🎉

다음 단계:
1. ✅ 개발 환경 설정 완료
2. ✅ 데모 앱 실행해보기
3. ✅ 자체 프로젝트에 SDK 통합
4. ✅ 첫 번째 OTA 업그레이드 테스트

행운을 빕니다! 즐거운 개발 되세요! 🚀

---

*이 온보딩 가이드는 iOS JL_OTA SDK v2.4.0 기준으로 작성되었습니다.*

*최종 업데이트: 2025년 11월*
