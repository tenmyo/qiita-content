<!--
title:   Android Studioのエミュレータをコマンドプロンプトから実行したい
tags:    AndroidSDK,AndroidStudio,emulator
id:      f547806f128606ae22d3
private: false
-->
AndroidのエミュレータにProxyとか設定したくて、コマンドプロンプト（やAndroid StudioのTerminal）から実行したいのですよ。
ところがよく分からないエラーが出てしまうのです。

```text:エラー！
> emulator @Nexus_5_API_22
[8920]:ERROR:./android/qt/qt_setup.cpp:28:Qt library not found at ..\emulator\lib64\qt\lib
Could not launch '..\emulator/qemu/windows-x86_64/qemu-system-i386.exe': No such file or directory
```

どうもAndroid SDKに複数エミュレータが入ってしまっていて、ダメなemulatorを起動しているみたい。

```text:emulatorの場所
> echo %ANDROID_HOME%
C:\Android\android-sdk

> where /R %ANDROID_HOME% emulator.exe
C:\Android\android-sdk\emulator\emulator.exe
C:\Android\android-sdk\tools\emulator.exe

> where emulator
C:\Android\android-sdk\tools\emulator.exe
```

別のほうのエミュレータを直接叩いたらうまく起動できました。

```text:違う方を実行
> %ANDROID_HOME%\emulator\emulator @Nexus_5_API_22
```