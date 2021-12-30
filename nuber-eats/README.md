## Introduction

### nest 프로젝트 설치하기
```sh
nest g application
  > nuber-eats-backend
```

## GraphQL API

### GraphQL 설치하기

> https://docs.nestjs.com/graphql/quick-start#installation

```sh
npm i @nestjs/graphql graphql@^15 apollo-server-express
```

### graphql 오토 스키파 파일 추가( docs: graphql code first )

> https://docs.nestjs.com/graphql/quick-start#code-first

```ts
@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
    }),
  ],
  controllers: [],
  providers: [],
})
```

### 임의의 모듈 추가
```sh
nest  g mo restaurants
```

### resolver 추가

`src/restaurants/restaurants.resolver.ts`

```ts
import { Query, Resolver } from '@nestjs/graphql';

@Resolver()
export class RestaurantsResolver {
  @Query(() => Boolean)
  isPizzaGood(): boolean {
    return true;
  }
}

`src/restaurants/restaurants.module.ts`

```ts
import { Module } from '@nestjs/common';
import { RestaurantsResolver } from './restaurants.resolver';

@Module({
  providers: [RestaurantsResolver],
})
export class RestaurantsModule {}
```







