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
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(1,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(691,228): error C2001: newline in constant [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(692,5): error C2062: type 'unsigned int' unexpected [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(696,13): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(697,113): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(702,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(718,13): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(719,113): error C2065: 'percent_done': undeclared identifier [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  messages.cpp
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(1,1): warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  windows.cpp
```

#2 Try:

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
index 8cd0b3e..47afc82 100644
--- a/src/clipboard/CMakeLists.txt
+++ b/src/clipboard/CMakeLists.txt
@@ -16,6 +16,7 @@ target_link_libraries(clipboard gui)
 
 if(WIN32)
   target_sources(clipboard PRIVATE src/windows.cpp)
+  target_compile_options(clipboard PRIVATE /source-charset:utf-8)
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
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(113,34): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  messages.cpp
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(87,51): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(88,52): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(89,52): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(90,50): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(96,42): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(97,42): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(105,48): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(107,36): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(108,38): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(109,41): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(110,45): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(111,51): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(112,57): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(113,63): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(158,31): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(159,39): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(160,37): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(161,31): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(162,35): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(165,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(166,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(174,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(175,38): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(176,44): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(177,50): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(207,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(207,20): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(208,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(210,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(210,20): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(212,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(212,20): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(213,31): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(213,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(214,37): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(214,37): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(214,37): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(216,38): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(217,36): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(218,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(219,32): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(219,32): warning C4566: character represented by universal-character-name '\u00F4' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(222,27): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(224,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(225,38): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(226,44): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(227,50): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(249,34): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(250,35): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(250,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(251,36): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(251,36): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(251,36): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(252,37): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(252,37): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(252,37): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(255,32): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(256,33): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(256,33): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(257,34): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(257,34): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(257,34): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(258,35): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(258,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(258,35): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(260,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(260,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(260,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(261,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(262,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(262,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(263,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(263,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(264,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(264,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(265,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(265,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(266,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(266,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,20): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(268,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(268,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(269,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(269,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(270,20): warning C4566: character represented by universal-character-name '\u00D6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(272,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(272,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(273,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(273,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(274,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(274,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(274,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(276,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(276,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(276,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(278,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(280,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(280,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u00C7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,38): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,38): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,38): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,38): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,38): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(283,39): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(283,39): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(283,39): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(284,37): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(284,37): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(284,37): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(285,31): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(285,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(285,31): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(285,31): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,31): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,31): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,31): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,35): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,35): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,35): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,38): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,38): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,38): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,38): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,36): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,36): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,36): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,29): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,29): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(291,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,32): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,32): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,32): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,32): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,32): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,27): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,27): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,27): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,34): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,34): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,34): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,34): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,34): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(299,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(300,38): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(301,44): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(302,50): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,30): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,30): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,30): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,30): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  windows.cpp
  Generating code
  Not all modules are compiled with -Gy (function comdat), build without incremental LTCG.
  Finished generating code
  clipboard.vcxproj -> C:\Users\hp\Desktop\Clipboard\build\clipboard.exe
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/CMakeLists.txt
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

