# javascript-typescript-cheatsheet


## Contents
- [Type manipulation](#type-manipulation)
  - [Type infering](#type-infering)
    - [Make types in terms of values that we already have](#make-types-in-terms-of-values-that-we-already-have)
      - [Skip redundant interface definiton](#skip-redundant-interface-definiton)
      - [Algebraic data type: Enumerated values](#algebraic-data-type-enumerated-values) 
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
- [Refactoring](#refactoring)
  - [Basic](#basic-refactoring) 
    - Validator object[#validator-object]
- [Functional programming using FP-TS](#Functional-programming-using-FP-TS)
  - [Reader monad](#Reader-monad)

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
  const NODE = { ...nodePorted, ...nodeEnvironmented };
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
  const POSTGRES = { ...postgresHosted, ...postgresPassworded, ...postgresPorted };
  type POSTGRES = typeof POSTGRES;

  /**
   * Application environments
   */
  const APP_ENV = {
    node: { ...NODE },
    postgres: { ...POSTGRES }
  };
  type APP_ENV = typeof APP_ENV;
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


## Refactoring
### Basic
#### Validator object
Before: 
```ts
const personValidator = {
      firstName: {
        minLength: 2,
        maxLength: 6
      },
      lastName: {
        minLength: 3,
        maxLength: 9
      },
      password: {
        capitalLetterRequired: true,
        capitalLetterPattern: /[A-Z]/g,
        digitRequired: true,
        digitPattern: /[A-Z]/g,
        minLength: 5,
        maxLength: 15
      }
    };
```
After: 
```ts

interface Person {
  firstName: string;
  lastName: string;
  password: string;
  registeredAt: Date;
}

type PersonValidationSuccess = { value: string };
type PersonValidationError = { error: Error };

type PersonValidation<P extends Person> = { [K in keyof P]: PersonValidationSuccess | PersonValidationError };

type ValidatorBase = { name: string; required: boolean };
type ValidatorTag<K extends string> = { _tag: K };

type NumberComparators = '>' | '<' | '===' | '!==';
type DateComparators = 'after' | 'before';

type ValidatorString = { pattern: RegExp } & ValidatorTag<'string'>;
type ValidatorNumber = { comparator: NumberComparators; expected: number } & ValidatorTag<'number'>;
type ValidatorDate = { comparator: DateComparators; expected: Date } & ValidatorTag<'date'>;

type ValidationFlag = ValidatorBase & { validator: ValidatorString | ValidatorNumber | ValidatorDate };
type ValidationFlags = ValidationFlag[];

type Validator<O> = {
  [K in keyof O]: {
    [K1 in 'flags']: ValidationFlags;
  };
};

type PersonValidator = Omit<Validator<Person>, 'fullName'>;

const personValidator: PersonValidator = {
      firstName: {
        flags: [
          {
            name: 'MinLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '>',
              expected: 3
            }
          },
          {
            name: 'MaxLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '<',
              expected: 10
            }
          }
        ]
      },
      lastName: {
        flags: [
          {
            name: 'MinLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '>',
              expected: 3
            }
          },
          {
            name: 'MaxLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '<',
              expected: 10
            }
          }
        ]
      },
      password: {
        flags: [
          {
            name: 'AtLeastOneCapitalLetter',
            required: true,
            validator: {
              _tag: 'string',
              pattern: /[A-Z]/g
            }
          },
          {
            name: 'AtLeastOneDigit',
            required: true,
            validator: {
              _tag: 'string',
              pattern: /[0-9]/g
            }
          },
          {
            name: 'MinLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '>',
              expected: 3
            }
          },
          {
            name: 'MaxLength',
            required: true,
            validator: {
              _tag: 'number',
              comparator: '<',
              expected: 10
            }
          }
        ]
      },
      registeredAt: {
        flags: [
          {
            name: 'AfterDate',
            required: true,
            validator: {
              _tag: 'date',
              comparator: 'after',
              expected: new Date()
            }
          }
        ]
      }
    };
```








