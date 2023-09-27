# dotnet8_linux_winforms

## 概要

↓のWinForms版の検証

https://github.com/Tobotobo/dotnet8_linux_wpf  
.NET 7 で WPF を Linux ビルドできるようになっているという情報を入手したため  
本当にできるのか実験した。  
なお、本実験は .NET 8 rc で実施している。  

## 結果概要
できる。  
が、NativeAOTとトリミングについてはWPFと一緒でまだダメ。

## 環境
```
# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```
```
# dotnet --version
8.0.100-rc.1.23455.8
```

## メモ

winformsのテンプレートをインストール
C:\Program Files\dotnet\templates\8.0.0-rc.1.23421.29\microsoft.dotnet.winforms.projecttemplates.8.0.0-rc.1.23419.5.nupkg
```
dotnet new --install ./microsoft.dotnet.winforms.projecttemplates.8.0.0-rc.1.23419.5.nupkg 
```

### dotnet8_linux_winforms の PropertyGroup に以下を追加
```
<EnableWindowsTargeting>true</EnableWindowsTargeting>
```

### ビルド
→　できる。
```
dotnet build
```
```
# dotnet build
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj を復元しました (3.37 sec)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Debug/net8.0-windows/dotnet8_linux_winforms.dll

ビルドに成功しました。
    0 個の警告
    0 エラー

経過時間 00:00:08.74
```

### パブリッシュ
→　できる。動く。
```
dotnet publish -r win-x64 --no-self-contained
```
```
# dotnet publish -r win-x64 --no-self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj を復元しました (2.39 sec)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/dotnet8_linux_winforms.dll
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/publish/
```
```
# du -k ./bin/Release/net8.0-windows/win-x64/publish/* | awk -F/ '{printf "%-45s %10d KB\n", $NF, $1}'
dotnet8_linux_winforms.deps.json                       4 KB
dotnet8_linux_winforms.dll                             8 KB
dotnet8_linux_winforms.exe                           156 KB
dotnet8_linux_winforms.pdb                            16 KB
dotnet8_linux_winforms.runtimeconfig.json              4 KB
```

### 自己完結 + シングルバイナリ
→　できるが相変わらず周囲に dll が残っていてシングルじゃない
```
dotnet publish -r win-x64 -p:PublishSingleFile=true --self-contained
```
```
# dotnet publish -r win-x64 -p:PublishSingleFile=true --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj を復元しました (10.83 sec)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/dotnet8_linux_winforms.dll
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/publish/
```
```
# du -k ./bin/Release/net8.0-windows/win-x64/publish/* | awk -F/ '{printf "%-45s %10d KB\n", $NF, $1}'
D3DCompiler_47_cor3.dll                             4804 KB
PenImc_cor3.dll                                      156 KB
PresentationNative_cor3.dll                         1208 KB
dotnet8_linux_winforms.exe                        149812 KB
dotnet8_linux_winforms.pdb                            16 KB
vcruntime140_cor3.dll                                108 KB
wpfgfx_cor3.dll                                     1916 KB
```

### 自己完結 + シングルバイナリ + トリミング
→　ダメ。  
https://learn.microsoft.com/ja-jp/dotnet/core/deploying/trimming/trim-self-contained
```
dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true --self-contained
```
```
# dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  復元対象のすべてのプロジェクトは最新です。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(255,5): error NETSDK1175: Windows Forms is not supported or recommended with trimming enabled. Please go to https://aka.ms/dotnet-illink/windows-forms for more details. [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
```

### 自己完結 + シングルバイナリ + トリミング + 警告無視
→　ダメ。生成はできるが起動できない。exeを実行すると一瞬カーソルがグルグルするだけでウィンドウがでない。
https://stackoverflow.com/questions/73590300/how-to-publish-trimmed-windows-forms-application  
<_SuppressWinFormsTrimError>true</_SuppressWinFormsTrimError>  
```
dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true -p:_SuppressWinFormsTrimError=true --self-contained
```
```
# dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true -p:_SuppressWinFormsTrimError=true --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  復元対象のすべてのプロジェクトは最新です。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/dotnet8_linux_winforms.dll
  Optimizing assemblies for size may change the behavior of the app. Be sure to test after publishing. See: https://aka.ms/dotnet-illink
  Optimizing assemblies for size. This process might take a while.
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/publish/
```
```
# du -k ./bin/Release/net8.0-windows/win-x64/publish/* | awk -F/ '{printf "%-45s %10d KB\n", $NF, $1}'
D3DCompiler_47_cor3.dll                             4804 KB
PenImc_cor3.dll                                      156 KB
PresentationNative_cor3.dll                         1208 KB
dotnet8_linux_winforms.exe                         53252 KB
dotnet8_linux_winforms.pdb                            16 KB
vcruntime140_cor3.dll                                108 KB
wpfgfx_cor3.dll                                     1916 KB
```

### NativeAOT
→　やっぱ無理  
https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/incompatibilities
```
dotnet publish -r win-x64 -p:PublishAot=true
```
```
# dotnet publish -r win-x64 -p:PublishAot=true
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj を復元しました (15.09 sec)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(255,5): error NETSDK1175: Windows Forms is not supported or recommended with trimming enabled. Please go to https://aka.ms/dotnet-illink/windows-forms for more details. [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
```

### NativeAOT + 警告無視
→　ダメ。NativeAOTはクロスコンパイル無理
```
dotnet publish -r win-x64 -p:PublishAot=true -p:_SuppressWinFormsTrimError=true
```
```
# dotnet publish -r win-x64 -p:PublishAot=true -p:_SuppressWinFormsTrimError=true
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  復元対象のすべてのプロジェクトは最新です。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> /home/dotnet8_linux_winforms/bin/Release/net8.0-windows/win-x64/dotnet8_linux_winforms.dll
/root/.nuget/packages/microsoft.dotnet.ilcompiler/8.0.0-rc.1.23419.4/build/Microsoft.NETCore.Native.Publish.targets(62,5): error : Cross-OS native compilation is not supported. [/home/dotnet8_linux_winforms/dotnet8_linux_winforms.csproj]
```

Epic - WinForms トリムに互換性を持たせる #4649  
https://github.com/dotnet/winforms/issues/4649



## 参考
- .NET 7 で WPF を Linux ビルドする  
  https://tech.guitarrapc.com/entry/2022/11/11/031555