<!--
title:   cmakeを使って、clang-tidyやclang-formatを行う
tags:    C,C++,CMake,clang-format,clang-tidy
id:      f8548ee9bab78f18cd25
private: false
-->
C/C++でコードを書く場合、`clang-tidy`や`clang-format`はとても便利です。
CMakeで、任意のターゲット[^1]に対して、`clang-tidy`や`clang-format`を行えるようにしました。

[^1]: グローバルに設定したら、ExternalProject(googletest)でえらいことになってしまった

```cmake:FormatFilesWithClangFormat.cmake
option(FORMAT_FILES_WITH_CLANG_FORMAT_BEFORE_EACH_BUILD
  "If the command clang-format is avilable, format source files before each build.\
Turn this off if the build time is too slow."
  ON)
find_program(CLANG_FORMAT_EXE clang-format)

function(clang_format target)
  if(CLANG_FORMAT_EXE)
    message(STATUS "Enable Clang-Format ${target}")
    get_target_property(MY_SOURCES ${target} SOURCES)
    add_custom_target(
      "${target}_format-with-clang-format"
      COMMAND "${CLANG_FORMAT_EXE}" -i -style=file ${MY_SOURCES}
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
      )
    if(FORMAT_FILES_WITH_CLANG_FORMAT_BEFORE_EACH_BUILD)
      add_dependencies(${target} "${target}_format-with-clang-format")
    endif()
  endif()
endfunction()
```

```cmake:DoClangTidy.cmake
cmake_minimum_required(VERSION 3.6)
option(CLANG_TIDY_ENABLE
  "If the command clang-tidy is avilable, tidy source files.\
Turn this off if the build time is too slow."
  ON)
find_program(CLANG_TIDY_EXE clang-tidy)

function(clang_tidy target)
  if(CLANG_TIDY_EXE)
    if(CLANG_TIDY_ENABLE)
      message(STATUS "Enable Clang-Tidy ${target}")
      set_target_properties(${target} PROPERTIES
        C_CLANG_TIDY "${CLANG_TIDY_EXE};-fix;-fix-errors"
        CXX_CLANG_TIDY "${CLANG_TIDY_EXE};-fix;-fix-errors")
      endif()
  endif()
endfunction()
```

```cmake:使い方
include(DoClangTidy)
include(FormatFilesWithClangFormat)

set(target nice_app)
add_executable(${target}
  main.cpp
)
clang_format(${target})
clang_tidy(${target})
```

## 解説
`FormatFilesWithClangFormat.cmake`/`DoClangTidy.cmake`を`include`します。
すると、関数`clang_tidy(target)`/`clang_format(target)`が定義されます。これを呼ぶとターゲットのビルド時に`clang-tidy`/`clang-format`が行われるようになります。
また、有効/無効化やコマンドパスを設定するオプションも設定されます。`ccmake`等で編集できます。

チェック内容やインデント設定等は、上記cmakeファイルを（コマンド引数に足す）のではなく、`.clang-tidy`/`.clang-format`ファイルで設定するのがおすすめです。

## 参考

- [冬休み到来! clang-tidy で安心安全な C/C++ コーディングを極めよう! - Qiita](/syoyo/items/0e75410c44ed73d4bdd7)
- [CMakeでビルドしているコードにclang-tidyを実行する - Qiita](/yoyomion/items/8ff1f5a63b4b4f757732)
- [C++コードのデトックス - Qiita](https://qiita.com/MitsutakaTakeda/items/6b9966f890cc9b944d75)