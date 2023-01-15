Hi! I am trying to build Clipboard from source under Win11 with VS2022 Preview Community Edition. Since my local language is Chinese Simplified, the default code page of my machine is CP936.

#1 Try:

```
pushd build
cmake .. -A Win32 > build.log
cmake --build . >> build.log
type build.log
```

Output:

```
-- Building for: Visual Studio 17 2022
-- The CXX compiler identification is MSVC 19.35.32019.0
-- The C compiler identification is MSVC 19.35.32019.0
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: D:/Program Files/Microsoft Visual Studio/2022/Preview/VC/Tools/MSVC/14.35.32019/bin/Hostx64/x86/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: D:/Program Files/Microsoft Visual Studio/2022/Preview/VC/Tools/MSVC/14.35.32019/bin/Hostx64/x86/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Looking for fork
-- Looking for fork - not found
-- Building Clipboard without X11 support
-- Could NOT find PkgConfig (missing: PKG_CONFIG_EXECUTABLE) 
-- Building Clipboard without Wayland support
-- Looking for pthread_cancel
-- Looking for pthread_cancel - not found
-- Looking for pthread.h
-- Looking for pthread.h - not found
-- Found Threads: TRUE  
-- Configuring done
-- Generating done
-- Build files have been written to: C:/Users/hp/Desktop/Clipboard/build
MSBuild version 17.5.0-preview-22525-01+3b7246b65 for .NET Framework
  Checking Build System
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/src/gui/CMakeLists.txt
  fork.cpp
  gui.cpp
  utils.cpp
  Generating Code...
  gui.vcxproj -> C:\Users\hp\Desktop\Clipboard\build\gui.lib
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/src/clipboard/CMakeLists.txt
  clipboard.cpp
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(1,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(683,228): error C2001: newline in constant [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(684,5): error C2062: type 'unsigned int' unexpected [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(688,13): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(689,100): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(710,13): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(711,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(711,100): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  messages.cpp
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(1,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  windows.cpp
```

#2 Try:

Patch src/clipboard/CMakeLists.txt:

```diff
diff --git a/src/clipboard/CMakeLists.txt b/src/clipboard/CMakeLists.txt
index 1d4cd02..64d9ab9 100644
--- a/src/clipboard/CMakeLists.txt
+++ b/src/clipboard/CMakeLists.txt
@@ -22,7 +22,7 @@ target_link_libraries(clipboard gui)
 
 if(WIN32)
   target_sources(clipboard PRIVATE src/windows.cpp)
+  target_compile_options(clipboard PRIVATE /source-charset:utf-8)
 elseif(APPLE)
   enable_language(OBJC)
   target_sources(clipboard PRIVATE

```

Build log:

```

```

#3 Try:

Patch CMakeLists.txt:

```diff
diff --git a/build/.gitignore b/build/.gitignore
deleted file mode 100644
index 77c2533..0000000
--- a/build/.gitignore
+++ /dev/null
@@ -1,6 +0,0 @@
-*
-*vscode*
-*idea*
-settings.json
-dummy/
-!.gitignore
\ No newline at end of file
diff --git a/src/clipboard/CMakeLists.txt b/src/clipboard/CMakeLists.txt
index 8cd0b3e..b90e608 100644
--- a/src/clipboard/CMakeLists.txt
+++ b/src/clipboard/CMakeLists.txt
@@ -16,6 +16,7 @@ target_link_libraries(clipboard gui)
 
 if(WIN32)
   target_sources(clipboard PRIVATE src/windows.cpp)
+  target_compile_options(clipboard PRIVATE /utf-8)
 elseif(APPLE)
   enable_language(OBJC)
   target_sources(clipboard PRIVATE

```

Build log:

```
-- Building for: Visual Studio 17 2022
-- The CXX compiler identification is MSVC 19.35.32019.0
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: D:/Program Files/Microsoft Visual Studio/2022/Preview/VC/Tools/MSVC/14.35.32019/bin/Hostx64/x86/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Building Clipboard without X11 support
-- Could NOT find PkgConfig (missing: PKG_CONFIG_EXECUTABLE) 
-- Building Clipboard without Wayland support
-- Looking for C++ include pthread.h
-- Looking for C++ include pthread.h - not found
-- Found Threads: TRUE  
-- Performing Test COMPILER_SUPPORTS_MARCH_NATIVE
-- Performing Test COMPILER_SUPPORTS_MARCH_NATIVE - Failed
-- Configuring done
-- Generating done
-- Build files have been written to: C:/Users/hp/Desktop/Clipboard/build
MSBuild version 17.5.0-preview-22525-01+3b7246b65 for .NET Framework
  Checking Build System
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/src/clipboard/CMakeLists.txt
  clipboard.cpp
  messages.cpp
  windows.cpp
  Generating code
  Not all modules are compiled with -Gy (function comdat), build without incremental LTCG.
  Finished generating code
  clipboard.vcxproj -> C:\Users\hp\Desktop\Clipboard\build\clipboard.exe
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/CMakeLists.txt
```

Add manifest file:

https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-utf8-code-page

https://stackoverflow.com/questions/6335352/how-can-i-embed-a-specific-manifest-file-in-a-windows-dll-with-a-cmake-build


```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly manifestVersion="1.0" xmlns="urn:schemas-microsoft-com:asm.v1">
  <assemblyIdentity type="win32" name="..." version="6.0.0.0"/>
  <application>
    <windowsSettings>
      <activeCodePage xmlns="http://schemas.microsoft.com/SMI/2019/WindowsSettings">UTF-8</activeCodePage>
    </windowsSettings>
  </application>
</assembly>
```

Save this mainfest file as `UTF-8.manifest`:
Run:

```
mt.exe -manifest UTF-8.manifest -outputresource:clipboard.exe;#1
```

