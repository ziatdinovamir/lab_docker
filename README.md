## Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере **CPack**

```sh
$ open https://cmake.org/Wiki/CMake:CPackPackageGenerators
```

## Tutorial

Создаем переменные окружения
```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GITHUB_EMAIL=<адрес_почтового_ящика>
$ alias edit=<nano|vi|vim|subl>
$ alias gsed=sed # for *-nix system
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
Клонируем репозиторий лабораторной работы №5 в репозиторий лабораторной работы №6
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab05 projects/lab06
$ cd projects/lab06
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
```
__Добавление версионирования в `CMakeLists.txt`__

После строки `project(print)` добавляет переменную `PRINT_VERSION_STRING` со значением версии
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
```
обавляет переменную `PRINT_VERSION`, которая собирает версию из четырёх компонентов: `major`, `minor`, `patch`, `tweak`
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
```
Добавляет переменную `PRINT_VERSION_TWEAK` со значением `0`
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
```
Добавляет переменную `PRINT_VERSION_PATCH` со значением `0`
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
```
Добавляет переменную `PRINT_VERSION_MINOR` со значением `1`
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
```
 Добавляет переменную `PRINT_VERSION_MAJOR` со значением `0`
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt
```
Показывает все изменения, которые вы сделали в файлах
```sh
$ git diff
```
```sh
diff --git a/CMakeLists.txt b/CMakeLists.txt
index ec70034..d73a3ae 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -1,5 +1,13 @@
 cmake_minimum_required(VERSION 3.14)
 project(print)
+set(PRINT_VERSION_MAJOR 0)
+set(PRINT_VERSION_MINOR 1)
+set(PRINT_VERSION_PATCH 0)
+set(PRINT_VERSION_TWEAK 0)
+set(PRINT_VERSION
+  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
+set(PRINT_VERSION_STRING "v${PRINT_VERSION}")
+set(PRINT_VERSION_STRING "v${PRINT_VERSION}")
 
 set(CMAKE_CXX_STANDARD 17)
 set(CMAKE_CXX_STANDARD_REQUIRED ON)
```
__Создание файлов для пакетов__
```sh
$ touch DESCRIPTION && edit DESCRIPTION
$ touch ChangeLog.md
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```
__Создание CPack конфигурации__
```sh
$ cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
```

```sh
$ cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```

```sh
$ gsed -i 's/lab05/lab06/g' README.md
```
__Создаём тег с версией v0.1.0.0 для текущего коммита и отправляем изменения и теги в удалённый репозиторий__
```sh
$ git add .
$ git commit -m"added cpack config"
$ git tag v0.1.0.0
$ git push origin master --tags
```
__Создаем и собираем проект__
```sh
mkdir -p .github/workflows

cat > .github/workflows/ci.yml <<'EOF'
name: CI Build and Package

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        include:
          - os: ubuntu-latest
            generator: "Unix Makefiles"
            cpack_generator: "TGZ"
            deps: "sudo apt-get update && sudo apt-get install -y cmake build-essential"
          - os: windows-latest
            generator: "Visual Studio 17 2022"
            cpack_generator: "ZIP"
            deps: "choco install cmake --installargs 'ADD_CMAKE_TO_PATH=System' || echo 'CMake already installed'"
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Install dependencies (${{ matrix.os }})
      run: ${{ matrix.deps }}
      shell: bash
    
    - name: Configure CMake
      run: cmake -H. -B_build -DCPACK_GENERATOR="${{ matrix.cpack_generator }}"
      shell: bash
    
    - name: Build
      run: cmake --build _build --config Release
      shell: bash
    
    - name: Create package
      run: cmake --build _build --target package --config Release
      shell: bash
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: packages-${{ matrix.os }}
        path: |
          _build/*.tar.gz
          _build/*.zip
          _build/*.exe
          _build/*.deb
    
    - name: Create Release
      if: startsWith(github.ref, 'refs/tags/') && matrix.os == 'ubuntu-latest'
      uses: softprops/action-gh-release@v1
      with:
        files: |
          _build/*.tar.gz
          _build/*.zip
          _build/*.exe
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```
```sh
$ cmake -H. -B_build
```
```sh
cmake -H. -B_build
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/amir/ziatdinovamir/workspace/projects/lab06/_build
```
```sh
$ cmake --build _build
```
```sh
cmake --build _build
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
```
```sh
$ cd _build
$ cpack -G "TGZ"
```
```sh
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/amir/ziatdinovamir/workspace/projects/lab06/_build/print-0.1.0.0-Linux.tar.gz generated.
```
```sh
$ cd ..
```
```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
```
```sh
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/amir/ziatdinovamir/workspace/projects/lab06/_build
```
```sh
$ cmake --build _build --target package
```
```sh
[100%] Built target print
Run CPack packaging tool...
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/amir/ziatdinovamir/workspace/projects/lab06/_build/print-0.1.0.0-Linux.tar.gz generated.
```
__Сохраняем артефакты__
```sh
$ mkdir artifacts
$ mv _build/*.tar.gz artifacts
$ tree artifacts
```
```sh
tree artifacts
artifacts
└── print-0.1.0.0-Linux.tar.gz
```

## Report

```sh
$ popd
$ export LAB_NUMBER=06
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)

__Создание CPackConfig.cmake__
```sh
cat > CPackConfig.cmake <<'EOF'
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT "ziatdinov.amir.07@gmail.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Amir Ziatdinov <ziatdinov.amir.07@gmail.com>")

set(CPACK_PACKAGE_VERSION_MAJOR 0)
set(CPACK_PACKAGE_VERSION_MINOR 1)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(CPACK_PACKAGE_VERSION_TWEAK 0)
set(CPACK_PACKAGE_VERSION "0.1.0.0")

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Solver application - solves mathematical equations")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)

set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_PACKAGE_NAME "solver")

set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6, libstdc++6")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_RELEASE 1)

include(CPack)
EOF
```
__Создание DESCRIPTION файла__
```sh
cat > DESCRIPTION <<'EOF'
Solver application for mathematical equations.

Package includes:
- solver executable
- formatter library
- examples and tests

Platforms: Ubuntu Linux, Windows
EOF
```
__Подключение CPack в CMakeLists.txt__
```sh
echo "" >> CMakeLists.txt
echo "include(CPackConfig.cmake)" >> CMakeLists.txt
```
__Создание .gitignore__
```sh
сat > .gitignore <<'EOF'
_build/
build/
artifacts/
_test_package/

*.tar.gz
*.deb
*.rpm
*.zip
*.msi
*.exe
*.dmg

.vscode/
.idea/
*.swp
*.swo
*~

.DS_Store
Thumbs.db
EOF
```
__Создание GitHub Actions workflow__
```sh
mkdir -p .github/workflows

cat > .github/workflows/ci.yml <<'EOF'
name: CI Build and Package

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: ${{ matrix.os }} - ${{ matrix.build_type }}
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        build_type: [Release, Debug]
        include:
          - os: ubuntu-latest
            build_type: Release
            cpack_generator: "TGZ;DEB"
          - os: ubuntu-latest
            build_type: Debug
            cpack_generator: "TGZ"
          - os: windows-latest
            build_type: Release
            cpack_generator: "ZIP"
          - os: windows-latest
            build_type: Debug
            cpack_generator: "ZIP"
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Setup Ubuntu
      if: matrix.os == 'ubuntu-latest'
      run: |
        sudo apt-get update
        sudo apt-get install -y cmake build-essential rpm
      shell: bash
    
    - name: Setup Windows
      if: matrix.os == 'windows-latest'
      run: |
        choco install cmake --installargs 'ADD_CMAKE_TO_PATH=System' || echo "CMake already installed"
      shell: powershell
    
    - name: Configure CMake
      run: cmake -H. -B_build -DCMAKE_BUILD_TYPE=${{ matrix.build_type }}
      shell: bash
    
    - name: Build
      run: cmake --build _build --config ${{ matrix.build_type }}
      shell: bash
    
    - name: Create packages
      run: |
        cd _build
        cpack -G "${{ matrix.cpack_generator }}"
      shell: bash
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: ${{ matrix.os }}-${{ matrix.build_type }}
        path: |
          _build/*.tar.gz
          _build/*.deb
          _build/*.rpm
          _build/*.zip
      continue-on-error: true
    
    - name: Create Release
      if: startsWith(github.ref, 'refs/tags/') && matrix.os == 'ubuntu-latest' && matrix.build_type == 'Release'
      uses: softprops/action-gh-release@v1
      with:
        files: |
          _build/*.tar.gz
          _build/*.deb
          _build/*.rpm
          _build/*.zip
        generate_release_notes: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```
__Добавление, коммит и отправка файлов на GitHub__
```sh
git add .
git commit -m "Complete lab06"
git push origin main
```
__Создание и отправка тега для релиза__
```sh
git tag -a v1.0.0 -m "Release v1.0.0: solver application"
git push origin main --tags
```
__Локальная сборка и проверка пакетов__
```sh
cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release
```
```sh
-- The C compiler identification is GNU 13.3.0
-- The CXX compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (0.4s)
-- Generating done (0.0s)
-- Build files have been written to: /home/amir/ziatdinovamir/workspace/projects/lab06/_build
```
```sh
cmake --build _build
```
```sh
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
```
```sh
cd _build
cpack -G "TGZ"
```
```sh
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/amir/ziatdinovamir/workspace/projects/lab06/_build/solver-0.1.0.0-Linux.tar.gz generated.
```
```sh
cpack -G "DEB"
```
```sh
CPack: Create package using DEB
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/amir/ziatdinovamir/workspace/projects/lab06/_build/solver-0.1.0.0-Linux.deb generated.
-rw-rw-r-- 1 amir amir 678 Apr 14 12:54 _build/solver-0.1.0.0-Linux.deb
-rw-rw-r-- 1 amir amir  29 Apr 14 12:54 _build/solver-0.1.0.0-Linux.tar.gz
```
```sh
cd ..
```
```sh
tree artifacts
```
```sh
artifacts
├── solver-0.1.0.0-Linux.deb
└── solver-0.1.0.0-Linux.tar.gz

1 directory, 2 files
```
# lab_docker
