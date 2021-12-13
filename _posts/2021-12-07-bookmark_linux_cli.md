---
layout: post
author: Dboo
categories: Bookmarks
title: 자주쓰는 리눅스 명령어 모음
tags: Linux Command
---

### 기초 명령어

`cp <copy-file-path> <paste-file-path>`

### SSH 접속

`ssh <user>@<ip> -p <port>`
`ssh -i <pem-file-location> <user>@<ip> -p <port>`

### SSH로 파일 복사

`scp -i <pem-file-location> <copy-file-path> <user>@<ip>:<paste-path>`

### 기타

`free` : 메모리 체크
`df` : 용량 체크
