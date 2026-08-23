# Using the Codec-Enabled Avalonia Package

## Install From a Private Feed

The prebuilt CEF 134 Windows x64 NuGet source is available from the [GitHub
Release](https://github.com/TrumpCat/WebView-Avalonia/releases/tag/cef-134.3.9-codecs.1-win-x64).
Download [cef-134-win-x64-nuget.zip](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/cef-134-win-x64-nuget.zip),
extract it, and add the extracted folder as a local source:

```powershell
Expand-Archive .\cef-134-win-x64-nuget.zip -DestinationPath C:\nuget\cef134
dotnet nuget add source C:\nuget\cef134 --name CEF134Codecs
```

Configure the feed and install the top-level package:

```powershell
dotnet nuget add source \\server\cef-nuget --name CompanyCef
dotnet add package WebViewControl-Avalonia `
  --version 3.134.178-codecs.1 `
  --source CompanyCef
```

The package brings in the matching CEF runtime, CefGlue packages and
browser-process executable. Do not mix package versions from another CEF line.

The ZIP is only a transport container. NuGet needs the extracted directory,
which contains these packages:

```text
chromiumembeddedframework.runtime.134.3.9-codecs.1.nupkg
chromiumembeddedframework.runtime.win-x64.134.3.9-codecs.1.nupkg
CefGlue.Common.134.6998.178-9n1m.1.nupkg
CefGlue.Avalonia.134.6998.178-9n1m.1.nupkg
WebViewControl-Avalonia.3.134.178-codecs.1.nupkg
```

## Project File

For a normal Windows x64 Avalonia application:

```xml
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <RuntimeIdentifier>win-x64</RuntimeIdentifier>
  <PlatformTarget>x64</PlatformTarget>
</PropertyGroup>

<ItemGroup>
  <PackageReference Include="WebViewControl-Avalonia"
                    Version="3.134.178-codecs.1" />
</ItemGroup>
```

Build and publish the application normally:

```powershell
dotnet restore
dotnet publish -c Release -r win-x64 --self-contained true
```

The package targets place CEF resources under the expected `locales`
directory and copy the browser process to
`CefGlueBrowserProcess/9n1m.webview.exe`.

## Runtime Checklist

The application must deploy all files produced by `dotnet publish`, including:

- `libcef.dll`, CEF `.pak` files and `icudtl.dat`;
- the `locales` directory;
- `CefGlueBrowserProcess/9n1m.webview.exe`;
- the native support DLLs copied by the runtime package.

Do not remove the locale files or rename the browser-process executable.
The CEF runtime is initialized by the WebView library before the first
browser control is created.

## Media Validation

Test an actual H.264 and H.265 stream in the target application. Chromium's
`HTMLMediaElement.canPlayType()` is useful as a preflight check but does not
prove that a specific stream will decode.

H.265 playback can fail even with the codec-enabled CEF build when the target
machine lacks one of the following:

- the Windows HEVC Video Extensions component;
- a compatible GPU driver and DXVA path;
- hardware support for the stream profile, bit depth or chroma format.

Collect `chrome://gpu` output and the CEF log when diagnosing a failure.

## CEF Lines

Use CEF 134 for supported current Windows systems and .NET 8. Use CEF 106
only for the Windows 7 compatibility line and .NET 6. The lines have separate
runtime, CefGlue and WebView package versions and must not be combined.

## Run The Prebuilt Demo

Download the [compiled Demo](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/html5test-demo-win-x64.tar.gz),
extract it, and run:

```powershell
New-Item -ItemType Directory -Force .\cef134-demo | Out-Null
tar -xzf .\html5test-demo-win-x64.tar.gz -C .\cef134-demo
Start-Process .\cef134-demo\SampleWebView.Avalonia.exe
```

Keep all files in the extracted directory, including `libcef.dll`, `.pak`
resources, `locales` and `CefGlueBrowserProcess\9n1m.webview.exe`.
