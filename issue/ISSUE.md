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

In clipboard.cpp and messages.cpp, there are many non-ASCII characters stored in message string literals(std::string_view). When it comes to CJK locales, MSVC will try to parse these string literals with the system's current code page(instead of UTF-8), resulting in these compilation errors.

At first, I wanted to change the type of these strings to std::u8string_view, but I gave up due to the amount of work(COLs) involved.

#2 Try:

Patch src/clipboard/CMakeLists.txt:

```diff
diff --git a/src/clipboard/CMakeLists.txt b/src/clipboard/CMakeLists.txt
index db6ec89..64d9ab9 100644
--- a/src/clipboard/CMakeLists.txt
+++ b/src/clipboard/CMakeLists.txt
@@ -22,6 +22,7 @@ target_link_libraries(clipboard gui)
 
 if(WIN32)
   target_sources(clipboard PRIVATE src/windows.cpp)
+  target_compile_options(clipboard PRIVATE /source-charset:utf-8)
 elseif(APPLE)
   enable_language(OBJC)
   target_sources(clipboard PRIVATE

```

In this way, I want the compiler to treat the two source files containing the UNICODE message strings as UTF-8 encoded.

This strategy worked, although it elicited a bunch of warning messages.

Build log:

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
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\clipboard.cpp(163,34): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  messages.cpp
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(96,42): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(97,47): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(98,48): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(99,52): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(100,43): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(101,41): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(107,33): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(108,33): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(117,39): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(119,27): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(120,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(121,32): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(122,36): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(123,38): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(124,44): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(125,50): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(126,53): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(127,52): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(128,55): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(173,31): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(174,44): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(175,37): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(176,31): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(177,35): warning C4566: character represented by universal-character-name '\u00F1' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(180,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(181,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(189,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(190,34): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(191,40): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(192,51): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(222,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(222,20): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(223,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(225,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(225,20): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(227,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(227,20): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(228,31): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(228,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(229,37): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(229,37): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(229,37): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(231,38): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(232,36): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(233,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(234,37): warning C4566: character represented by universal-character-name '\u00E3' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(234,37): warning C4566: character represented by universal-character-name '\u00F4' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(237,27): warning C4566: character represented by universal-character-name '\u00F5' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(239,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(240,34): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(241,40): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(242,51): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(264,34): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(265,35): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(265,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(266,36): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(266,36): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(266,36): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,37): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,37): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(267,37): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(270,32): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,33): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(271,33): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(272,34): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(272,34): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(272,34): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(273,35): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(273,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(273,35): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(275,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(276,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(277,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(277,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(278,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(278,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(279,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(279,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(280,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(280,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(281,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,20): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(282,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(283,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(283,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(284,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(284,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(285,20): warning C4566: character represented by universal-character-name '\u00D6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(286,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(287,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(288,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(289,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,20): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(290,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(291,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(291,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(291,20): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(293,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(295,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(295,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,20): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,20): warning C4566: character represented by universal-character-name '\u00C7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,20): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,20): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(296,20): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,38): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,38): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,38): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,38): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(297,38): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,44): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,44): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(298,44): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(299,37): warning C4566: character represented by universal-character-name '\u2022' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(299,37): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(299,37): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(300,31): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(300,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(300,31): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(300,31): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(301,31): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(301,31): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(301,31): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(301,31): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(302,35): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(302,35): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(302,35): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(302,35): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,38): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,38): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,38): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(303,38): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(304,36): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(304,36): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(304,36): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(305,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(305,29): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(305,29): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(306,29): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(308,37): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(308,37): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(308,37): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(308,37): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(308,37): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(311,27): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(311,27): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(312,27): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(313,34): warning C4566: character represented by universal-character-name '\u00F6' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(313,34): warning C4566: character represented by universal-character-name '\u011F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(313,34): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(313,34): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(313,34): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(314,28): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(315,34): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(316,40): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(317,51): warning C4566: character represented by universal-character-name '\u2713' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(318,30): warning C4566: character represented by universal-character-name '\u0130' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(318,30): warning C4566: character represented by universal-character-name '\u00E7' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(318,30): warning C4566: character represented by universal-character-name '\u015F' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
C:\Users\hp\Desktop\Clipboard\src\clipboard\src\messages.cpp(318,30): warning C4566: character represented by universal-character-name '\u0131' cannot be represented in the current code page (936) [C:\Users\hp\Desktop\Clipboard\build\src\clipboard\clipboard.vcxproj]
  windows.cpp
  Generating code
  Not all modules are compiled with -Gy (function comdat), build without incremental LTCG.
  Finished generating code
  clipboard.vcxproj -> C:\Users\hp\Desktop\Clipboard\build\clipboard.exe
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/CMakeLists.txt

```

However, the problem is that the path containing the CJK character and the localized error message returned by FormatMessageA are not displayed correctly when the program is run.

#3 Try:

Patch CMakeLists.txt:

```diff
diff --git a/src/clipboard/CMakeLists.txt b/src/clipboard/CMakeLists.txt
index db6ec89..1d4cd02 100644
--- a/src/clipboard/CMakeLists.txt
+++ b/src/clipboard/CMakeLists.txt
@@ -22,6 +22,7 @@ target_link_libraries(clipboard gui)
 
 if(WIN32)
   target_sources(clipboard PRIVATE src/windows.cpp)
+  target_compile_options(clipboard PRIVATE /utf-8)
 elseif(APPLE)
   enable_language(OBJC)
   target_sources(clipboard PRIVATE

```

This time, in addition to specifying the ``/source-charset:utf-8``, I also specified the ``/execution-charset:utf-8``.

https://learn.microsoft.com/en-us/cpp/build/reference/utf-8-set-source-and-executable-character-sets-to-utf-8?view=msvc-170

Build log:

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
  messages.cpp
  windows.cpp
  Generating code
  Not all modules are compiled with -Gy (function comdat), build without incremental LTCG.
  Finished generating code
  clipboard.vcxproj -> C:\Users\hp\Desktop\Clipboard\build\clipboard.exe
  Building Custom Rule C:/Users/hp/Desktop/Clipboard/CMakeLists.txt

```

No errors, no warnings.

This time, characters like `╳` and `▏` can show correctly. But error messages display is still a mass. The reason is that we specify the execution charset as UTF-8, but FormatMessageA still returns an error message with a native code page encoding.

There may be two ways to fix this issue:

Method 1:

Starting with Win10, Microsoft allows us to change the default encoding of of how Win32 APIs like FormatMessageA.

Add manifest file:

https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-utf8-code-page

https://stackoverflow.com/questions/6335352/how-can-i-embed-a-specific-manifest-file-in-a-windows-dll-with-a-cmake-build

There is no need to change executing code:

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

