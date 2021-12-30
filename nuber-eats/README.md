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
```

`src/restaurants/restaurants.module.ts`

```ts
import { Module } from '@nestjs/common';
import { RestaurantsResolver } from './restaurants.resolver';

@Module({
  providers: [RestaurantsResolver],
})
export class RestaurantsModule {}
```

### Entity 추가

`src/restaurants/entities/restaurant.entity.ts`

```ts
import { Field, ObjectType } from '@nestjs/graphql';

@ObjectType()
export class Restaurant {
  @Field(() => String)
  name: string;

  @Field(() => Boolean, { nullable: true })
  isGood?: boolean;
}
```

`src/restaurants/restaurants.resolver.ts`

```ts
import { Query, Resolver } from '@nestjs/graphql';
import { Restaurant } from './entities/restaurant.entity';

@Resolver()
export class RestaurantsResolver {
  @Query(() => Restaurant)
  myRestaurant() {
    return true;
  }
}
```

### Args 추가해보기

`src/restaurants/restaurants.resolver.ts`

```ts
import { Args, Query, Resolver } from '@nestjs/graphql';
import { Restaurant } from './entities/restaurant.entity';

@Resolver(() => Restaurant)
export class RestaurantsResolver {
  @Query(() => [Restaurant])
  restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] {
    return [];
  }
}
```






