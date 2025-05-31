---
layout: post
title: "C++ の環境構築"
categories: C++
---
# vscodeでC++コーディング環境を構築する

Ubuntu環境でC++コーディング環境を構築する方法メモ．メモ書き程度ですが，参考になれば幸いです．

## 1. 準備

そもそも単純なC++環境であれば，Linux系OSでの環境構築は簡単だと思います．単に標準ライブラリ(iostreamなど)を使う場合は，vscodeに以下の拡張機能があれば，不自由なくコンパイルから実行まで行けるはずです．

- C/C++(Microsoft)：シンタックスハイライトや文法ミスを指摘してくれる
- Code Runner(Jun Han)：vscode右上辺りに実行ボタンとか出現する

今回は，外部ライブラリ(boostなど)を使用したい場合を想定しています．vcpkgを用いてパッケージをインストールした場合を解説します．

vcpkgはMicrosoft社開発のパッケージマネージャで，簡単にライブラリをインストールして管理できます．これとCMakeを利用して，実行ファイルをBuildする方法を見ていきます．必要な拡張機能は，上記のものに加えて次の3点です：

- CMake Tools(Microsoft)：CMakeを使用してビルドする際に，vcpkgを通じてインストールしたライブラリを組み込める
- CMake(twxs)：CMakeファイル用のシンタックスハイライトなどの機能を提供します
- C++デバッグ系拡張機能：IDEのようなデバッグを行えるような拡張機能(調査中)

ひとまずこれら拡張機能をインストールしたら，vscodeを再起動しておきましょう．

```command
Ctrl + Shift + P
```

でコマンドパレットを起動して，"Developer: Reload Window"を選ぶと起動中のvscodeウィンドウが開き直されます(他にも起動しているなら，killなりした方がいいでしょう)．

## 2. プロジェクト用ディレクトリを作成する

まずはC++を用いたプログラミングを行うためのディレクトリを切りましょう．

```Bash
~$ mkdir cpp_Project
```

このディレクトリで"main.cpp"を作成し，vscodeでディレクトリごと開きましょう(ドラッグアンドドロップで行けるはず)．

その後，簡単に次のようなコーディングを行い，実行までがうまく行くことを確認しておきます：

```C++
#include <iostream>

using namespace std;

int main(){
    cout << "Hello_World!" << endl;
}
```

## 3. vcpkgの設定

まずはvcpkgのインストールを行います．

```Bash
git clone https://github.com/microsoft/vcpkg.git
```

ブートストラップのスクリプトを実行します．

```Bash
cd /your/vcpkg/path
./bootstrap-vcpkg.sh
```

PATH環境変数を設定し，vcpkgコマンドを使えるようにしましょう．永続化して問題ないなら「.bashrc」を再読込させます．

```Bash
export PATH=$PATH:/path/vcpkg
source ~/.bashrc
```

「.bashrc」にこのような記述があればよいです．

```Bash
# Add vcpkg settings
export VCPKG_ROOT=/path/vcpkg
export PATH=$VCPKG_ROOT:$PATH
```

## 4. vscode周りの設定

プロジェクトのディレクトリ「/cpp_prj」に対して，vscodeでCmakeや実行を行うための設定方法を紹介します．
なお，個々人の環境構築のスタイルがあるのであくまで一例であり，より効率的な管理方法もあるかと思います．

原則として，「.vscode」というvscodeの設定などのjsonファイルを保存しておくディレクトリがありますが，プロジェクト毎に作成するのが良いです．
フォントや文字サイズなど共通の設定などをする場合はホームディレクトリの「.vscode」へjsonファイルを置くこともありますが，基本的にはプロジェクト単位で「.vscode」ディレクトリを切るのが良いでしょう．

### task.json

ビルドコマンドの設定を行います．簡単に言えば「gcc main.c -o main」を行っているに近しいです．：

```json
{
    "version": "2.0.0", 
    "tasks": [
        {
            "label": "Build C++", 
            "type": "shell", 
            "command": "/usr/bin/clang", 
            "args": [
                "-g", 
                "${file}", 
                "-o", 
                "${fileDirname}/${fileBasenameNoExtension}",
                "-I", 
                "/home/yourProject/vcpkg_installed/x64-linux/include0", 
                "-L", 
                "/home/yourProject/vcpkg_installed/x64-linux/lib", 
                "-lyour_library_name"
            ], 
            "group": {
                "kind": "build",
                "isDefault": true
            }, 
            "problemMatcher": ["$gcc"]
        }
    ]
}

```

tasks.jsonの環境変数の一例：

| プレースホルダ | 説明 |
| ---- | ---- |
| ${workspaceFolder} | 現在のワークスペースのパス |
| ${userHame} | ユーザのホームフォルダパス |
| ${file} | 現在開いているファイルのフルパス：/path/to/main.cpp |
| ${fileDirname} | 現在開いているファイルのディレクトリ名 |
| ${fileBasename} | 現在のファイル名(拡張子含む) |
| ${fileBasenameNoExtension} | 現在のファイル名(拡張子を含まない) |
| ${pathSeparator} | ファイルパスの区切り文字 |
| ${/} | ${pathSeparator}の短縮形 |

### launch.json

デバッグ用のjsonです．MIModeでデバッガ(ここではgdb)を選びます．

```json
{
    "configurations": [
        {
            "name": "C++ Launch", 
            "type": "cppdbg",
            "request": "launch", 
            "program": "${workspaceRoot}/build/cpp_web_scraper", 
            "args": [], 
            "stopAtEntry": false, 
            "cwd": "${workspaceFolder}", 
            "environment": [], 
            "externalConsole": false, 
            "MIMode": "gdb", 
            "miDebuggerArgs": "/usr/bin/gdb", 
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb", 
                    "text": "-enable-pretty-printing", 
                    "ignoreFailures": true
                }
            ], 
            "preLaunchTask": "Build C++ with Clang"
        }
    ]
}

```

### c_cpp_properties.json

vscodeのc拡張機能で使用する設定ファイルです．includeファイルやどのコンパイラを使用するかを設定できます：

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "/home/yourProject/**",
                "/usr/include",
                "/home/yourProject/vcpkg_installed/x64-linux/include/**", 
                "${workspaceFolder}/**"
            ], 
            "defines": [],
            "compilerPath": "/usr/bin/clang++",
            "cStandard": "c17",
            "cppStandard": "c++14",
            "intelliSenseMode": "linux-clang-x64"
        }
    ],
    "version": 4
}

```

### setting.json

Cのプログラミング環境を整えるための設定で，エディタの設定なども行います．この辺は本当にお好みなので，調べていただいて設定すると良いでしょう．

```json
{
    "folders": [
        {
            "path": "/home/yourProject"
        }
    ], 
    "settings": {
        "cmake.configureSettings": {
            "CMAKE_TOOLCHAIN_FILE": "/home/yourProject/vcpkg/scripts/buildsystems/vcpkg.cmake"
        }, 
        "cmake.configureArgs": [
            "-DCMAKE_TOOLCHAIN_FILR=/home/yourProject/vcpkg/scripts/buildsystems/vcpkg.cmake"
        ]
    },
    "code-runner.executorMap": {
        "cpp": "/home/yourProject/build/cpp_web_scraper"
    }, 
    "C_Cpp.intelliSenseEngine": "default",
    "editor.formatOnSave": true,
    "C_Cpp.clang_format_style": "{BasedOnStyle: Google, IndentWidth: 4}",
    "code-runner.clearPreviousOutput": true, 
    "code-runner.saveFilesBeforeRun": true,
    "files.associations": {
        "*.ipp": "cpp",
        "cctype": "cpp",
        "clocale": "cpp",
        "cmath": "cpp",
        "csetjmp": "cpp",
        "csignal": "cpp",
        "cstdarg": "cpp",
        "cstddef": "cpp",
        "cstdio": "cpp",
        "cstdlib": "cpp",
        "cstring": "cpp",
        "ctime": "cpp",
        "cwchar": "cpp",
        "cwctype": "cpp",
        "any": "cpp",
        "array": "cpp",
        "atomic": "cpp",
        "strstream": "cpp",
        "bit": "cpp",
        "*.tcc": "cpp",
        "bitset": "cpp",
        "cfenv": "cpp",
        "charconv": "cpp",
        "chrono": "cpp",
        "cinttypes": "cpp",
        "codecvt": "cpp",
        "complex": "cpp",
        "condition_variable": "cpp",
        "cstdint": "cpp",
        "deque": "cpp",
        "forward_list": "cpp",
        "list": "cpp",
        "map": "cpp",
        "set": "cpp",
        "unordered_map": "cpp",
        "unordered_set": "cpp",
        "vector": "cpp",
        "exception": "cpp",
        "algorithm": "cpp",
        "functional": "cpp",
        "iterator": "cpp",
        "memory": "cpp",
        "memory_resource": "cpp",
        "numeric": "cpp",
        "optional": "cpp",
        "random": "cpp",
        "ratio": "cpp",
        "regex": "cpp",
        "source_location": "cpp",
        "string": "cpp",
        "string_view": "cpp",
        "system_error": "cpp",
        "tuple": "cpp",
        "type_traits": "cpp",
        "utility": "cpp",
        "rope": "cpp",
        "slist": "cpp",
        "fstream": "cpp",
        "future": "cpp",
        "initializer_list": "cpp",
        "iomanip": "cpp",
        "iosfwd": "cpp",
        "iostream": "cpp",
        "istream": "cpp",
        "limits": "cpp",
        "mutex": "cpp",
        "new": "cpp",
        "ostream": "cpp",
        "scoped_allocator": "cpp",
        "shared_mutex": "cpp",
        "sstream": "cpp",
        "stdexcept": "cpp",
        "streambuf": "cpp",
        "thread": "cpp",
        "typeindex": "cpp",
        "typeinfo": "cpp",
        "valarray": "cpp",
        "variant": "cpp"
    }
}

```

## 5. Cmake関連

Cmake関連，マジでわからん．

とりあえずうまくいった例を示すが，基本これで何とかなるのではないか：

```cmake
cmake_minimum_required(VERSION 3.1.0)
project(cpp_web_scraper VERSION 0.1.0)

set(CMAKE_TOOLCHAIN_FILE "/home/yourProject/vcpkg/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_PREFIX_PATH "/home/yourProject/vcpkg_installed/x64-linux")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# assign include directory of project
include_directories(/home/yourProject/vcpkg_installed/x64-linux)

link_directories(/home/yourProject/vcpkg_installed/x64-linux)

find_package(fmt REQUIRED)
find_package(cpr REQUIRED)
find_package(LibXml2 REQUIRED)
find_package(spdlog REQUIRED)

# add source file of project
add_executable(cpp_web_scraper main.cpp)
target_compile_features(cpp_web_scraper PRIVATE cxx_std_20)
target_compile_definitions(cpp_web_scraper PRIVATE LIBXML_STATIC)

# add link of library

target_link_libraries(cpp_web_scraper PRIVATE fmt::fmt)

target_link_libraries(cpp_web_scraper PRIVATE cpr::cpr)

target_link_libraries(cpp_web_scraper PRIVATE LibXml2::LibXml2)

target_link_libraries(cpp_web_scraper PRIVATE spdlog::spdlog)

target_include_directories(cpp_web_scraper PRIVATE /home/yourProject/vcpkg_installed/x64-linux/include)
target_link_directories(cpp_web_scraper PRIVATE /home/yourProject/vcpkg_installed/x64-linux/lib)

```

なお，vcpkgはおそらくマニフェストモードになっているかと思いますので，単純に

```Bash
vcpkg install packageName
```

だとエラーになると思います．
そのため，「vcpkg.json」にインストールしたいc++のパッケージを記述します：

```json
{
  "dependencies": [
    "cpr",
    "fmt",
    "libxml2",
    "spdlog"
  ]
}
```

これでインストールしたいライブラリを記述した後，

```Bash
vcpkg install
```

でよい．

### 補足

多分vcpkgでインストールしたライブラリ群は，「.h」のようにヘッダファイルのみのヘッダオンリライブラリになっていることが多いと思います．「.cpp」のようなソースファイルが無いので簡単にセットアップできて，かつコンパイル時にインライン化とかしてくれます．それに抽象化のためにテンプレートを使っていることも多いので，ヘッダファイル内で完結させている可能性があります．とにかく便利ってことです．

## 6. 自作のクラスを追加したい

これまでインストールしてきたライブラリを使って新しく何らかのクラスを作成した時，どのように自身のvscode環境に追加するか．
正直vcpkgだのcmakeだのよくわからないものを使用する都合上，どうしてもファイルがとっちらかってしまいます．
なるべく綺麗に管理するために，tasks.jsonなどの修正は必ず追える様に，gitなどで管理すると良いでしょう(してなくて地獄を見た)．

まずクラスファイルはヘッダファイル(.hpp)と実装ファイル(.cpp)に別れます．
非常に簡単に言えば，「CMake Build」コマンドでうまくコンパイルが通れば良いわけで，基をたどれば

```Bash
clang -g main.cpp myclass.cpp -o main -L ...
```

と書いてコンパイルが通れば良いわけです(そんなに簡単な問題ではないことは，vcpkgなどよくわかんない管理ソフトを導入する弊害でもある)．

さて，例えばmain.cppで愚直に実装した処理をクラスを用いて，オブジェクト指向的に正しく実装しましょうとなったとします．
このとき，既存のコンパイル環境がうまく動作するのであれば，問題は次の2つに分離できます：

- CMakeにおけるコンパイルコマンドが不正
- ファイル同士のリンクの不整合

ぶっちゃけvscode上でのCMakeはややこしいですが，難しくはないです．
例として，「main.cpp，scraper.cpp，scraper.hpp」というmain.cppにScraperというクラスを導入する方法を考えます．
触るのは主に2つ：

tasks.json

※正直argsの-g以降にscraper.cppを置く意味があるかは現状ちょっと不明

```json
{
    "version": "2.0.0", 
    "tasks": [
        {
            "label": "Build C++", 
            "type": "shell", 
            "command": "/usr/bin/clang", 
            "args": [
                "-g", 
                "scraper.cpp",
                "${file}", 
                "-o", 
                "${fileDirname}/${fileBasenameNoExtension}",
                "-I", 
                "/home/yourProject/vcpkg_installed/x64-linux/include0", 
                "-L", 
                "/home/yourProject/vcpkg_installed/x64-linux/lib", 
                "-lyour_library_name"
            ], 
            "group": {
                "kind": "build",
                "isDefault": true
            }, 
            "problemMatcher": ["$gcc"]
        }
    ]
}

```

CMakeLists.txt

```CMake
cmake_minimum_required(VERSION 3.1.0)
project(cpp_web_scraper VERSION 0.1.0)

set(CMAKE_TOOLCHAIN_FILE "/home/yourProject/vcpkg/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_PREFIX_PATH "/home/yourProject/vcpkg_installed/x64-linux")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# assign include directory of project
include_directories(/home/yourProject/vcpkg_installed/x64-linux)

link_directories(/home/yourProject/vcpkg_installed/x64-linux)

find_package(fmt REQUIRED)
find_package(cpr REQUIRED)
find_package(LibXml2 REQUIRED)
find_package(spdlog REQUIRED)

# add source file of project
add_executable(cpp_web_scraper main.cpp scraper.cpp)    # ここ大事
target_compile_features(cpp_web_scraper PRIVATE cxx_std_20)
target_compile_definitions(cpp_web_scraper PRIVATE LIBXML_STATIC)

# add link of library

target_link_libraries(cpp_web_scraper PRIVATE fmt::fmt)

target_link_libraries(cpp_web_scraper PRIVATE cpr::cpr)

target_link_libraries(cpp_web_scraper PRIVATE LibXml2::LibXml2)

target_link_libraries(cpp_web_scraper PRIVATE spdlog::spdlog)

target_include_directories(cpp_web_scraper PRIVATE /home/yourProject/vcpkg_installed/x64-linux/include)
target_link_directories(cpp_web_scraper PRIVATE /home/yourProject/vcpkg_installed/x64-linux/lib)

```

要は

```Bash
clang -g main.cpp myclass.cpp -o main -L ...
```

をうまくvscodeで実行できるようにするわけです．

これに対して，「ファイル同士のリンクの不整合」は大変です．
「undefined reference to ...」というようなエラーは，上述のCMake関連のエラーを疑いますが，それ以外は地道にエラーメッセージを読んで解いていくしか無いです．私が遭遇した内容は次のとおりです：

- std::string STR でうまく定数化が出来なかった

これは単純にC++17環境だったこともあって直感的な定数化が出来ませんでした．解法としては，次のようにしました．string_viewで保持して，stringのコンストラクタでstring型にしました：

```C++
// scraper.hpp
static constexpr std::string_view USERAGENTS {"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safare/537.36"};

// scraper.cpp
std::string ua(USERAGENTS);
cpr::Header headers = {{"User-Agent", ua}};
```

- 型のキャスト

```C++
const std::string& inputExpr
```

のように宣言したinputExprを，メンバ変数のexprに入れます．しかしlibxmlライブラリのある型に合致しておらず，エラーが出現しました．
「xmlChar」にキャストするような処理でエラーが出現しましたが，これはxmlChar型は単に「unsigined char」なので，「this->expr.c_str()」でキャストして事なきを得ました．
