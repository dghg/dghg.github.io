---
layout: post
title: "Sequelize 공식 문서 - Association"
date: 2020-10-15 00:00:00
excerpt: ""
tags:
- javascript
- sequelize
categories:
- 공부
---

# Sequelize 공식 문서 - Association

## Association
Sequelize는 기본적인 1:1, 1:N, N:M(Many-to-Many) 관계를 지원합니다.

이것을 위해, 시퀼라이즈는 4개의 associaton 타입을 지원하고, 이것은 관계를 만들기 위해 조합되어 사용됩니다.

```
HasOne
BelongsTo
HasMany
BelongsToMany
```

 이 가이드는 4개의 association들을 정의하는 방법에 대해 설명하고, 3개의 표준 associaton type(1:1, 1:N, 1:M) 를 정의하기 위해 4개의 association들을 조합하는 방법에 대해 설명합니다.

## Defining the Sequelize associations
4개의 타입들을 비슷한 방법으로 정의됩니다. 우리가 2개의 모델 ```A, B```를 가지고 있다고 생각해 봅시다. 시퀼라이즈에게 2개의 모델들 사이에 association이 있다고 알려주려면 함수를 호출하면 됩니다.

```
const A = sequelize.define('A', ..);
const B = sequelize.define('B", ..);

A.hasOne(B);
A.belongsTo(B);
A.hasMany(B);
A.belongsToMany(B, { through: 'C'});
// C라는 새로운 테이블을 이용해 A와 B사이의 belongsToMany 관계 정의
```

4개의 함수 모두 두번째 매개변수로 options을 줄 수 있습니다.
(처음 3개 함수는 optional이고,```belongsToMany```함수의 through 옵션은 포함해야 합니다.)

```
A.hasOne(B, { /* options */ });
A.belongsTo(B, { /* options */ });
A.hasMany(B, { /* options */ });
A.belongsToMany(B, { through: 'C', /* options */ });
```

association이 정의된 순서는 연관되어 있습니다. 즉, 4개의 케이스에 대해 순서는 중요합니다.
위의 예시에서,```A```는 **source** model이고, ```B```는 **target** model 입니다. 이 용어들은 중요합니다.

첫번째 ```A.hasOne(B)``` associaton 는 ```A```와 ```B``` 사이에 존재하는 1:1 관계를 의미합니다. **여기서 foreign key는 target model인 ```B```에 정의됩니다.**

두번째 ```A.belongsTo(B)``` association는 ```A```와 ```B``` 사이에 존재하는 1:1 관계를 의미합니다. **여기서 foreign key는 source model인 ```A```에 정의됩니다.**

두번째 ```A.hasMany(B)``` asociation는 ```A```와 ```B``` 사이에 존재하는 1:N 관계를 의미합니다. **여기서 foreign key는 target model인 ```B```에 정의됩니다.**

이 3개의 함수들은 Sequelize가 자동으로 적절한 모델에 foreign-key를 추가합니다.

마지막 ```A.belongsToMany(B, { through: 'C'})``` association은 ```A```와 ```B``` 사이에 존재하는 N:M관계를 의미합니다.  이것은 새로운 관계 테이블 ```C```를 이용합니다. ```C```는 2개의 foreign key를 가지고 있습니다.(```aId bId```) Sequelize는 자동으로 이러한 모델 ```C```를 생성합니다.

Note: 위의 에시에서 string ```C```가 함수에 전달됩니다. 이 경우 Sequelize는 이 이름으로 모델을 생성합니다. 이외에도 모델을 미리 정의해두고 option으로 모델을 전달해줄 수도 있습니다.

이것들은 association을 위한 중요한 아이디어입니다. 그러나, 이러한 관계들은 Sequelize를 더 잘 사용하기 위해 보통 한 쌍으로 구성되어 사용됩니다.

## Creating the standard relationships

위에서 언급한대로, Sequelize associaton들은 보통 쌍으로 구성됩니다.
 - 1:1 관계를 만들기 위해, ```hasOne```과 ```belongsTo```가 함께 사용됩니다.
 - 1:N 관계를 만들기 위해, ```hasMany```과 ```belongsTo```가 함께 사용됩니다.
  - N:M 관계를 만들기 위해, 두개의 ```belongsToMany```가 함께 사용됩니다.

## 1:1 관계

### Philosophy
  Sequelize를 이용한 측면에서 살펴보기 전에, 1:1관계에서 무슨 일이 일어나는지 고려해보는것은 유용합니다.

  우리가 두개의 모델 ```Foo``` 와 ```Bar```를 가지고 있다고 생각해 봅시다. 우리는 두개의 모델 사이에 1:1관계를 만드는 것을 원합니다. 우리는 RDB에서 테이블 중 하나에 foreign key를 만듬으로써 수행할 수 있다는 것을 알고 있습니다.
  이 케이스에서, 매우 적절한 질문은 어떤 테이블에 foreign key를 넣을까 입니다. 즉, 우리가 ```Foo```에 ```barId``` column을 넣거나, 대신에 ```Bar```에 ```FooId```를 넣기를 원합니다.

  두개의 옵션 모두 ```Foo``` 와 ```Bar``` 사이에 1:1 관계를 정립하기에 맞는 방법들입니다. 하지만, 우리가 "Foo와 Bar 사이에 1:1 관계가 존재한다" 라고 말할때 그 관계가 필수인지 또는 선택적인지 불분명합니다. 즉 Foo가 Bar 없이 존재할 수 있을까요? 반대로 Bar가 Foo 없이 존재할 수 있을까요? 이 문제에 대한 답은 foreign key를 어디에 넣는지 이해하는데 도움을 줍니다.

### 목표
  예제의 나머지 부분에서, ```Foo``` 와 ```Bar``` 두개의 모델을 사용한다고 가정합시다. 우리는 두개의 모델 사이에 ```Bar```가 ```FooId```를 가지는 1:1 관계를 만들기를 원합니다.

### 구현
  목표를 달성하기 위한 방법은 다음과 같습니다.
  ```
  Foo.hasOne(Bar);
  Bar.belongsTo(Foo);
  ```
  옵션이 아무것도 주어지지 않았으므로, Sequlize는 테이블의 이름들로부터 무엇을 할지 추론합니다. 이 경우 Sequelize는 ```Bar```에 ```fooId```를 추가합니다.

  위의 코드를 수행한 뒤 ```Bar.sync()```를 호출하면 다음과 같은 SQL이 수행됩니다.

  ```
  CREATE TABLE IF NOT EXISTS "foos" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "bars" (
  /* ... */
  "fooId" INTEGER REFERENCES "foos" ("id") ON DELETE SET NULL ON UPDATE CASCADE
  /* ... */
```
### Options
다양한 옵션들이 함수 호출 시 전달될 수 있습니다.

##### onDelete, onUpdate
 - ON DELETE 와 ON UPDATE 설정을 위해 다음과 같이 할 수 있습니다.
 ```
 Foo.hasOne(Bar, {
   onDelete: 'RESTRICT',
   onUpdate: 'RESTRICT',
  });
  ```
  가능한 옵션들은 ```RESTRICT CASCADE NO ACTION SET DEFAULT SET NULL```입니다.

  기본적으로는 ```ON DELETE=SET NULL, ON UPDATE=CASCADE``` 입니다.

### foreign key 커스터마이징
foreign key를 ```myFooId```처럼 다른 이름으로 설정하고 싶다면 다음과 같습니다.

```
// Option 1
Foo.hasOne(Bar, {
  foreignKey: 'myFooId'
});
Bar.belongsTo(Foo);

// Option 2
Foo.hasOne(Bar, {
  foreignKey: {
    name: 'myFooId'
  }
});
Bar.belongsTo(Foo);

// Option 3
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: 'myFooId'
});

// Option 4
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: {
    name: 'myFooId'
  }
});
```
위 코드 처럼, ```foreignKey``` 옵션에 string 또는 object를 전달하면 됩니다. object를 받았다면, 이 object는 ```sequelize.define```처럼 작동하게 됩니다. 따라서 ```type, allowNull, defaultValue```등을 명시할 수 있습니다.

예를 들어, foreign key로```UUID```를 사용하고 싶다면, 다음과 같이 하면 됩니다.

```
const { DataTypes } = require("Sequelize");

Foo.hasOne(Bar, {
  foreignKey: {
    // name: 'myFooId'
    type: DataTypes.UUID
  }
});
Bar.belongsTo(Foo);
```

### 관계가 필수인지 옵션인지
기본적으로, association 은 선택적으로 여겨집니다. 즉 예시에서 ```fooId```는 null이 될 수 있습니다. 이것은 ```allowNull: false``` 옵션을 줌으로써 변경할 수 있습니다.
```
Foo.hasOne(Bar, {foreignkey: {
  allowNull: false,
  }});
```

## 1:N 관계

### Philosophy
1:N 관계는 하나의 소스를 여러개의 targets 들과 연결합니다. 또한 이 targets들은 오직 하나의 source에 연결됩니다.
즉 foreign key를 어디에 위치해야 할지 선택해야 하는 1:1 관계와 달리 1:N 관계에서는 하나의 옵션이 존재합니다. 만약에 Foo 가 여러개의 Bars를 가지고 있으면(각 Bar들은 하나의 Foo에 속합니다.) 알맞은 구현은 ```Bar``` 가 ```fooId```를 가지고 있게 하는 것입니다. 반대는 불가능합니다. 왜냐하면 하나의 Foo는 여러개의 Bar를 가지고 있기 떄문입니다.
(RDB는 atomic value만을 가져아 함)

### 목표
이 예제에선, 우리는 ```Team``` 과 ```Player``` 모델을 사용할 것입니다.
우리는 Sequelize에게 1:N관계가 존재한다고 말할 것입니다. 즉 하나의 ```Team```이 여러개의 ```Player```들을 가지고, 반면에 ```Player```는 오직 하나의 ```Team```에만 속합니다.

### 구현
```
Team.hasMany(Player);
Player.belongsTo(Team);
```
언급한 대로, association을 만드는 중요한 방법은 한 쌍의 Sequelize(```hasMany belongsTo```)를 사용하는 것입니다.

위의 코드는 ```sync()``` 이후 다음과 같은 SQL이 수행됩니다.

```
CREATE TABLE IF NOT EXISTS "Teams" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "Players" (
  /* ... */
  "TeamId" INTEGER REFERENCES "Teams" ("id") ON DELETE SET NULL ON UPDATE CASCADE,
  /* ... */
);
```

### options
적용되는 옵션들은 1:1 관계일 떄와 같습니다.


## N:M 관계

### Philosophy
N:M 관계는 하나의 source를 여러개의 targets과 연결합니다. 동시에 이러한 targets들은 다른 sources들과 연결될 수 있습니다.

이것들은 다른 관계처럼 하나의 테이블에 하나의 foreign key를 추가함으로 수행될 수 없습니다. 대신에 [Junction Model](#https://en.wikipedia.org/wiki/Associative_entity)라는 개념이 사용됩니다. 이것은 extra table로 두개의 foreign key를 가지고 있습니다. 이것은 association 관계를 추적하기 위해 유지됩니다. Junction table은 또한 *join table* 이나 *through table*이라 불립니다.

### 목표
예시를 위해서, 우리는 ```Movie``` 와 ```Actor```라는 모델을 사용합니다. 하나의 actor는 여러개의 movies에 참여할 수 있고, 하나의 movie는 여러명의 actors를 가질 수 있습니다. 관계를 추적하는 Junction table은 ```ActorMovies```라고 불리고, 이 테이블은 두개의 foreign key ```movieId```와 ```actorId```를 가지고 있습니다.

### 구현
```
const Movie = sequlize.define('Movie', {name: DataTypes.STRING});
const Actor = sequelize.define('Actor', {name: DataTypes.STRING});
Movie.belongsToMany(Actor, {through: 'ActorMovies'});
Actor.belongsToMany(Movie, {through: 'ActorMovies'});
```

`through` option으로 문자열이 주어졌기 때문에 Sequlize는 자동으로 ```ActorMovies```라는 모델을 생성합니다. 이 모델이 `Junction model` 역할을 합니다.

이 코드는 다음과 같은 SQL을 수행합니다.
```
CREATE TABLE IF NOT EXISTS "ActorMovies" (
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "MovieId" INTEGER REFERENCES "Movies" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  "ActorId" INTEGER REFERENCES "Actors" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  PRIMARY KEY ("MovieId","ActorId")
);
```

문자열 대신, 직접 model을 전달하는것 또한 지원합니다. 이 경우 주어진 모델이 junction model역할을 합니다.

```
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
const ActorMovies = sequelize.define('ActorMovies', {
  MovieId: {
    type: DataTypes.INTEGER,
    references: {
      model: Movie, // 'Movies' would also work
      key: 'id'
    }
  },
  ActorId: {
    type: DataTypes.INTEGER,
    references: {
      model: Actor, // 'Actors' would also work
      key: 'id'
    }
  }
});
Movie.belongsToMany(Actor, { through: ActorMovies });
Actor.belongsToMany(Movie, { through: ActorMovies });
```

이 코드는 다음과 같은 SQL을 수행합니다.

```
CREATE TABLE IF NOT EXISTS "ActorMovies" (
  "MovieId" INTEGER NOT NULL REFERENCES "Movies" ("id") ON DELETE RESTRICT ON UPDATE CASCADE,
  "ActorId" INTEGER NOT NULL REFERENCES "Actors" ("id") ON DELETE RESTRICT ON UPDATE CASCADE,
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  UNIQUE ("MovieId", "ActorId"),     -- Note: Sequelize generated this UNIQUE constraint but
  PRIMARY KEY ("MovieId","ActorId")  -- it is irrelevant since it's also a PRIMARY KEY
);
```

### options
 위의 2개의 관계와 달리 기본적으로 ```ON UPDATE, ON DELETE = CASCADE```입니다.
 또한 N:M 관계는 junction model에 uniqueKey를 생성할 수 있습니다.

## Basic of queries involving associaton
기본적인 관계 정의와 함께, 우리는 포함된 관계들의 기본적인 쿼리를 살펴볼 것입니다.
가장 자주사용되는 쿼리는 *read* 쿼리입니다.(i.e. SELECTs)

이것을 공부하기 위해, 우리는 ```Ships```와 ```Captains``` 모델을 가지고 있고, 이들 사이의 1:1 관계가 존재합니다. 그리고 foreign key가 null이 되는 것을 허용합니다. 즉 Ship이 Captain 없이 존재할 수 있고, 반대도 같습니다.

```
// This is the setup of our models for the examples below
const Ship = sequelize.define('ship', {
  name: DataTypes.TEXT,
  crewCapacity: DataTypes.INTEGER,
  amountOfSails: DataTypes.INTEGER
}, { timestamps: false });
const Captain = sequelize.define('captain', {
  name: DataTypes.TEXT,
  skillLevel: {
    type: DataTypes.INTEGER,
    validate: { min: 1, max: 10 }
  }
}, { timestamps: false });
Captain.hasOne(Ship);
Ship.belongsTo(Captain);
```

### Fetching associaton - Eager Loading vs Lazy Loading
Eager Loading과 Lazy Loading의 개념은 Sequelize에서 어떻게 fetch association이 작동하는지 이해하기 위한 기본입니다.
Lazy loading은 정말로 원할때만 관련된 data들을 fetching 해오는 기술을 의미합니다.
반대로 Eager Loading은 처음부터 large query를 이용하여 한꺼번에 모든것을 fetching 해오는 기술입니다.

#### Lazy Loading example
```
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  }
});
// Do stuff with the fetched captain
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
// Now we want information about his ship!
const hisShip = await awesomeCaptain.getShip();
// Do stuff with the ship
console.log('Ship Name:', hisShip.name);
console.log('Amount of Sails:', hisShip.amountOfSails);
```
위의 예제를 살펴보면, 우리는 두개의 쿼리를 실행했습니다. 이것은 단지 우리가 사용하기 원할 떄(`getShip`) ship만을 fetching 해옵니다. 이것은 특히 ship이 필요할지 안할지 모를때 특히 유용합니다. 몇몇 케이스에서 우리는 아마 조건적으로 fetch를 원하고, 이러한 방식은 필요할떄만 fetch 함으로써 시간과 메모리를 절약할 수 있습니다.

#### Eager Loading example

```
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  },
  include: Ship
});
// Now the ship comes with it
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
console.log('Ship Name:', awesomeCaptain.ship.name);
console.log('Amount of Sails:', awesomeCaptain.ship.amountOfSails);
```

위의 예제를 살펴보면, include 옵션을 사용함으로써 Eager Loading이 수행됩니다. 하나의 쿼리만이 데이터베이스에서 수행되었습니다(Instance와 함께 연관된 데이터 또한 가져옵니다.)

이것이 Sequelize에서의 Eager Loading 입니다. 더 자세한 것은 [the dedicated guide on Eager Loading](#https://sequelize.org/master/manual/eager-loading.html)에 있습니다.

### Creating, updating and deleting

- 위의 예제에서는 association이 포함되어 있을때 어떻게 데이터를 fetching 해오기 위한 기본적인 쿼리들을 알아봤습니다.
creating, updating and deleting을 위해서는 다음과 같이 할 수 있습니다:
- 직접적으로 모델 쿼리 사용하기
```
// Example: creating an associated model using the standard methods
Bar.create({
  name: 'My Bar',
  fooId: 5
});
// This creates a Bar belonging to the Foo of ID 5 (since fooId is
// a regular column, after all). Nothing very clever going on here.
```
- 또는 이후에 설명할, 연관된 모델을 위한 **특별한 메소드/Mixins** 등을 사용할 수 있습니다.

## Association Aliases & Custom Foreign Keys

위의 모든 예제에서, Sequelize는 자동적으로 foreign key의 이름을 정의했습니다. 예를 들어 `Ship`과 `Captain` 예제에서 Sequelize는 자동으로 Ship model에 `captainId` 필드를 추가합니다. 하지만 사용자가 custom foreign key를 사용하는 것  어렵지 않습니다.

간단한 형태의 `Ship` 과 'Captain'을 고려해 봅시다. (less fields)
```
const Ship = sequelize.define('ship', { name: DataTypes.TEXT }, { timestamps: false });
const Captain = sequelize.define('captain', { name: DataTypes.TEXT }, { timestamps: false });
```
여기서 foreign key에 다른 이름을 명시하는 방법엔 3가지가 있습니다.
 - foreign key를 직접 제공
 - Alias를 정의하기
 - 둘다 하기

### Recap : 기본 설정
간단하게 `Ship.belongsTo(Captain)`을 사용함으로써 sequelize는 자동으로 foreign key 를 생성합니다.
```
Ship.belongsTo(Captain); // This creates the `captainId` foreign key in Ship.

// Eager Loading is done by passing the model to `include`:
console.log((await Ship.findAll({ include: Captain })).toJSON());
// Or by providing the associated model name:
console.log((await Ship.findAll({ include: 'captain' })).toJSON());

// Also, instances obtain a `getCaptain()` method for Lazy Loading:
const ship = Ship.findOne();
console.log((await ship.getCaptain()).toJSON());
```

### 1. 직접 foreign key 이름 제공하기
association을 정의할 때, 옵션과 함께 foreign key를 제공할 수 있습니다.

```
Ship.belongsTo(Captain, { foreignKey: 'bossId' }); // This creates the `bossId` foreign key in Ship.

// Eager Loading is done by passing the model to `include`:
console.log((await Ship.findAll({ include: Captain })).toJSON());
// Or by providing the associated model name:
console.log((await Ship.findAll({ include: 'Captain' })).toJSON());

// Also, instances obtain a `getCaptain()` method for Lazy Loading:
const ship = Ship.findOne();
console.log((await ship.getCaptain()).toJSON());
```

### 2. Alias 정의하기
Alias를 정의하는 것은, 1번처럼 단지 custom foreign key를 명시하는 것 보다 더욱 강력합니다.

```
Ship.belongsTo(Captain, { as: 'leader' }); // This creates the `leaderId` foreign key in Ship.

// Eager Loading no longer works by passing the model to `include`:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // Throws an error
// Instead, you have to pass the alias:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// Or you can pass an object specifying the model and alias:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// Also, instances obtain a `getLeader()` method for Lazy Loading:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```

Aliases를 정의하는 것은, 특히 같은 모델 간에 2개 이상의 관계를 정의할 때 특히 유용합니다. 예를 들어, 두개의 모델 `Mail` 과 `Person`이 있을 때, 우리는 메일의 `sender`와 `receiver`를 나타내기 위해 두 모델 간에는 두개의 association이 필요합니다. 즉 그렇지 않다면, `mail.getPerson()`같은 메소드 호출이 모호해집니다. `sender`와 `receiver` Aliases가 있다면, 두개의 사용가능한 메소드가 있습니다 : `mail.getSender()` `mail.getReceiver()` 그리고 이 두 메소드는 모두 `Promise<Person>`을 반환합니다.

`hasOne` 또는 `belongsTo` 관계의 alias를 정의할 떄, 이름으로 단수형을 사용해야 합니다. 반면 `hasMany` `belongsToMany` 관계의 alias를 정의할 때는, 복수 형을 사용해야 합니다. 마지막으로 N:M 관계의 alias를 정의하는 것은 [Advanced Many-to-Many Associations guide](#https://sequelize.org/master/manual/advanced-many-to-many.html)에서 다룹니다.

### 3. 둘 다 하기
우리는 alias를 정의함과 동시에 foreign key 또한 정의할 수 있습니다.
```
Ship.belongsTo(Captain, { as: 'leader', foreignKey: 'bossId' }); // This creates the `bossId` foreign key in Ship.

// Since an alias was defined, eager Loading doesn't work by simply passing the model to `include`:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // Throws an error
// Instead, you have to pass the alias:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// Or you can pass an object specifying the model and alias:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// Also, instances obtain a `getLeader()` method for Lazy Loading:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```

## Special methods/mixins added to instances
두개의 모델사이에 association을 정의할 떄, 모델들의 객체는 association 간의 상호작용을 위해 특별한 메소드들을 갖습니다.

예를 들어, 두개의 associated 된 모델 `Foo` `Bar`가 있을 때, 그들의 객체는 association type에 따라 다음과 같은 사용가능한 methods/mixins를 얻습니다.

### Foo.hasOne(Bar)
 - `Foo.getBar()`
 - `Foo.setBar()`
 - `Foo.createBar()`

```
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBar()); // null
await foo.setBar(bar1);
console.log((await foo.getBar()).name); // 'some-bar'
await foo.createBar({ name: 'yet-another-bar' });
const newlyAssociatedBar = await foo.getBar();
console.log(newlyAssociatedBar.name); // 'yet-another-bar'
await foo.setBar(null); // Un-associate
console.log(await foo.getBar()); // null
```

### Foo.belongsTo(Bar)
`Foo.hasOne(Bar)`와 같습니다.
- `Foo.getBar()`
- `Foo.setBar()`
- `Foo.createBar()`

### Foo.hasMany(Bar)
 - `Foo.getBars()`
 - `Foo.countBars()`
 - `Foo.hasBar()`
 - `Foo.hasBars()`
 - `Foo.setBars()`
 - `Foo.addBar()`
 - `Foo.addBars()`
 - `Foo.removeBar()`
 - `Foo.removeBars()`
 - `Foo.createBar()`
```
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBars()); // []
console.log(await foo.countBars()); // 0
console.log(await foo.hasBar(bar1)); // false
await foo.addBars([bar1, bar2]);
console.log(await foo.countBars()); // 2
await foo.addBar(bar1);
console.log(await foo.countBars()); // 2
console.log(await foo.hasBar(bar1)); // true
await foo.removeBar(bar2);
console.log(await foo.countBars()); // 1
await foo.createBar({ name: 'yet-another-bar' });
console.log(await foo.countBars()); // 2
await foo.setBars([]); // Un-associate all previously associated bars
console.log(await foo.countBars()); // 0
```
`getter` 메소드는 일반적인 `findAll` 메소드 처럼 인자로 `options`을 받을 수 있습니다.


### Foo.belongsToMany(Bar)
`Foo.hasMany(Bar)`와 같습니다.

- `Foo.getBars()`
- `Foo.countBars()`
- `Foo.hasBar()`
- `Foo.hasBars()`
- `Foo.setBars()`
- `Foo.addBar()`
- `Foo.addBars()`
- `Foo.removeBar()`
- `Foo.removeBars()`
- `Foo.createBar()`

### Note: Method names
위의 예제에서처럼, Sequelize가 이 association methods에 주는 이름들은 접두사 (`get`, `add`, `set`)와 모델이름(첫번째 글자를 대문자로) 로 이루어 졌습니다. 필요한 경우 `foo.setBars()`처럼 복수형이 사용될 수 있습니다.
특히 불규칙한 복수형 또한 Sequelize가 자동적으로 다룹니다. ( Person > People, Hypothesis > Hypotheses)
만약 Alias가 정의되어 있다면, 메소드 이름의 형태로 모델 이름 대신 사용됩니다.
```
Task.hasOne(User, { as: 'Author'});
..task.getAuthor()
```


## 왜 associations들이 쌍으로 정의될까
앞에 언급한것과 위의 대부분의 예제들에서, Sequleize에서 associations들은 주로 한 쌍으로 정의되었습니다.

 - 1:1 관계를 만들기 위해서, `hasOne` 과 `belongsTo` 가 함께 사용됩니다.
 - 1:N 관계를 만들기 위해서, `hasMany`와 `belongsTo` 가 함께 사용됩니다.
 - N:M 관계를 만들기 위해서, `belongsToMany`가 두번 사용됩니다.

 Sequelize가 두 모델 사이에 association을 정의할 때, 오직 *source* model만이 정보를 알고있게 됩니다. 예를 들어 `Foo.hasOne(Bar)`를 사용하였을 때 `Foo` 모델만이 이 association의 존재를 알게 됩니다. 따라서 `Foo` 객체가 `getBar` `setBar` `createBar` 메소드를 가지는 반면, `Bar` 객체는 아무것도 가지지 못합니다.

비슷하게, Eager Loading 을 수행할 때 `Foo.findOne{include: ~}` 는 사용 가능하지만(`Foo` 가 association을 알고있기 떄문에) 반대로 `Bar.findOne({include: ~})`는 사용할 수 없습니다.

따라서, Sequelize를 완전하게 사용하고 싶다면 주로 한 쌍으로 관계를 셋업하여, 두 모델 전부 관계에 대해 알고있게 만듭니다.



## Multiple associations involving the same models
Sequelize에서, 같은 모델들 간에 여러개의 associations들을 정의하는것 또한 가능합니다. 단지 다른 alias를 사용하면 됩니다.
```
Team.hasOne(Game, { as: 'HomeTeam', foreignKey: 'homeTeamId'});
Team.hasOne(Game, { as : 'AwayTeam', foreignKey: 'awayTeamId'});
Game.belongsTo(Team);
```

## primary key가 아닌 필드를 참조하는 Associations
위의 모든 예제에서, associations들은 포함된 모델 간의 primary key(IDs)를 참조하도록 정의되었습니다. 그러나 Sequelize는 association을 구성하기 위해서 primary key가 아닌 다른 필드를 사용하는 것을 허용합니다.

Note: 해당 필드가 primary key는 아니더라도, 반드시 unique 해야 합니다. 그렇지 않으면 에러가 발생합니다.
