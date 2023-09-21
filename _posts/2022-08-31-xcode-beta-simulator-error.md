---
title: "Flutter Research - Xcode15 Beta8 업데이트 이후 ios 시뮬레이터 Build Error"
categories:
  - Dart/Flutter
tags:
  - Flutter Research

toc: true
toc_sticky : true

date : 2023-08-31
last_modified_at : 2023-08-31
---

# Beginning
Mac 버전을 Sonoma 14.0(Beta)로 업그레이드 하면서 Flutter로 앱 개발을 진행하던 중 Xcode를 실행하려 하니 금지 아이콘과 함께 지원되지 않는 프로그램이라는 내용을 담은 문구가 나왔습니다. 그래서 이것저것 알아본 끝에 Xcode Beta를 설치해서 사용하고 있었습니다.

그러다가 ios Simulator의 키보드가 잘 켜지지 않고 렉이 걸리는 현상이 있어서 Xcode15 Beta8로 업데이트 하게 되었습니다.

# Problem
그런데 ios Simulator에서 앱을 build하니 다음과 같은 오류가 발생했다.

```
Launching lib/main.dart on iPhone 14 in debug mode...
Running Xcode build...
Xcode build done.                                            4.7s
Failed to build iOS app
Error (Xcode): DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead

Could not build the application for the simulator.
Error launching application on iPhone 14.
```

# Solution
Github의 Issue들을 찾아본 끝에 Flutter 프로젝트의 ios 폴더 속 Podfile에 아래의 코드를 추가하니 해결되었습니다.

```swift
post_install do |installer|
      installer.pods_project.targets.each do |target|
          target.build_configurations.each do |config|
          xcconfig_path = config.
          //아래 코드를 추가
          base_configuration_reference.real_path
          xcconfig = File.read(xcconfig_path)
          xcconfig_mod = xcconfig.gsub(/DT_TOOLCHAIN_DIR/, "TOOLCHAIN_DIR")
          File.open(xcconfig_path, "w") { |file| file << xcconfig_mod }

          //...

          end
      end
  end
```