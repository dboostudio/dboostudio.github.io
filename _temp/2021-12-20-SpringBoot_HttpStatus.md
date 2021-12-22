---
layout: post
author: Dboo
categories: SPRING_BOOT
title: SpringBoot 에서 기본적으로 제공하는 HttpStatus Code 정리
tags: SpringBoot HttpStatus
---

자주쓸거 같거나, 혹은 숙지해두고 싶은 코드만 정리했다.

- 1xx : Informational

|Value|Series|
|-|-|
|101|CONTINUE|
|102|SWITCHING_PROTOCOLS|
|103|PROCESSING|
|104|CHECKPOINT|

- 2xx : Success

|Value|Series|
|-|-|
|200|OK|
|201|CREATED|
|208|ALREADY_REPORTED|

- 3xx : Redirection

|Value|Series|
|-|-|
|303|SEE_OTHER|

- 4xx : Client Error

|Value|Series|
|-|-|
|400|BAD_REQUEST|
|401|UNAUTHORIZED|
|403|FORBIDDEN|
|404|NOT_FOUND|
|405|METHOD_NOT_ALLOWED|
|408|REQUEST_TIMEOUT|
|413|PAYLOAD_TOO_LARGE|
|414|URI_TOO_LONG|

- 5xx : Server Error

|Value|Series|
|-|-|
|500|INTERNAL_SERVER_ERROR|
|502|BAD_GATEWAY|
