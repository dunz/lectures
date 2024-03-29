## 0. Introduction

NestJs는 Node.js의 프레임워크로 express 위에서 동작하고 fastify로 바꿀수 있다

## 1. Architecture of NestJS

- main: 이름 고정인 필수 엔트리 파일
- App.module: 루트 모듈 (필요한 모듈들 연결)
- Controller: 익스프레스의 라우팅 개념, url과 함수 매핑 함수 실행정도의 엳ㄱ할
- Service: 비즈니스로직 구현

## 2. Rest API

### 컨트롤러 만들고 CRUD 라우팅 만들기


컨트롤러 만들기
```sh
$ nest g co
  > movies
```

`src/app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { MoviesController } from './movies/movies.controller';

@Module({
  imports: [],
  controllers: [MoviesController],
  providers: [],
})
export class AppModule {}
```

CRUD 만들기

`src/movies/movies.controller.ts`

```ts
import { Controller, Delete, Get, Param, Patch, Post } from '@nestjs/common';

@Controller('movies')
export class MoviesController {
  @Get()
  getAll() {
    return 'This will return all movies';
  }

  @Get('/:id')
  getOne(@Param('id') movieId: string) {
    return `This will return one movie ${movieId}`;
  }

  @Post()
  create() {
    return `This will create a movie`;
  }

  @Delete('/:id')
  remove(@Param('id') movieId: string) {
    return `This will delete a movie with the id: ${movieId}`;
  }

  @Patch('/:id')
  patch(@Param('id') movieId: string) {
    return `This will patch a movie with the id: ${movieId}`;
  }
}
```

### Body, Query 값 받아오기

`src/movies/movies.controller.ts`

```ts
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Patch,
  Post,
  Query,
} from '@nestjs/common';

@Controller('movies')
export class MoviesController {
  ...
  
  @Get('search')
  search(@Query('title') title: string) {
    return `We are searching for a movie tish a title ${title}`;
  }

  @Get(':id')
  getOne(@Param('id') movieId: string) {
    return `This will return one movie ${movieId}`;
  }

  @Post()
  create(@Body() moviData) {
    return moviData;
  }

  ...

  @Patch(':id')
  patch(@Param('id') movieId: string, @Body() updateData) {
    return {
      updateMovie: movieId,
      ...updateData,
    };
  }
}
```

> - 객체를 리턴할 경우 자동으로 json으로 파싱해서 응답한다
> - 같은 뎁스의 Get 라우팅 중 고정주소가 다이나믹파라미터의 아래에 있을경우 고정주소전에 다이나믹 파라미터로 인식되기 때문에 고정 주소를 위로 올려야한다 (순서 중요)

### Entity 만든 후 서비스에서 비즈니스로직 만들기

엔티티 만들기

`src/movies/entities/movie.entity.ts`

```ts
export class Movie {
  id: number;
  title: string;
  year: number;
  genres: string[];
}
```

비즈니스 로직 만들기

`src/movies/movies.service.ts`

```ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { Movie } from './entities/movie.entity';

@Injectable()
export class MoviesService {
  private movies: Movie[] = [];

  getAll(): Movie[] {
    return this.movies;
  }

  getOne(id: string): Movie {
    const movie = this.movies.find((movie) => movie.id === Number(id));
    if (!movie) {
      throw new NotFoundException(`Not found movie with id: ${id}`);
    }
    return movie;
  }

  deleteOne(id: string) {
    this.getOne(id);
    this.movies = this.movies.filter((movie) => movie.id !== Number(id));
  }

  create(movieData) {
    this.movies.push({
      id: this.movies.length + 1,
      ...movieData,
    });
  }

  update(id: string, updateData) {
    const movie = this.getOne(id);
    this.deleteOne(id);
    this.movies.push({ ...movie, ...updateData });
  }
}
```

`src/movies/movies.controller.ts`

```ts
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Patch,
  Post,
  Query,
} from '@nestjs/common';
import { MoviesService } from './movies.service';
import { Movie } from './entities/movie.entity';

@Controller('movies')
export class MoviesController {
  constructor(private readonly moviesService: MoviesService) {}

  @Get()
  getAll(): Movie[] {
    return this.moviesService.getAll();
  }

  @Get('search')
  search(@Query('title') title: string) {
    return `We are searching for a movie tish a title ${title}`;
  }

  @Get(':id')
  getOne(@Param('id') movieId: string): Movie {
    return this.moviesService.getOne(movieId);
  }

  @Post()
  create(@Body() movieData) {
    return this.moviesService.create(movieData);
  }

  @Delete(':id')
  remove(@Param('id') movieId: string) {
    return this.moviesService.deleteOne(movieId);
  }

  @Patch(':id')
  patch(@Param('id') movieId: string, @Body() updateData) {
    return this.moviesService.update(movieId, updateData);
  }
}
```

### DTO 벨리데이션 추가하기

`class-validator`, `class-transformer` 설치

```sh
$ npm install class-validator class-transformer
```

`src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );
  await app.listen(3000);
}
bootstrap();
```

`src/movies/entities/dto/create-movie.dto.ts`


```ts
import { IsNumber, IsString } from 'class-validator';

export class CreateMovieDto {
  @IsString()
  readonly title: string;

  @IsNumber()
  readonly year: number;

  @IsString({ each: true })
  readonly genres: string[];
}
```

> - whitelist: DTO에 해당하지 않은 키값은 무시
> - forbidNonWhitelisted: DTO에 해당하지 않는 키값을 보낼경우 오류
> - transform: 선언한 파라미터 타입에 맞게 요청으로 받은 데이터를 변환 처리한 후에 컨트롤러로 넘김

### DTO PartialType으로 옵셔널 타입 적용하기

`@nestjs/mapped-types` 설치하기

```sh
$ npm install @nestjs/mapped-types
```

`src/movies/entities/dto/update-movie.dto.ts`

```ts
import { CreateMovieDto } from './create-movie.dto';
import { PartialType } from '@nestjs/mapped-types';

export class UpdateMovieDto extends PartialType<CreateMovieDto>(
  CreateMovieDto,
) {}
```

이렇게 하게되면 `CreateMoviDto`의 벨리데이션은 동일하게 적용하면서 옵셔널 한 값의 DTO를 중복코드없이 쉽게 만들수 있다

### 구조 정리하기

`app.module`에는 `app.controller`와 `app.service`만 controllers, providers로 주입해주고 그 외의 것들은 하나의 모듈로 감싸서
imports 에 넣어주는 구조가 권장된다

`src/app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { MoviesModule } from './movies/movies.module';
import { AppController } from './app.controller';

@Module({
  imports: [MoviesModule],
  controllers: [AppController],
  providers: [],
})
export class AppModule {}
```

`src/app.controller.ts`

```ts
import { Controller, Get } from '@nestjs/common';

@Controller()
export class AppController {
  @Get()
  home() {
    return 'Welcome to my API';
  }
}
```

`src/movies/movies.module.ts`

```ts
import { Module } from '@nestjs/common';
import { MoviesController } from './movies.controller';
import { MoviesService } from './movies.service';

@Module({
  controllers: [MoviesController],
  providers: [MoviesService],
})
export class MoviesModule {}
```

### 권장 문법

기본적으로 express 위에서 nestjs가 돌아가기 때문에 express처럼 req, res 인자를 직접 받아올수 있다

```ts
home(@Req() req, @Res res) {
  ...
}
```


하지만 이 경우 nestjs가 express에 대한 의존도가 높아져 훗날 fastify로 마이그레이션 시 문제가 발생하기 때문에 사용하지 않는걸 권장한다

참고로 fastify는 express보다 약 2배정도 빠르다
