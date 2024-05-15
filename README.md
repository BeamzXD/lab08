## Answers Laboratory work VI

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)

Для этого нужно добавить ветвление в конфигурационные файлы для **CI** со следующей логикой:</br>
если **commit** помечен тэгом, то необходимо собрать пакеты (`DEB, RPM, WIX, DragNDrop, ...`) </br>
и разместить их на сервисе **GitHub**.

1. Создаем файл `CMakeLists.txt` для создания пакетов с бинарным файлом solver:
````
cmake_minimum_required(VERSION 3.4)
project(lab06)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/solver_application)

include(CPack.cmake)
````

2. Создаем файл `CPack.cmake`, на примере туториала:

````
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT makes.k.s@mail.ru)
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for solver")
set(CPACK_PACKAGE_VENDOR "BeamzXD")
set(CPACK_PACKAGE_PACK_NAME "solver-${PRINT_VERSION}")

set(CPACK_SOURCE_INSTALLED_DIRECTORIES 
	"${CMAKE_SOURCE_DIR}/solver_application; solver_application"
	"${CMAKE_SOURCE_DIR}/solver_lib; solver_lib"
	"${CMAKE_SOURCE_DIR}/formatter_ex_lib; formatter_ex_lib"
	"${CMAKE_SOURCE_DIR}/formatter_lib; formatter_lib")

set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "solves equations")

set(CPACK_GENERATOR "DEB;RPM")

set(CPACK_RPM_PACKAGE_SUMMARY "solves equations")

include(CPack)
````

3. В директории .github/workflows создаем два файла: 
- `CI.yml` для билдинга в Github Actions при каждом push или pull_request
````
name: CMake

on:
 push:
  branches: [main]
 pull_request:
  branches: [main]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Configure Solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build Solver
    run: cmake --build ${{github.workspace}}/build

````
- `CI_release.yml` для билдинга при каждом push нового тэга.
````
name: CMake

on:
 push:
   tags:
     - v*.*.*

jobs: 

  build_packages_Linux:

    runs-on: ubuntu-latest
    
    permissions:
      contents: write

    steps:
    - uses: actions/checkout@v3

    - name: Configure Solver
      run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

    - name: Build Solver
      run: cmake --build ${{github.workspace}}/build

    - name: Build package
      run: cmake --build ${{github.workspace}}/build --target package

    - name: Build source package
      run: cmake --build ${{github.workspace}}/build --target package_source

    - name: Make a release
      uses: ncipollo/release-action@v1.14.0
      with:
        artifacts: "build/*.deb,build/*.rpm,build/*.tar.gz,build/*.zip"
        token: ${{ secrets.GITHUB_TOKEN }}
````
Для того что бы создать тэги нужно использовать следующую команду:
````
git tag -a v*.*.* -m "v*.*.*"
````
И пушим его
````
git push origin v*.*.*
````
Все теги можно увидеть во вкладке `Tags`

```
Copyright (c) 2024 Lozanov Ilia IU8-25
```
