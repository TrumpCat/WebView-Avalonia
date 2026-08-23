# CEF Codec Package Distribution

This project builds CEF 134 with the Chromium codec flags required for H.264
and platform HEVC/H.265 paths on Windows x64. The build also keeps CEF 106 as
the Windows 7 compatibility line.

## Build Outputs

Run the Windows wrapper from the repository root:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass `
  -File .\package-cef-106-134.ps1 `
  -Line 134 `
  -Rid win-x64 `
  -Proxy http://127.0.0.1:7897
```

The proxy is optional. It can also be supplied with `CEF_PROXY`.

Default output locations:

```text
dist/cef-134/win-x64/nuget/  NuGet packages
dist/cef-134/win-x64/demo/   Self-contained Avalonia demo
dist/logs/                   Build logs
```

The package feed must contain the complete set of packages. For CEF 134
Windows x64 this includes:

```text
chromiumembeddedframework.runtime.134.3.9-codecs.1.nupkg
chromiumembeddedframework.runtime.win-x64.134.3.9-codecs.1.nupkg
CefGlue.Common.134.6998.178-9n1m.1.nupkg
CefGlue.Avalonia.134.6998.178-9n1m.1.nupkg
WebViewControl-Avalonia.3.134.178-codecs.1.nupkg
```

Consumers reference only `WebViewControl-Avalonia`; NuGet resolves the other
managed and native runtime dependencies automatically.

## Local Private Feed

For a small team, put the complete `nuget` directory on an SMB share:

```powershell
dotnet nuget add source \\server\cef-nuget --name CompanyCef
dotnet add package WebViewControl-Avalonia `
  --version 3.134.178-codecs.1 `
  --source CompanyCef
```

Alternatively add the source in `NuGet.Config`:

```xml
<configuration>
  <packageSources>
    <add key="CompanyCef" value="\\server\cef-nuget" />
  </packageSources>
</configuration>
```

For a hosted private feed use Azure Artifacts, GitHub Packages, ProGet or
BaGet. Upload all packages for one version as one release operation; do not
upload only the top-level package or only the RID-specific runtime package.

## GitHub Release Assets

GitHub rejects ordinary repository files larger than 100 MB. The native CEF
DLL and the complete NuGet feed are therefore distributed as a Release asset,
not committed to the source tree. Create the bundle locally:

```powershell
Compress-Archive `
  -Path .\dist\cef-134\win-x64\nuget, .\dist\cef-134\win-x64\demo `
  -DestinationPath .\dist\cef-134-win-x64-bundle.zip
Get-FileHash .\dist\cef-134-win-x64-bundle.zip -Algorithm SHA256
```

Upload the ZIP and its SHA-256 value to a GitHub Release for the matching
source revision. The source PR should document the Release URL and package
version after the Release is created.

The current CEF 134 Windows x64 assets are available here:

- [NuGet feed ZIP](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/cef-134-win-x64-nuget.zip)
- [Compiled Demo](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/html5test-demo-win-x64.tar.gz)
- [NuGet + Demo bundle](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/cef-134-win-x64-bundle.zip)
- [SHA-256 checksums](https://github.com/TrumpCat/WebView-Avalonia/releases/download/cef-134.3.9-codecs.1-win-x64/cef-134-win-x64-SHA256SUMS.txt)

The assets are hosted on the builder fork because the current GitHub account
does not have direct write permission to `Timskt/WebView-Avalonia`. The PR
targets that repository's `feature/h264-h265-codecs` branch.

## Download And Install

To use the NuGet packages, download `cef-134-win-x64-nuget.zip`, extract it,
and register the extracted directory as a local NuGet source:

```powershell
Expand-Archive .\cef-134-win-x64-nuget.zip -DestinationPath C:\nuget\cef134
dotnet nuget add source C:\nuget\cef134 --name CEF134Codecs
dotnet add package WebViewControl-Avalonia `
  --version 3.134.178-codecs.1 `
  --source CEF134Codecs
```

The extracted directory must contain all five `.nupkg` files. Do not point
NuGet at the ZIP file itself.

To run the compiled Demo, download `html5test-demo-win-x64.tar.gz`, extract it
with 7-Zip or Windows `tar`, then run the executable from the extracted folder:

```powershell
New-Item -ItemType Directory -Force .\cef134-demo | Out-Null
tar -xzf .\html5test-demo-win-x64.tar.gz -C .\cef134-demo
Start-Process .\cef134-demo\SampleWebView.Avalonia.exe
```

The Demo is a CEF 134 `win-x64` build. Use it on a Windows x64 machine and
keep the entire extracted directory together.

## Demo

Extract the Demo archive on a Windows x64 machine and run:

```powershell
.\SampleWebView.Avalonia.exe
```

The demo uses the codec-enabled CEF 134 runtime. H.264 playback is expected
when the media test stream is compatible. H.265 playback still depends on the
Windows HEVC component, GPU driver and hardware decode support. A successful
`canPlayType()` result is only a capability hint; validate with an actual
video stream on the target hardware.

## Verification

Before publishing a package set, run:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass `
  -File .\package-cef-106-134.ps1 `
  -Line 134 `
  -PreflightOnly
```

For source-only validation:

```powershell
git diff --check
dotnet build .\WebView\SampleWebView.Avalonia\SampleWebView.Avalonia.csproj `
  -c ReleaseAvalonia -p:Platform=x64 -p:CefLine=134 `
  --no-restore
```

Codec licensing, patent and target-system HEVC requirements remain the
responsibility of the distributor and application owner.
