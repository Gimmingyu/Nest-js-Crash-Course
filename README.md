# Nest-js-Crash-Course

Crash course Learning Nest js

## 설치

```shell
$ npm i -g @nestjs/cli
$ nest new project-name
```

## 개요

- app.controller.ts : 단일 경로가 있는 기본 컨트롤러
- app.controller.spec.ts: 컨트롤러에 대한 단위 테스트
- app.module.ts : 애플리케이션의 루트 모듈
- app.service.ts : 기본 서비스 제공
- main.ts : 핵심 기능 `Nest Factory`를 사용하여 Nest 애플리케이션 인스턴스를 생성하는 애플리케이션의 항목

## 플랫폼

Nest js는 어댑터를 생성해주면 모든 Node HTTP 프레임 워크와 함께 작동이 가능하다.

> EX: platform-express, platform-fastify

## 컨트롤러

컨트롤러는 들어오는 요청을 처리하고 클라이언트에 응답을 반환하는 역할 수행.

기본 컨트롤러를 만들기 위해 클래스와 데코레이터를 사용한다.

데코레이터는 클래스를 메타데이터와 연결하고 Nest가 라우팅 맵을 생성할 수 있도록 한다.

컨트롤러 cats 추가 예시

```shell
nest g controller cats
CREATE src/cats/cats.controller.spec.ts (478 bytes)
CREATE src/cats/cats.controller.ts (97 bytes)
UPDATE src/app.module.ts (322 bytes)
```

**유효성 검사가 내장된 CRUD 컨트롤러를 빠르게 생성하려면 CRUD 생성기도 있다.**

```shell
$ nest g resource [name]
```

cats.controller.ts

```ts
import { Controller } from "@nestjs/common";

@Controller("cats")
export class CatsController {}
```

cats.controller.spec.ts

```ts
import { Test, TestingModule } from "@nestjs/testing";
import { CatsController } from "./cats.controller";

describe("CatsController", () => {
  let controller: CatsController;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [CatsController]
    }).compile();

    controller = module.get<CatsController>(CatsController);
  });

  it("should be defined", () => {
    expect(controller).toBeDefined();
  });
});
```

### 리소스

```ts
import { Controller, Get, Post } from "@nestjs/common";

@Controller("cats")
export class CatsController {
  @Post()
  create(): string {
    return "This action adds a new cat";
  }

  @Get()
  findAll(): string {
    return "This action returns all cats";
  }
}
```

> 상태코드
> POST 요청(201)을 제외하고는 기본적으로 200이다.
> 데코레이터를 사용해(`@HttpCode(204)`처럼 임의로 변경도 가능)

> 헤더
> 사용자 지정 응답 헤더를 지정하려면 `@Header()` 데코레이터 또는 `res.header()` 호출하면 된다.

> 리디렉션
> `@Redirect(url?, statusCode?)` `res.redirect()` 로 표현 가능

> 경로 매개변수
> `@Param`을 사용하여 매개변수를 받아올 수 있다.

```ts
@Get(':id')
findOne(@Param() params): string {
	console.log(params.id);
	return `This action returns a #${params.id} cat`;
}
```

### 페이로드 요청시 Data Transfer Object 스키마 사용방법

create-cat.dto.ts

```ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

CatsController

```ts
@Post()
async create(@Body() createCatDto: CreateCatDto) {
	return 'This action adds a new cat'
}
```

## 공급자

예시

cats.service.ts

```ts
import { Injectable } from "@nestjs/common";
import { Cat } from "./interfaces/cat.interface";

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

cat.interface.ts

```ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

cat.controller.ts

```ts
import { Controller, Get, Post, Body } from "@nestjs/common";
import { CreateCatDto } from "./dto/create-cat.dto";
import { CatsService } from "./cats.service";
import { Cat } from "./interfaces/cat.interface";

@Controller("cats")
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

### 의존성 주입

```ts
constructor(private catsService: CatsService) {}
```

### 선택적 provider

경우에 따라 반드시 해결할 필요가 없는 종속성이 있다.

```ts
import { Injectable, Optional, Inject } from "@nestjs/common";

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject("HTTP_OPTIONS") private httpClient: T) {}
}
```

### 속성 기반 주입

위의 예시들은 전부 생성자 기반 주입이었다.
어떤 경우에는 속성 기반 주입이 효과적이다.

최상위 클래스가 하나 또는 여러 공급자에 의존하는 경우, super()를 사용하지말고, `@Inhect()`를 사용하자.
다만, 예시와 같은 경우가 아니면 생성자 기반 주입이 효과적일 것이다.

```ts
import { Injectable, Inject } from "@nestjs/common";

@Injectable()
export class HttpService<T> {
  @Inject("HTTP_OPTIONS")
  private readonly httpClient: T;
}
```

### 공급자 등록

app.module.ts

```ts
import { Module } from "@nestjs/common";
import { CatsController } from "./cats/cats.controller";
import { CatsService } from "./cats/cats.service";

@Module({
  controllers: [CatsController],
  providers: [CatsService]
})
export class AppModule {}
```

## 모듈

각 app에는 최소한 하나의 루트 모듈이 있다.

루트 모듈을 이용해 Nest가 app graph를 빌드한다.

프로젝트 규모가 커지면 모듈을 확장하는 것을 고려해야한다.

데코레이터: `@Module()`

providers, controllers, imports, exports => 모듈을 설명하는 요소들

모듈은 공급자를 캡슐화한다.

현재 모듈의 일부도 아니고 가져온 모듈에서 내보낸 것도 아닌데 주입할 수는 없다.

### 기능 모듈

기능 모듈은 단순히 특정 기능과 관련된 코드를 구성함으로써 명확한 경계를 정해준다.

기능 모듈을 사용하면 프로젝트 규모가 커질 때 복잡성 관리와 SOLID 원칙 준수를 도와준다.

cats.module.ts

```ts
import { Module } from "@nestjs/common";
import { CatsController } from "./cats.controller";
import { CatsService } from "./cats.service";

@Module({
  controllers: [CatsController],
  providers: [CatsService]
})
export class CatsModule {}
```

모듈 추가 예시

```shell
$ nest g controller cats
$ nest g service cats
$ nest g module cats
```

그리고 나서 루트 모듈로 가져오면 된다.

app.module.ts

```ts
import { Module } from "@nestjs/common";
import { CatsModule } from "./cats/cats.module";

@Module({
  imports: [CatsModule]
})
export class AppModule {}
```

### 공유 모듈

Nest에서 모듈은 기본적으로 싱글톤이다.

모듈을 공유함으로써 유동성을 높이고 확장할 때 편리하게 모듈을 관리할 수 있다.

아래는 자기 자신을 export 하는 예시 (의존성 주입도 가능하지만, 모듈 클래스 자체를 주입할 수는 없다.)
cats.module.ts

```ts
import { Module } from "@nestjs/common";
import { CatsController } from "./cats.controller";
import { CatsService } from "./cats.service";

@Module({
  controllers: [CatsController],
  providers: [CatsService]
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

### 글로벌 모듈

모든 곳에서 동일한 모듈 세트를 공유하는 것은 비효율적이다.

`@Global()`을 이용하자

글로벌 모듈은 루트또는 코어 모듈에 한 번만 등록되어야 한다.

```ts
import { Module, Global } from "@nestjs/common";
import { CatsController } from "./cats.controller";
import { CatsService } from "./cats.service";

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

### 동적 모듈

provider를 동적으로 등록하고 구성할 수 있는 모듈을 만들게 도와준다.

DatabaseModule.ts

```ts
import { Module, DynamicModule } from "@nestjs/common";
import { createDatabaseProviders } from "./database.providers";
import { Connection } from "./connection.provider";

@Module({
  providers: [Connection]
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers
    };
  }
}
```

> `forRoot()`는 동기식 또는 비동기식으로 동적 모듈을 반환할 수 있다.

## 미들웨어

미들웨어는 Route Handler전에 호출되는 함수이다.

Nest 미들웨어는 기본적으로 Express 미들웨어와 동일하다.

하나의 미들웨어는 성공시 next()를 호출해 스택의 다음 미들웨어를 호출한다.

logger.middleware.ts

```ts
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log("Request...");
    next();
  }
}
```

### 미들웨어에서 의존성 주입

Nest 미들웨어는 Dependency Injection을 완벽하게 지원한다.

공급자 및 컨트롤러와 마찬가지로 동일한 모듈 내에서 사용 가능한 종속성을 주입할 수 있다.

미들웨어는 모듈 데코레이터 영역에는 들어갈 수 없지만, 모듈 클래스의 메서드로 사용이 가능하다.

미들웨어를 포함하는 모듈은 NestModule을 구현해주어야 한다.

app.module.ts

```ts
import { Module, NestModule, MiddlewareConsumer } from "@nestjs/common";
import { LoggerMiddleware } from "./common/middleware/logger.middleware";
import { CatsModule } from "./cats/cats.module";

@Module({
  imports: [CatsModule]
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes("cats");
  }
}
```

### consumer

consumer는 미들웨어를 도와주는 클래스이다.
forRoutes()를 이용해 더 효과적으로 적용할 수도 있다.

```ts
import { Module, NestModule, MiddlewareConsumer } from "@nestjs/common";
import { LoggerMiddleware } from "./common/middleware/logger.middleware";
import { CatsModule } from "./cats/cats.module";
import { CatsController } from "./cats/cats.controller";

@Module({
  imports: [CatsModule]
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes(CatsController);
  }
}
```

> **`apply()`메서드에 여러 미들웨어를 집어넣는 것도 가능하다.**

### 기능적(함수) 미들웨어로의 전환

미들웨어에 종속성이 필요하지 않다면 기능적 미들웨어를 사용하는 것이 나을 수 있다.

```ts
import { Request, Response, NextFunction } from "express";

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
}
```

app.module.ts

```ts
consumer.apply(logger).forRoutes(CatsController);
```

### 다중 미들웨어

```ts
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

### 글로벌 미들웨어

```ts
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```

## 예외 처리
