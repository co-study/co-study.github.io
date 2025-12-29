---
layout: default
parent: 안나 스터디
title: Nest.js forRoot & forRootAsync
date: 2025-11-23
author: 조안나
category: anna
tags:
  - NestJS
---

forRoot와 forRootAsync는 모두 모듈 초기화 시 필요한 옵션을 설정하는 함수입니다.
forRootAsync는 옵션을 만드는 useFactory 함수를 async로 동작하도록 설정할 수 있습니다.
또는, 다른 의존성이 준비된 뒤에 초기화를 해야 하는 경우에도 사용됩니다.

### ex) 데이터베이스 초기화

```ts
// forRoot - 하드코딩된 설정
TypeOrmModule.forRoot({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password123',
})


// forRootAsync - 동적으로 설정 가져오기
TypeOrmModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (configService: ConfigService) => ({
    type: 'mysql',
    host: configService.get('DB_HOST'),
    port: configService.get('DB_PORT'),
    username: configService.get('DB_USER'),
    password: configService.get('DB_PASS'),
  })
})

```

- forRoot는 하드코딩된 옵션 객체를 그대로 넘겨줍니다.
- forRootAsync는 옵션을 설정하기 위한 의존성인 ConfigService와, 옵션을 생성하는 팩토리 함수를 함께 넘겨줍니다. forRootAsync 함수를 사용해야 환경에 따라 다른 의존성을 사용할 수 있습니다.
- useFactory 함수가 async가 아닐 때도 forRootAsync가 사용되는 이유는 inject 옵션을 통해 옵션을 가져오기  위해 필요한 configService와 같은 타 의존성을 주입할 수 있기 때문입니다.

```ts
import { readFile } from 'fs/promises';

TypeOrmModule.forRootAsync({
  inject: [ConfigService],
  useFactory: async (configService: ConfigService) => {
    // SSL 인증서 파일을 비동기로 읽기
    const sslCert = await readFile(
      configService.get('DB_SSL_CERT_PATH'), 
      'utf-8'
    );
    
    return {
      type: 'mysql',
      host: configService.get('DB_HOST'),
      port: configService.get('DB_PORT'),
      username: configService.get('DB_USER'),
      password: configService.get('DB_PASS'),
      ssl: {
        ca: sslCert,  // 파일에서 읽어온 인증서
      },
    };
  }
})
```

- 파일을 읽는 비동기 작업이 필요한 경우 이렇게 useFactory에 async 키워드를 붙여 사용할 수 있습니다. 이렇게 하면 NestJS가 파일 읽기가 완료될 때까지 기다린 후 모듈을 초기화하므로, SSL 인증서가 제대로 로드된 상태로 DB 연결을 시작할 수 있습니다.

reference : <https://docs.nestjs.com/graphql/quick-start#async-configuration>
