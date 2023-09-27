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

### 自己完結 + シングルバイナリ + トリミング + 警告無視　※Windowsでも実施
→　やっぱダメ。実行してもグルグルするだけ。
```
> cmd -c ver
Microsoft Windows [Version 10.0.22621.2283]
```
```
> dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true -p:_SuppressWinFormsTrimError=true --self-contained

.NET 8.0 へようこそ!
---------------------
SDK バージョン: 8.0.100-rc.1.23463.5

----------------
Installed an ASP.NET Core HTTPS development certificate.
To trust the certificate, run 'dotnet dev-certs https --trust'
Learn about HTTPS: https://aka.ms/dotnet-https

----------------
ASP.NET Core の HTTPS 開発証明書をインストールしました。
証明書を信頼するには、'dotnet dev-certs https --trust' (Windows および macOS のみ) を実行します。
HTTPS の詳細については、https://aka.ms/dotnet-https を参照してください
----------------
最初のアプリを作成するには、https://aka.ms/dotnet-hello-world を参照してください
最新情報については、https://aka.ms/dotnet-whats-new を参照してください
ドキュメントを探索するには、https://aka.ms/dotnet-docs を参照してください
GitHub で問題の報告とソースの検索を行うには、https://github.com/dotnet/core を参照してください
'dotnet --help' を使用して使用可能なコマンドを確認するか、https://aka.ms/dotnet-cli にアクセスしてください
--------------------------------------------------------------------------------------
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj を復元しました (4.89 sec)。
C:\Program Files\dotnet\sdk\8.0.100-rc.1.23463.5\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用していま
す。https://aka.
ms/dotnet-support-policy をご覧ください [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\bin\Release\net8.0-windows\win-x64\dotnet8_linux_winforms.dll
  Optimizing assemblies for size may change the behavior of the app. Be sure to test after publishing. See: https://aka.ms/dotnet-illink
  Optimizing assemblies for size. This process might take a while.
  dotnet8_linux_winforms -> C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\bin\Release\net8.0-windows\win-x64\publish\
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

### NativeAOT + 警告無視 + Windows
→　たぶんダメ？exeは作れたし実行できたけど、実行時にCOMのエラーが起きる。  
無視して続行できたが、この後どうなるかわからない。
```
> cmd -c ver
Microsoft Windows [Version 10.0.22621.2283]
```
```
>dotnet publish -r win-x64 -p:PublishAot=true -p:_SuppressWinFormsTrimError=true
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj を復元しました (4.84 sec)。
C:\Program Files\dotnet\sdk\8.0.100-rc.1.23463.5\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用していま
す。https://aka.
ms/dotnet-support-policy をご覧ください [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
  dotnet8_linux_winforms -> C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\bin\Release\net8.0-windows\win-x64\dotnet8_linux_winforms.dll
  Optimizing assemblies for size may change the behavior of the app. Be sure to test after publishing. See: https://aka.ms/dotnet-illink
  Generating native code
C:\Users\tobot\.nuget\packages\runtime.win-x64.microsoft.dotnet.ilcompiler\8.0.0-rc.1.23419.4\framework\System.ComponentModel.TypeConverter.dll : warning IL3053: Assembly 'System.ComponentMod
el.TypeConverter' produced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3000: System.Windows.Forms.ThreadExceptionDialog.ThreadExceptionDialog(Exception): 'System.Reflection.Assembly.Location.get' always returns an empty string for assemblies embe
dded in a single-file app. If the path to the app directory is needed, consider calling 'System.AppContext.BaseDirectory'. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_ 
winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\System.Windows.Forms.Primitives.dll : warning IL3053: Assembly 'Syst
em.Windows.Forms.Primitives' produced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\System.Windows.Forms.dll : warning IL3053: Assembly 'System.Windows. 
Forms' produced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\System.Windows.Forms.Design.dll : warning IL3053: Assembly 'System.W
indows.Forms.Design' produced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3000: System.Drawing.Design.ToolboxItem.GetType(IDesignerHost,AssemblyName,String,Boolean): 'System.Reflection.AssemblyName.CodeBase.get' always returns an empty string for as
semblies embedded in a single-file app. If the path to the app directory is needed, consider calling 'System.AppContext.BaseDirectory'. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\d 
otnet8_linux_winforms.csproj]
ILC : warning IL3000: System.Drawing.Design.ToolboxItem.GetType(IDesignerHost,AssemblyName,String,Boolean): 'System.Reflection.AssemblyName.CodeBase.get' always returns an empty string for as 
semblies embedded in a single-file app. If the path to the app directory is needed, consider calling 'System.AppContext.BaseDirectory'. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\d 
otnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\WindowsBase.dll : warning IL3053: Assembly 'WindowsBase' produced AO
T analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\System.Xaml.dll : warning IL3053: Assembly 'System.Xaml' produced AO
T analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\PresentationFramework.dll : warning IL3053: Assembly 'PresentationFr
amework' produced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3000: MS.Internal.AppModel.ContentFilePart.GetEntryAssemblyLocation(): 'System.Reflection.Assembly.Location.get' always returns an empty string for assemblies embedded in a si
ngle-file app. If the path to the app directory is needed, consider calling 'System.AppContext.BaseDirectory'. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csp 
roj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\PresentationCore.dll : warning IL3053: Assembly 'PresentationCore' p
roduced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3002: System.Windows.Documents.TextRangeSerialization.WriteStartXamlElement(ITextRange,ITextPointer,XmlWriter,XamlTypeMapper,Boolean,Boolean): Using member 'System.Reflection.
Module.Name.get' which has 'RequiresAssemblyFilesAttribute' can break functionality when embedded in a single-file app. Returns <Unknown> for modules with no file path. [C:\Users\tobot\Deskto 
p\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3002: System.Windows.Documents.TextRangeSerialization.WriteStartXamlElement(ITextRange,ITextPointer,XmlWriter,XamlTypeMapper,Boolean,Boolean): Using member 'System.Reflection. 
Module.Name.get' which has 'RequiresAssemblyFilesAttribute' can break functionality when embedded in a single-file app. Returns <Unknown> for modules with no file path. [C:\Users\tobot\Deskto 
p\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3002: System.Windows.Documents.TextRangeSerialization.WriteStartXamlElement(ITextRange,ITextPointer,XmlWriter,XamlTypeMapper,Boolean,Boolean): Using member 'System.Reflection. 
Module.Name.get' which has 'RequiresAssemblyFilesAttribute' can break functionality when embedded in a single-file app. Returns <Unknown> for modules with no file path. [C:\Users\tobot\Deskto 
p\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\runtime.win-x64.microsoft.dotnet.ilcompiler\8.0.0-rc.1.23419.4\framework\System.Linq.Expressions.dll : warning IL3053: Assembly 'System.Linq.Expressions' produc
ed AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
C:\Users\tobot\.nuget\packages\microsoft.windowsdesktop.app.runtime.win-x64\8.0.0-rc.1.23420.5\runtimes\win-x64\lib\net8.0\ReachFramework.dll : warning IL3053: Assembly 'ReachFramework' produ 
ced AOT analysis warnings. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
ILC : warning IL3000: WinRT.DllModule..cctor(): 'System.Reflection.Assembly.Location.get' always returns an empty string for assemblies embedded in a single-file app. If the path to the app d
irectory is needed, consider calling 'System.AppContext.BaseDirectory'. [C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\dotnet8_linux_winforms.csproj]
  ILC: Method '[PresentationFramework]System.Windows.Documents.MsSpellCheckLib.RCW+SpellCheckerFactoryCoClass..ctor()' will always throw because: Invalid IL or CLR metadata in 'Void SpellChec
  kerFactoryCoClass..ctor()'
  ILC: Method '[PresentationFramework]System.Windows.Documents.MsSpellCheckLib.RCW+SpellCheckerFactoryCoClass.UnregisterUserDictionary(string,string)' will always throw because: Invalid IL or 
   CLR metadata in 'Void SpellCheckerFactoryCoClass.UnregisterUserDictionary(System.String, System.String)'
  ILC: Method '[PresentationFramework]System.Windows.Documents.MsSpellCheckLib.RCW+SpellCheckerFactoryCoClass.RegisterUserDictionary(string,string)' will always throw because: Invalid IL or C 
  LR metadata in 'Void SpellCheckerFactoryCoClass.RegisterUserDictionary(System.String, System.String)'
  ILC: Method '[PresentationFramework]System.Windows.Documents.MsSpellCheckLib.RCW+SpellCheckerFactoryCoClass.CreateSpellChecker(string)' will always throw because: Invalid IL or CLR metadata 
   in 'ISpellChecker SpellCheckerFactoryCoClass.CreateSpellChecker(System.String)'
  ILC: Method '[PresentationCore]MS.Internal.AppModel.CustomCredentialPolicy+InternetSecurityManager..ctor()' will always throw because: Invalid IL or CLR metadata in 'Void InternetSecurityMa
  nager..ctor()'
  ILC: Method '[WindowsBase]MS.Internal.Security.AttachmentService+AttachmentServices..ctor()' will always throw because: Invalid IL or CLR metadata in 'Void AttachmentServices..ctor()'       
  ILC: Method '[System.DirectoryServices]System.DirectoryServices.UnsafeNativeMethods+PropertyEntry..ctor()' will always throw because: Invalid IL or CLR metadata in 'Void PropertyEntry..ctor
  ()'
  ILC: Method '[System.DirectoryServices]System.DirectoryServices.UnsafeNativeMethods+PropertyValue..ctor()' will always throw because: Invalid IL or CLR metadata in 'Void PropertyValue..ctor 
  ()'
  dotnet8_linux_winforms -> C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\bin\Release\net8.0-windows\win-x64\publish\
```
```
>dir
 ドライブ C のボリューム ラベルは Windows です
 ボリューム シリアル番号は E49E-31A4 です

 C:\Users\tobot\Desktop\dotnet\dotnet8_linux_winforms\bin\Release\net8.0-windows\win-x64\native のディレクトリ

2023/09/27  14:47    <DIR>          .
2023/09/27  14:47    <DIR>          ..
2023/09/27  14:47        56,771,584 dotnet8_linux_winforms.exe
2023/09/27  14:47       211,513,344 dotnet8_linux_winforms.pdb
               2 個のファイル         268,284,928 バイト
               2 個のディレクトリ  90,500,292,608 バイトの空き領域
```
![](doc/image/スクリーンショット%202023-09-27%20145147.png)

## 参考
- .NET 7 で WPF を Linux ビルドする  
  https://tech.guitarrapc.com/entry/2022/11/11/031555