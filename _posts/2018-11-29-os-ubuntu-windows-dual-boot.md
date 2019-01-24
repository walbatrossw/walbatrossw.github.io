---
layout: article
title: 우분투 & 윈도우 듀얼 부팅시 세팅해야할 것들
category: ubuntu
tag: ubuntu
key: 20181129b
---


## 1. 우분투 & 윈도우 듀얼 부팅 시간 설정

우분투 윈도우 듀얼부팅 환경으로 설정하고, 윈도우로 부팅하면 시간이 맞지 않는 경우가
발생할 경우 아래와 같이 설정해준다.

### 1.1 해결 방법

```shell
timedatectl set-local-rtc 1
```

```shell
sudo gedit /etc/default/rcS
```

```shell
UTC=no
```

- 출처 : https://tuwlab.com/ece/28472

## 2. 우분투 & 윈도우 듀얼부팅 순서 변경하기

### 2.1 grub 설정파일 편집

```shell
sudo gedit /etc/default/grub
```

### 2.2 수정

`GRUB_DEFAULT=0` 값을 변경, 보통 4번의 윈도우가 위치하기 때문에 4번으로 변경

### 2.3 부팅 진입시간 변경

`GRUB_TIMEOUT=10` 변경

### 2.4 변경 사항 갱신

```shell
sudo update-grub
```
