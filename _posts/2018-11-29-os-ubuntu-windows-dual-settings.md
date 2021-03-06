---
layout: article
title: 우분투 18.04 & 윈도우10 듀얼 부팅 설치 및 설정
date: 2018-11-29 00:00:00 # 작성 시간 2018-11-08 00:00:00
category: ubuntu
tags: ubuntu
key: 20181129b
---


## 1. 설치 준비

- 디스크 관리에서 Ubuntu 설치를 위한 파티션 준비
- UEFI 펌웨어일 경우 빠른 시작 해제

## 2. 우분투 설치

### 2.1 윈도우 듀얼 부팅으로 설치
- 기타 설치 선택

### 2.2 파티션 설정
- SWAP 영역 : 램크기 만큼 설정
- /(루트) 영역 : 원하는 크기 만큼 설정
- /home 영역 : 원하는 크기 만큼 설정

### 2.3 지역 설정
- 서울 선택

### 2.4 언어 선택
- 한국어 / 104키보드 설정

## 3. 설치 완료 후 우분투 설정 및 개발 환경 세팅

### 3.1 업데이트 확인

### 3.2 터미널 설정하기

- 터미널 크기 조정
- 색 : 투명도 조정
- 스크롤 막대 표시 제거

### 3.3 GDebi 설치 : deb설치 파일 인스톨을 위해

```shell
sudo apt-get install gdebi
```

### 3.4 Chrome 설치

- [크롬 다운로드](https://www.google.co.kr/chrome/index.html)
- gdebi를 이용하여 설치

### 3.5 KDE Desktop 설치 (선택사항)

```shell
sudo apt -y install kubuntu-desktop
```

### 3.6 Gnome Tweak Tool, Gnome Extensions 설치 : 테마 및 유틸 설치를 위해

- Unity Tweak은 18.04부터 없어짐

```shell
sudo apt-get install gnome-tweak-tool
```

```shell
sudo apt install chrome-gnome-shell
```

- Extensions(https://extensions.gnome.org/) 설치
  - Arc Menu : 윈도우 버튼같은 역할
  - Coverflow Alt-tab : Alt-tab 3D 화면전환
  - Dash to Dock : Dock 설정
  - OpenWeather : 날씨
  - User Themes : 우분투 테마

### 3.7 한글 입력기 nimf 설치

```shell
sudo add-apt-repository ppa:hodong/nimf
sudo apt-get update
sudo apt install nimf nimf-libhangul
```

- 키보드 한글 키 설정 방법 : 설정 > 기능개선 > 키보드와 마우스 > 추가 배치 옵션 이동하여 수정

### 3.8 우분투 테마 및 아이콘 설치

- [best 10 우분투 테마](https://www.omgubuntu.co.uk/2017/11/best-gtk-themes-for-ubuntu)
- [paper icon theme](https://snwh.org/paper/download)

### 3.9 우분투 소프트웨어를 통해 유틸리티 및 개발도구 설치
- VLC : 동영상 플레이어
- GIMP : 이미지 편집기
- Pinta : 윈도우 그림판과 유사한 기능
- FileZilla
- Putty
- IntelliJ : IDE
- Atom : Script Editor
- GitKraken : Git Gui

### 3.10 Tomcat 설치

- [Tomcat 공식 홈페이지](http://tomcat.apache.org/)
- Tomcat을 다운받아 workspace에 버전별로 세팅


### 3.11 Git 설치

```shell
sudo apt-get install git
```

### 3.12 Java 설치 및 환경변수 설정

#### 3.12.1 Java 설치

```shell
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

#### 3.12.2 환경변수 설정

```shell
sudo gedit /etc/profile
```

```shell
# 환경변수
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib:$CLASS_PATH
```

```shell
source /etc/profile
```

### 3.13 Mysql 5.7 설치 및 한글 인코딩 설정

#### 3.13.1 Mysql, Workbench 설치

```shell
sudo apt-get install mysql-server-5.7
sudo apt-get install mysql-workbench
```

#### 3.13.2 Root 비밀번호 설정

```shell
sudo mysql
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_new_password';
```

[출처 : stackoverflow](https://stackoverflow.com/questions/25777943/failed-to-connect-to-mysql-at-127-0-0-13306-with-user-root-access-denied-for-us)

#### 3.13.3 한글 인코딩 설정

```shell
sudo gedit /etc/mysql/my.cnf
```

```
[client]
default-character-set = utf8

[mysqld]
character-set-client-handshake=FALSE
init_connect="SET collation_connection = utf8_general_ci"
init_connect="SET NAMES utf8"
character-set-server = utf8
collation-server = utf8_general_ci

[mysqldump]
default-character-set = utf8

[mysql]
default-character-set = utf8
```

```shell
sudo service mysql restart
```


#### 3.14 개발용 폰트 설치

- [D2](https://github.com/naver/d2codingfont)
- [Nanum Gothic Coding](https://github.com/naver/nanumfont/blob/master/README.md)
- [Hack](https://sourcefoundry.org/hack/)


**참조 블로그**

- [http://programmingsummaries.tistory.com/389](http://programmingsummaries.tistory.com/389)
- [http://jimnong.tistory.com/694?category=575588](http://jimnong.tistory.com/694?category=575588)
- [http://webdir.tistory.com/217](http://webdir.tistory.com/217)

---
