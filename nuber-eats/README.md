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

### 임의의 모듈 추가하기
```sh
nest  g mo restaurants
```

### resolver 추가하기

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

### Entity 추가하기

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

### Args 추가하기

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

### ArgsType 추가하기

`src/restaurants/dtos/create-restaurant.dto.ts`

```ts
import { Args, ArgsType, Field, InputType } from '@nestjs/graphql';

@ArgsType()
export class CreateRestaurantDto {
  @Field(() => String)
  name: string;

  @Field(() => Boolean)
  isVegan: boolean;

  @Field(() => String)
  address: string;

  @Field(() => String)
  ownerName: string;
}
```

`src/restaurants/restaurants.resolver.ts`

```ts
@Resolver(() => Restaurant)
export class RestaurantsResolver {
  ...

  @Mutation(() => Boolean)
  createRestaurant(@Args() createRestaurantDto: CreateRestaurantDto) {
    console.log('createRestaurantInput', createRestaurantDto);
    return true;
  }
}
```

> `@InputType`은 하나의 ObjectType으로 묶어서 인자를 넘길수 있고<br/>
> `@ArgsType`은 하나의 클래스로 선언하지만 해당 클래스의 프로퍼티들을 풀어서 넘겨준다

### Class Validator 사용하기

설치

```sh
npm i class-validator
```

적용하기

`src/restaurants/dtos/create-restaurant.dto.ts`

```ts
import { ArgsType, Field } from '@nestjs/graphql';
import { IsBoolean, IsString, Length } from 'class-validator';

@ArgsType()
export class CreateRestaurantDto {
  @Field(() => String)
  @IsString()
  @Length(5, 10)
  name: string;

  @Field(() => Boolean)
  @IsBoolean()
  isVegan: boolean;

  @Field(() => String)
  @IsString()
  address: string;

  @Field(() => String)
  @IsString()
  ownerName: string;
}
```

`src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

## Database Configuration

### postgreSQL 설치하기

https://postgresapp.com/

postgre앱, postico앱 설치하기

### 데이터베이스 만들기

postico앱에서 데이터 베이스 추가

### 유저 패스워드 변경하기

추가한 데이터베이스를 postgre 앱에서 실행하여 터미널을 띄운후에 명령어 입력

```sh
ALTER USER dunz WITH PASSWORD '12345'
```

