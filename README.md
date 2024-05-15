## Answers Laboratory work VIII

## Homework

1. Клонируем репозиторий 6 лабаратрной и убираем из нее все файлы связанные с Cpack.

2. Создаем файл `Dockerfile`, на примере туториала:

````
FROM ubuntu:18.04
RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build

ENV LOG_PATH /home/logs/log.txt

VOLUME /home/logs

WORKDIR _install/bin

ENTRYPOINT ./demo
````

3. В директории .github/workflows создаем `CI.yml` файл: 
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

  - name: Install Docker
    run: sudo apt install runc containerd docker.io

  - name: Run Docker
    run: sudo docker build -t logger .

````
Проверяем `actions`
```
Copyright (c) 2024 Lozanov Ilia IU8-25
```
