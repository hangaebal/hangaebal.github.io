---
layout: post
title:  "API Hour Tutorial 1"
date:   2016-07-28 15:26:00 +0900
---



# API Hour Tutorial 1 (All-in-one script)

> *http://pythonhosted.org/api_hour/tutorials/all_in_one.html 를 요약 번역*

> API-Hour를 설치하지 않았다면 먼저 설치 해야 한다.


-----

## all_in_one.py 파일 작성

```python
import logging

import api_hour
import aiohttp.web
from aiohttp.web import Response

logging.basicConfig(level=logging.INFO)  # enable logging for api_hour


class Container(api_hour.Container):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Declare HTTP server
        self.servers['http'] = aiohttp.web.Application(loop=kwargs['loop'])
        self.servers['http'].ah_container = self  # keep a reference in HTTP server to Container

        # Define HTTP routes
        self.servers['http'].router.add_route('GET',
                                              '/',
                                              self.index)

    # A HTTP handler example
    # More documentation: http://aiohttp.readthedocs.org/en/latest/web.html#handler
    async def index(self, request):
        message = 'Hello World !'
        return Response(text=message)


    # Container methods
    async def start(self):
        # A coroutine called when the Container is started
        await super().start()


    async def stop(self):
        # A coroutine called when the Container is stopped
        await super().stop()


    def make_servers(self):
        # This method is used by api_hour command line to bind your HTTP server on socket
        return [self.servers['http'].make_handler(logger=self.worker.log,
                                                  keep_alive=self.worker.cfg.keepalive,
                                                  access_log=self.worker.log.access_log,
                                                  access_log_format=self.worker.cfg.access_log_format)]
```

## 구동

  - all_in_one.py 파일이 있는 디렉토리에서 아래 명령어로 구동
  
  ```bash
  $ api_hour all_in_one:Container
  ```

  - http://127.0.0.1:8000/ 접속 확인

## 워커 숫자에 따른 벤치마크

  - 아래 명령어로 16 워커로 구동
  
  ```bash
  $ api_hour -w 16 all_in_one:Container
  ```
  
  - [`wrk`](https://github.com/wg/wrk)를 활용해서 단일 워커와 비교 테스트
  
  ```bash
  $ wrk -t12 -c400 -d30s http://127.0.0.1:8000/
  ```

