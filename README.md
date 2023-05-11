# javascript-typescript-cheatsheet


## Contents
- [Buglist](#buglist)
- [Advanced topics](#advanced-topics)
  - [Mixins](#mixins) 
- [Overloading](#overloading)
  - [Overloading function declarations](#overloading-function-declarations)
- [Enums](#enums)
  - [The basics](#the-basics)
  - [Enums at runtime](#enums-at-runtime)
  - [Downsides of enums](#downsides-of-enums)
    - [Downside: logging](#downside-logging)
    - [Downside: loosse type checking](#downside-loosse-type-checking)
  - [Use cases for enums](#use-cases-for-enums)
    - [Bit patterns/flag](#bit-patterns)
    - [Multiple constants](#multiple-constants)
    - [More self-descriptive than booleans](#more-self-descriptive-than-booleans)
    - [Better string constants](#Better-string-constants)
  - [Alternatives to enums](#alternatives-to-enums)
    - [Unions of singleton values](#unions-of-singleton-values)
      - [Primitive literal types](#primitive-literal-types)
    - [Discriminated unions](#discriminated-unions)
    - [Object literals as enums](#object-literals-as-enums)
      - [Object literals with string-valued properties](#object-literals-with-string-valued-properties)    
- [Type manipulation](#type-manipulation)
  - [Type infering](#type-infering)
    - [Make types in terms of values that we already have](#make-types-in-terms-of-values-that-we-already-have)
      - [Skip redundant interface definiton](#skip-redundant-interface-definiton)
      - [Algebraic data type: Enumerated values](#algebraic-data-type-enumerated-values)
      - [Use as const for constant values](#use-as-const-for-constant-values)
    - [Custom generic utils](#custom-generic-utils)
      - [AsyncReturnType](#AsyncReturnType)
      - [Asyncify](#Asyncify)
      - [RecursivePartial](#RecursivePartial)
  - [Generics](#generics)
    - [Domain modeling](#domain-modeling)   
- [Testing static types](#testing-static-types)
  - [Simple solutions](#simple-solutions)
  - [Testing via code](#testing-via-code)
- [Type narrowing](#type-narrowing)
  - [What does the type narrowing mean ?](#type-narrowing)
  - [In operator narrowing](#in-operator-narrowing)
    - [Use case: discriminated union](#use-case-discriminated-union)
    - [Optional properties](#optional-properties)
- [FAQ Typescript](#faq-typescript)
  - [How can i change readonly property in TS?](#how-can-i-change-readonly-property-in-ts)
  - [Property 'x' has no initializer and is not assigned in the ctor?](#property-'x'-has-no-initializer-and-is-not-assigned-in-the-ctor)  
  - [How to use setter in ctors?](#how-to-use-setters-in-ctors)
  - [How to add types to Object.fromEntries?](#how-to-add-types-to-object.fromEntries)
  - [Typescript Constructor Shorthand](#typescript-constructor-shorthand)
- [Refactoring](#refactoring)
- [Functional programming](#Functional-programming)
- 

## Buglist
| Error description | Link |
| ----------- | ----------- |
| **A mixin class must have a constructor with a single rest parameter of type 'any[]'.ts(2545)** | [Issue](https://github.com/microsoft/TypeScript/issues/37142) |

## Advanced topics

### Mixins
Mixin is a class that contains methods for use by other classes without having to be the parent class of those other classes. How those other classes gain access to the mixin's methods depends on language. Mixins are sometimes described as being **'included'** rather than **`inherited`** .

Mixins can be used to avoid multiple inheritance(the diamond problem) or to **work around lack of support multiple inheritance**. Mixins can also be viewed as a **interface with implemented methods**. This pattern is an example of **dependency inversion principle**.

Typescript 4.2 adds support for declaring that the constructor function is abstract. This is mostly used by people who use the mixin pattern.

```ts
type AbstractConstructor<T> = abstract new (...args: unknown[]) => T;
```

**In case you have a constructor function in mixin, there is a [bug](#buglist).** 



## Overloading
Sometimes a single type signature does not adequately describe how a function works.

### Overloading function declarations

```ts
interface Customer {
      id: string;
      fullName: string;
    }

    const joe: Customer = { id: '12345', fullName: 'Joe Novak' };
    const jane: Customer = { id: '56789', fullName: 'Jane Novakova' };

    const customerById = new Map<string, Customer>([
      [joe.id, joe],
      [jane.id, jane]
    ]);


    class UnknownIdError extends Error {}
    class InvalidFormatIdError extends Error {}
    class UndefinedIdError extends Error {}

    function getFullName(customer: Customer): string;
    function getFullName(
      customer: Map<string, Customer>,
      id: string
    ): string | UnknownIdError | InvalidFormatIdError | UndefinedIdError;
    function getFullName(
      customer: Customer | Map<string, Customer>,
      id?: string
    ): string | UnknownIdError | InvalidFormatIdError | UndefinedIdError {
      if (customer instanceof Map) {
        if (id) {
          if (id.length === 0) return new InvalidFormatIdError(`Not valid id: ${id}`);

          const customerUndefined = customer.get(id);
          return customerUndefined !== undefined ? customerUndefined.fullName : new UnknownIdError(`Unknown id: ${id}`);
        } else {
          return new UndefinedIdError(`Id is undefined.`);
        }
      }
      return customer.fullName;
    }

    const customerAsAny: Customer | Map<string, Customer> = customerById;

    deepStrictEqual(getFullName(jane), jane.fullName);
    deepStrictEqual(getFullName(customerById, joe.id), joe.fullName);

    //  Edge case
    deepStrictEqual(getFullName(customerAsAny as any, undefined as any), new UndefinedIdError(`Id is undefined.`));

    deepStrictEqual(getFullName(customerById, ''), new InvalidFormatIdError(`Not valid id: `));
    deepStrictEqual(getFullName(customerById, '09876'), new UnknownIdError(`Unknown id: 09876`));
```

My advise is to only use **overloading** when it cannot`be  avoided. One alternative is to split an overloaded function into multiple functions with different names, for example: 
```ts
getFullName()
getFullNameViaMap()
getFullNameFromMap()
```

## Enums
### The Basics
An enums maps member names to member values. Namespace for constant values.

The enumerator names are usually **identifiers that beheve as constants** in the language. In some languages, the declaration of an enumereted type also intentionally defines an ordering of its members(are implicitly represented as integers), in others, the enumerators are unordered.
`Boolean type` is ofen a pre-defined enum.

An enumerated type can be seen as a degenerate case: **a tagged union of unit types**.

**Unit type** is a type that allows only one value.
In JS `Undefined` - its only value `undefined` and the same applies to `Null`.

An enumerated type is a special case of **sum type** in which the constructors take no arguments, as exactly one value is defined for each constructor.

**Nullary constructor** - is a constructor that takes no arguments. If a constructor does not take any data arguments, it is nullary.
```ts
const nullaryDataConstructor = O.none;
const unaryDataConstructor O.some(1);
```

### Enums at runtime
TS compiles enums to JS objects.

The normal mapping is from enum member to member values:

```ts
enum FilePermission {
  READ,
  WRITE
}

console.log(FilePermission.READ === 0); //true
```
Numeric enums also support a `reverse mapping` from member values to member names.

```ts
enum FilePermission {
  READ,
  WRITE
}

console.log(FilePermission[0] === 'READ'); //true
```
One use case for reverse mapping is printing the name of enum member.

```ts
enum FilePermission {
  READ,
  WRITE
}

const transformFilePermission = (filePerm: FilePermission) => `FilePermission.${FilePermission[filePerm]}`;
console.log(transformFilePermission(FilePermission.READ) === 'FilePermission.READ');
```
String-based enums have a simpler representation at runtime. TS does not support reverse mapping for string-based enums.


### Downsides of enums
While enums can be useful in TS, there are a few downsides to consider:

- Enums can bloat your code. Enums in TS are implemented as objects with string key mapping to number values. This can add a lot of extra code to your project.
- Enums can be confusing: Enums can be used to represent both numeric and string values.

#### Downside: logging
When logging members of numeric enums, we only see numbers.
```ts
enum FilePermission {
    READ, WRITE
}

console.log(FilePermission.READ)  //0
console.log(FilePermission.WRITE);  //1
```

#### Downside: loosse type checking
When using enum as a type, the values that are allowed statically are not just those of the enum members - any number is accepted:
```ts
enum FilePermission {
  READ,
  WRITE
}

function updateFile(path: string, filePerm: FilePermission){}

updateFile('../src/Ord.ts', 10); // no error
```

### Use-cases for enums
#### Bit patterns/flag
https://exploringjs.com/tackling-ts/ch_enums.html#use-cases-for-enums
#### Multiple constants
https://exploringjs.com/tackling-ts/ch_enums.html#use-cases-for-enums
#### More self-descriptive than booleans
https://exploringjs.com/tackling-ts/ch_enums.html#use-cases-for-enums
#### Better string constants
https://exploringjs.com/tackling-ts/ch_enums.html#use-cases-for-enums

### Alternatives to enums

#### Unions of singleton values
The `Singleton type` is a type with one element. 

##### Primitive literal types
Primitive literal types are `singleton types`.
```ts
type BigInt = 1234n; // --target must be ES2020+
type String = 'abc';
```
Two use-cases for `primitive literal types` are:
- `Overloading on string parameters - event handling` which enables the first argument of the following method call to determine the type of the second argument.
```ts
function addEventListener(elem: HTMLElement, type: 'click',
  listener: (event: MouseEvent) => void): void;
function addEventListener(elem: HTMLElement, type: 'keypress',
  listener: (event: KeyboardEvent) => void): void;
function addEventListener(elem: HTMLElement, type: string,  // (A)
  listener: (event: any) => void): void {
    elem.addEventListener(type, listener); // (B)
  }
```
This allows us to change the type parameter of `listener` depending on the value of parameter `type`.

- We can use a union of primitive literal types to define a type by enumerating its members:
```ts
type NODE_ENV = 'production' | 'development' | 'test';
type Direction = 'Up' | 'Down';
```

#### Discriminated unions

#### Object literals as enums

##### Object literals with string-valued properties
```ts
const NODE_ENV = { production: 'production', development: 'development:', test: 'test' } as const;
type NODE_ENV = typeof NODE_ENV[keyof typeof NODE_ENV];
```





## Type manipulation
Typescript's type system allows expressing types in terms of `other types`. The simplest form of this idea is generic.
We actually have at our disposal a wide variety of `type operators`: `keyof operator`, `typeof operator`, `Type['a'] - index access type` and so on...

### Type infering
**Type infering** - enables the compiler to generate **type interfaces in the compile-time** and check the correctness of the implementation.
In short, it is a Typescript feature that **can obtain, infer data types from your code implementation**.
Typescript checks the code, infer types of variables, and performs static analysis.
This is a crucial feature that seperates Typescript from other type-safe programming languages.

This Typescript feature tries to avoid situations where programmers have to create type interface APIs for almost everything and then implement business logic compatiable with these interface declarations.
This is suiteable for quick prototyping in the application layer.

### Make types in terms of values that we already have

#### Skip redundant interface definiton
Every time you create a type interface, you start to duplicate your code.

Dont:
```ts
type User = {
    username: string;
    age: number;
  };

const user: User = {
    username: 'joe',
    age: 10
  };
```
You should use `typeof` operator.

Do:
```ts
type User = typeof user;

const user = {
    username: 'joe',
    age: 10
  };
```
#### Algebraic data type: Enumerated values
One of the best features of Typescript is `Pattern matching` based on `enumerated values`.
```ts
/**
   * Utility type
   */

  /**
   * RecordToUnion
   */
  type RecordToUnion<R extends Record<string | number, unknown>> = keyof R;

  type FunctionType = (...args: any[]) => any;
  type ReturnType<F> = F extends (...args: any[]) => infer R ? R : never;

  const createObject =
    <F extends string, L extends unknown>([field, value]: [F, L] | readonly [F, L]) =>
    <Fun extends FunctionType, R extends ReturnType<Fun>>(fun: Fun): { [K in F]: R } =>
      ({
        [field]: fun(value)
      } as { [K in F]: R });

  const createEnv = <E extends string>(env: E) => ({ env: env });

  /**
   * Infrastructure types
   *
   */

  /**
   * Node environments enum
   */
  const NODE_ENV = { production: 'production', development: 'development:', test: 'test' } as const;
  type NODE_ENV = RecordToUnion<typeof NODE_ENV>;

  /**
   * Data constructor for node environments
   */
  const createNodeEnv = (env: NODE_ENV): NODE_ENV => env;

  const nodeEnvironmented = createObject(['ENV', 'production'] as const)(createNodeEnv);

  /**
   * Node ports enum
   */
  const NODE_PORT = { 3000: 3000, 4000: 40000 } as const;
  type NODE_PORT = RecordToUnion<typeof NODE_PORT>;

  /**
   * Data constructor for node ports
   */
  const createNodePort = (port: NODE_PORT): NODE_PORT => port;

  const nodePorted = createObject(['PORT', 3000])(createNodePort);

  /**
   * Node config
   */
  const NODE = { ...nodePorted, ...nodeEnvironmented, ...createEnv('node') } as const;
  type NODE = typeof NODE;

  /**
   * Postgres
   */

  const POSTGRES_HOST = 'POSTGRES_HOST';
  type POSTGRES_HOST = typeof POSTGRES_HOST;
  /**
   * Data constructor
   */
  const createPostgresHost = (host: POSTGRES_HOST): POSTGRES_HOST => host;

  const postgresHosted = createObject(['HOST', POSTGRES_HOST])(createPostgresHost);

  const POSTGRES_PASSWORD = 'POSTGRES_PASSWORD';
  type POSTGRES_PASSWORD = typeof POSTGRES_PASSWORD;

  /**
   * Data constructor
   */
  const createPostgresPassword = (password: POSTGRES_PASSWORD): POSTGRES_PASSWORD => password;

  const postgresPassworded = createObject(['PASSWORD', POSTGRES_PASSWORD])(createPostgresPassword);

  const POSTGRES_PORT = { 5432: 5432 };
  type POSTGRES_PORT = RecordToUnion<typeof POSTGRES_PORT>;

  /**
   * Data constructor
   */
  const createPostgresPort = (port: POSTGRES_PORT): POSTGRES_PORT => port;

  const postgresPorted = createObject(['PORT', POSTGRES_PORT])(createPostgresPort);

  /**
   * Postgres config
   */
  const POSTGRES = { ...postgresHosted, ...postgresPassworded, ...postgresPorted, ...createEnv('postgres') } as const;
  type POSTGRES = typeof POSTGRES;

  /**
   * MYSQL
   */
  const MYSQL = { ...postgresPorted, HOST: 'MYSQL_HOST', PASSWORD: 'MYSQL_PASSWORD', ...createEnv('mysql') } as const;
  type MYSQL = typeof MYSQL;

  /**
   * Database
   */
  type DATABASE = MYSQL | POSTGRES;

  /**
   * Application environments
   */

  const APP_ENV = { node: { ...NODE } as NODE, database: { ...POSTGRES } as DATABASE } as const;
  type APP_ENV = typeof APP_ENV;
```
Pattern matching: 
```ts
const printDatabaseInfo = (db: DATABASE) => (log: typeof console.log) => {
    switch (db.env) {
      case 'postgres':
        log(`env: ${db.env},${db.HOST}, PORT: ${db.PORT}`);
        break;
      case 'mysql':
        log(`PORT: ${db.PORT}, ${db.HOST}, env: ${db.env}, `);
        break;
    }
  };
}
```
#### Use as const for constant values
Typescript has `as const` syntax that helps with defining constant values. This feature makes a general type such as `string` to more specific type such as `literal string`;
```ts
const Configuration = {
  serverPort: 1337,
  dbPort: 5432,
} as const;

type Configuration = typeof Configuration;
// type Configuration = {
//    readonly serverPort: 1337;
//    readonly dbPort: 5432;
// }
```

### Custom generic utils
#### AsyncReturnType

```ts
const getUserMock = async () => ({
    data: {
      1: {
        userName: 'joe',
        password: 'pw123',
        email: 'Limonadovyjoe@microsoft.com',
        company: 'Microsoft',
        phone: '602459802',
        createdAt: 1677755799857,
        sendNewsletter: false
      }
    }
  });
  
type FunSignature = (...args: any[]) => any;
type ReturnType<F extends FunSignature> = F extends (...args: any) => infer R ? R : never;

type PromiseFlatten<P> = P extends Promise<infer R> ? PromiseFlatten<R> : P;
type AsyncReturnType<F extends FunSignature> = ReturnType<F> extends Promise<infer R> ? PromiseFlatten<R> : never;

type Response = AsyncReturnType<typeof getUserMock>;
//   {
//     data: {
//         1: {
//             userName: string;
//             password: string;
//             email: string;
//             company: string;
//             phone: string;
//             createdAt: number;
//             sendNewsletter: boolean;
//         };
//     };
// }
```
#### Asyncify
Just using **parameters** is not ideal because it does not handle the **this** arg. If a function did not specify the this parameter, it will
be inferred to `unknown`.

```ts
type FunSignature = (...args: any[]) => any;
type isAny<T> = 0 extends 1 & T ? true : false;
type IsUnknown<T> = isAny<T> extends false ? (unknown extends T ? true : false) : false;

type Asyncify<F extends FunSignature> = F extends (this: infer ThisArg, ...args: infer Arg) => infer R
    ? IsUnknown<ThisArg> extends true
      ? (...args: Arg) => Promise<R>
      : (this: ThisArg, ...args: Arg) => Promise<R>
    : never;

type LoggerFun = (entry: { message: string; level: 'Info' | 'Error' }) => void;
type LoggerInfoFactory = () => { message: string; level: 'Info' };

type LoggerFunAsync = Asyncify<LoggerFun>;
//   (entry: {
//     message: string;
//     level: 'Info' | 'Error';
// }) => Promise<void>

type LoggerInfoFactoryAsync = Asyncify<LoggerInfoFactory>;
// () => Promise<{
//     message: string;
//     level: 'Info';
// }>
```
#### RecursivePartial
Optional properties are also handled using this conditional : `R[K] extends object | undefined`.

```ts
type UserWithArray = { 1: { surname: string; activities: [1, 2, 3]; geo: { x: 1; y: 2 } } };
type UserNested = { data: { user: { id: { name: 'joe'; attrs?: { name: string; age: 100 } } } } };

type RecursivePartial<R> = {
    [K in keyof R]?: R[K] extends object | undefined
      ? RecursivePartial<R[K]>
      : R[K] extends Array<infer V>
      ? RecursivePartial<V>[]
      : R[K];
  } & {};

type UserNestedPartial = RecursivePartial<UserNested>;

const user: UserNestedPartial = { data: { user: { id: { attrs: {} } } } };
```

### Generics
Generic function/class is function/class that is being able to work on a variety of types rather than a single one.

#### Domain modeling
Don`t overuse generics: **Every time you manually pass a static type as a parameter into a generics your code starts to smell.**

Generics is usefull for some **general-purpose utility types** such as `Asyncify, RecursivePartital`. But you have to be carefull, **generics in your custom data model could add extra complexity**.

Don`t: 

```ts
type UserFactory<T extends object> = { firstName: string; lastName: string } & T;
type UserWithPhone = UserFactory<{phone: number}>;
type UserWithEmail = UserFactory<{email: string}>;

type User = UserWithPhone | UserWithEmail;
```

Do:

```ts
type UserName = {fistName: string; lastName: string};
type UserWithPhone = UserName & {phone: number};
type UserWithEmail = UserName & {email: string};
type User = UserWithPhone | UserWithEmail;
```

Better, more detailed-types(high granularity - based on atomic attributes):

```ts
type FirstNamed = { firstName: string };
type LastNamed = { lastName: string };
type UserNamed = FirstNamed & LastNamed;

type Phoned = { phone: number };
type UserWithPhone = UserNamed & Phoned;

type Emailed = {email: string};
type UserWithEmail = UserNamed & Emailed;

type User = UserWithPhone | UserWithEmail;
```

## Testing static types
When it comes to TypeScript code:
- There are many options for testing its behavoir at runtime
- There are far fewer options options for testing its **compile-type types**

### Simple solutions
Do not use comments to check if the result is as intended as this  is very prone to make typos in comments... 
```ts
type User = {
    firstName: string;
    age: number;
    isAdult: boolean;
    x: string;
  };

type CoordsTest = {
    x: number;
    y: number;
    isSerializable: boolean;
  };
  
type SimpleMergeIntersectionV1 = SimpleMergeIntersection<CoordsTest, User>;
//   type UserWithCoordsV2 = {
//     isAdult: boolean;
//     x: string;
//     y: number;
//     isSerializable: boolean;
//     firstName: string;
//     age: number;
// }
```
We should look for automated check. 

### Testing via code
Some type testing libraries implement type checks in TS and lets us use them in our code.

`tsd` 
- is a tool for running tests againts `.d.ts` files and also it **performs custom compilation** to check its type assertions.
- checking for errors: we check for erros via the function `expectError()`

```ts
import { expectType } from 'tsd';

type User = {
  firstName: string;
  age: number;
  isAdult: boolean;
  x: string;
};

type CoordsTest = {
  x: number;
  y: number;
  isSerializable: boolean;
};

declare const test: SimpleMergeIntersection<CoordsTest, User>;

type UserWithCoordsIntersection = { x: string };
expectType<UserWithCoordsIntersection>(test);

```


## Type narrowing
It is type analysis on JS runtime control flow constructs like `if/else`, conditional ternaries, loops, thuthiness checks, etc.. which can all affect those types.  
**Type guard** - is an expression that performs runtime check that guaranthees the type in the current scope.  
**Narrowing** - is the process of refining types to more specific types that declared  

### In operator narrowing
#### Use case: discriminated union

```ts
type Article = {
  frontMatter: Record<string, unknown>;
  content: string;
}

type NotFound = {
  notFound: true;
}

type Page = Article | NotFound;

const renderPage = (page: Page) => {
  if('content' in page) {
    return page.content;
  }
  return '404 - Not found';
}
```
Typescript won't warn us when we made a typo in `content` expression. This is why type predicates are more interesting.  

#### Optional properties
Optional properties will exist in both sides for narrowing.
```ts
type User = {
  id: number;
  username: string;
};

type UserCredentials = {
  username: string;
  password: string;
};

type Login = (credentials: UserCredentials) => TE.TaskEither<Error, User>;

interface Authenticator {
  login: Login;
}

interface AuthenticatorFacebook extends Authenticator {
  login2phase?: Login;
}
interface AuthenticatorTwitter extends Authenticator {
  login2phase?: Login;
}


const authenticate = (auth: Authenticator | AuthenticatorFacebook | AuthenticatorTwitter) => {
  if('login2phase' in auth) {
    auth; // AuthenticatorFacebook | AuthenticatorTwitter
  }
}
```

## FAQ Typescript

### How can i change readonly property in TS?
Typescript provides `readonly` keyword which allows setting value on **initialization** or in **constructor function** only.
```ts

```
### Property 'x' has no initializer and is not assigned in the ctor?
It's because of `Strict Class Initialization` flag which has been introduced in TS 2.7. version.
```ts
class LogEntriesSerializerV2 {
  // Property '_entries' has no initializer and is not definitely assigned in the constructor.ts(2564)
  private _entries: LogEntries[];

  private setEntries(entries: EntryDate | EntryDate[] | Entry | Entry[]) {
    if (isEntry(entries) || isEntryDate(entries)) {
      this._entries = [entries];
    } else {
      this._entries = entries;
    }
  }

  set entries(entries: EntryDate | EntryDate[] | Entry | Entry[]) {
    this.setEntries(entries);
  }

  constructor(entries: Entry | Entry[]);
  constructor(entries: EntryDate | EntryDate[]);
  constructor(entries: EntryDate | EntryDate[] | Entry | Entry[]) {
    this.setEntries(entries);
  }
}
```
To fix the error, use the method below - assigment in constructor.
```ts
class LogEntriesSerializerV2 {
   entries: LogEntries[];


  constructor(entries: Entry | Entry[]);
  constructor(entries: EntryDate | EntryDate[]);
  constructor(entries: EntryDate | EntryDate[] | Entry | Entry[]) {
    // refactor - use factory method instead of overloaded ctors
    if (isEntry(entries) || isEntryDate(entries)) {
      this.entries = [entries];
    } else {
      this.entries = entries;
    }
  }
}
```
### How to use setters in ctors?
Setters and getters should not be called explicitly. The setter is automatically invoked when you assign a value to the property `this.name = name`. Many times some logic is located in setters and as general rule, it is usually better to **seperate validation or transformation(map) logic from data objects - separation of concerns - SOC**.

```ts
set name(newName: string){

}
```

Make a validator class and have it accept and examine some Person object. 

```ts
interface LogRecord {
  readonly message: string;
  toString(): string;
}

export interface Entry extends LogRecord {
  readonly time: Date;
  readonly level: Level;
}

export interface EntryDate extends LogRecord {
  readonly message: string;
  readonly date: Date;
}

type LogEntries = Entry | EntryDate;

 export class Entry implements Entry {
   toString(): string {
     return `${this.level} - ${this.time}: ${this.message}`;
   }
   constructor(readonly message: string, readonly time: Date, readonly level: Level) {}
 }

export class EntryDate implements EntryDate {
  toString(): string {
    return `${this.date.toString()}: ${this.message}`;
  }
  constructor(readonly message: string, readonly date: Date) {}
```
In this case: its better to have own class that has the responsibility of tranforming object to string: 

### How to add types to Object.fromEntries?

V1: Refactor nested conditional types.

```ts
type WritableDeep<T> = { -readonly [K in keyof T]: WritableDeep<T[K]> };

type RecordIndexTypes = string | number | symbol;
type RecordKey<T> = T extends RecordIndexTypes ? T : never;

type FromEntries<T> = T extends readonly [infer K, unknown][]
  ? {
      [K1 in RecordKey<K>]: T extends Array<infer Item>
        ? Item extends [infer K2, infer V2]
          ? K1 extends K2
            ? V2
            : never
          : never
        : never;
    }
  : never;

type FromEntriesReadonly<T> = FromEntries<WritableDeep<T>>;
```

V2:
```ts
type Cast<X, Y> = X extends Y ? X : Y;
type CastToString<T> = Cast<T, string>;

export type FromEntries<T> = T extends readonly [infer K, unknown][]
  ? { [K1 in CastToString<K>]: Extract<T[number], [K1, any]>[1] }
  : never;
```

You can extend existing `fromEntries` method on global object `Object` or you can define your own helper object.

Adding overload of `fromEntries` method in file:

```ts
declare global {
  interface ObjectConstructor {
    fromEntries<T>(entries: T): FromEntriesReadonly<T>;
  }
}


const userEntries = [
  ['username', 'joe'],
  ['company', 'google']
] as const;

const user = Object.fromEntries(userEntries);
// const user: {
//   username: 'joe';
//   company: 'google';
// };
```

Creating own Helper object:

```ts
const ObjectHelper = { create: <T>(entries: T): FromEntriesReadonly<T> => Object.fromEntries as any };

const userJoe = ObjectHelper.create(userEntries);
// const user: {
//   username: 'joe';
//   company: 'google';
// };
```

### Typescript Constructor Shorthand
This concept is called **parameter properties** and is created by prefixing a constructor argument with one of the access modifiers:
`public`, `private`, `protected` or `readonly`. The resulting field gets those modifier(s):

```ts
class Entry {
constructor(readonly message: string, readonly date: Date, readonly level: 'Debug' | 'Error') {}
}
```

In case of extending abstract class, you need to call `super` in constructor method.

```ts
type Kind = 'Circle' | 'Rectangle';
type Shape = Circle | Rectangle;

    abstract class ShapeBase {
      abstract readonly kind: Kind;
      abstract area(): number;
    }

    class Circle extends ShapeBase {
      readonly kind: 'Circle' = 'Circle';
      constructor(readonly radius: number) {
        super();
      }
      area(): number {
        return Math.PI * this.radius ** 2;
      }
    }

    class Rectangle extends ShapeBase {
      readonly kind: 'Rectangle' = 'Rectangle';
      constructor(readonly width: number, readonly height: number) {
        super();
      }
      area(): number {
        return this.width * this.height;
      }
    }

    const printShape = (shape: Shape) => {
      switch (shape.kind) {
        case 'Circle':
          console.log(`Circle: ${shape.radius}`);
          break;
        case 'Rectangle':
          console.log(`Rectangle: ${(shape.width, shape.height)}`);
      }
    };
```



## Refactoring
