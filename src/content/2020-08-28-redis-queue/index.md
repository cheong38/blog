---
draft: false
title: Redis 를 메세지 큐 (Message Queue) 로 사용하기 (feat. Bull)
author: [Woojin]
date: 2020-08-28
image: ./redis.png
imageAlt: Redis logo
tags: ['redis']
---

# 개요

Redis 는 일반적으로 데이터베이스 캐시와 메세지 브로커로 쓰이는 오픈소스로 된 인메모리 데이터 저장소이다.
Redis 에서는 pub/sub 기능도 제공하는데, [여기서](https://redis.io/topics/pubsub) 자세한 내용을 찾아볼 수 있다.
Redis pub/sub 의 주요한 특성 중 하나는 메세지가 구독하는 모든 클라이언트에게 보내진다는 것이다.
즉, 하나의 publish 에 대해서 여러 클라이언트가 실행될 수 있다는 의미이다.
혹은 RPOPLPUSH 등을 이용해 [Reliable Queue 패턴](https://redis.io/commands/rpoplpush#pattern-reliable-queue) 을 구현해야 하는데 너무 low-level 디테일을 관여해야 한다.
본 포스팅에서는 [`Bull`](https://www.npmjs.com/package/bull) 을 이용해 Redis 를 메세지 큐로 활용하는 법을 설명한다.

# Bull
`Bull` 은 Redis 기반의 큐를 제공하는 Nodejs 라이브러리이다.
새 버전인 [`BullMQ4`](https://github.com/taskforcesh/bullmq) 개발 진행 중이지만 아직 베타이고 star 수도 그렇게 높지 않아 본 포스팅에서는 `Bull` 로 진행한다.

`Bull`은 Redis 의 low-level 디테일을 감추고, Delayed jobs, Retries, Priority, Automatic recovery 등과 같은 기능을 제공함으로써 보다 신뢰도 높고 사용하기 편리하게 만들어준다.

# 실험

본 포스팅에서는 두 가지 실험을 진행한다.

1. 여러 클라이언트가 대기 중일 때, 오직 하나의 job 만 실행되는가

2. 클라이언트가 나중에 실행되어도 job 이 실행되는가

실험에 사용된 코드는 [github](https://github.com/cheong38/nest-bull)에서 확인할 수 있다.

## 사전 준비

- Nodejs v12.x.x
- [Yarn](https://yarnpkg.com/)
- [Nestjs](https://nestjs.com/)
- [Docker](https://www.docker.com/)

## 디렉토리 구조
```
.
├── consumer/
├── nest-producer/
└── producer/
```

## Consumer

먼저 AppModule 에 Redis 설정 값을 넣은 `Bull` 모듈을 import 해준다.

```typescript
// consumer/src/app.module.ts

import { BullModule } from '@nestjs/bull'
import { Module } from '@nestjs/common'

import { TestConsumer } from './test.consumer'

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'test',
      redis: {
        host: 'localhost',
        port: 6379
      }
    })
  ],
  controllers: [],
  providers: [TestConsumer],
})
export class AppModule {}

```

`test` 라는 이름의 큐에 메세지가 추가되면 `doJob()` 이라는 메서드가 실행되게 된다.

```typescript
// consumer/src/test.consumer.ts

import { Process, Processor } from '@nestjs/bull'
import { Job } from 'bull'

@Processor('test')
export class TestConsumer {
  @Process()
  async doJob(job: Job) {
    console.log('consumer', job.data)
  }
}

```

## Nest Producer

먼저 AppModule 에 Redis 설정 값을 넣은 `Bull` 모듈을 import 해준다.

```typescript
// nest-producer/src/app.module.ts

import { BullModule } from '@nestjs/bull'
import { Module } from '@nestjs/common';

import { AppController } from './app.controller';

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'test',
      redis: {
        host: 'localhost',
        port: 6379
      }
    })
  ],
  controllers: [AppController],
  providers: [],
})
export class AppModule {}
```

`AppController` 의 `getHello()` 메서드가 실행되면 `test` 큐에 `{ data: 1 }` 이라는 데이터와 함께 메세지를 추가한다.

```typescript
// nest-producer/src/app.controller.ts

import { InjectQueue } from '@nestjs/bull'
import { Controller, Get } from '@nestjs/common';
import { Queue } from 'bull'

@Controller()
export class AppController {
  constructor(
    @InjectQueue('test') private readonly testQueue: Queue
  ) {}

  @Get()
  async getHello(): Promise<string> {
    await this.testQueue.add({ data: 1 })
    return 'hello'
  }
}
```

## Producer (without nestjs)

Nestjs 없이 `Bull` 라이브러러만을 이용해서 `test` 큐에 job 을 추가한다.

```javascript
// producer/main.js
const Queue = require('bull')

const testQueue = new Queue(
  'test',
  {
    redis: {
      host: 'localhost',
      port: 6379
    }
  }
)

main()
  .then(() => {
    console.log('done')
    process.exit(0)
  })
  .catch((error) => {
    console.error(error)
    process.exit(1)
  })

async function main() {
  await testQueue.add({ data: 1 })
}

```

## Redis 실행 (with Docker)

```shell
$ docker run --rm -it -p 6379:6379 --name bull-redis redis
```

## 여러 클라이언트가 대기 중일 때, 오직 하나의 job 만 실행되는가

한 터미널에서 아래의 명령어를 실행한다.
```shell
$ cd consumer
$ yarn start:1
```

다른 터미널에서 아래의 명령어를 실행한다.
```shell
$ cd consumer
$ yarn start:2
```

이제 두 개의 consumer 가 `test` 큐에 메세지가 들어오면 처리할 수 있도록 대기하고 있다.

다른 터미널에서 아래의 명령어를 실행한다.

```shell
$ cd producer
$ yarn start
```

그러면 한 번에 오직 하나의 consumer 에서 log 가 출력되는 것을 확인할 수 있다.
즉, 메세지 하나 당 오직 하나의 클라이언트만 처리를 진행하게 된다.

## 클라이언트가 나중에 실행되어도 job 이 실행되는가

한 터미널에서 아래의 명령어를 실행한다.
```shell
$ cd producer
$ yarn start
```

그리고 또 다른 터미널에서 아래의 명령어를 실행한다.
```shell
$ cd consumer
$ yarn start:1
```

큐에 메세지가 먼저 들어가게 되고 나중에 그 메세지를 처리하기 위한 클라이언트가 실행된 경우를 시물레이션 하고 있다.
이 경우, consumer 가 실행되자마자 log 가 출력되는 것을 확인할 수 있다.
즉, job processor 가 나중에 실행되어도 처리되지 못한 메세지가 처리될 수 있음을 확인할 수 있다.
