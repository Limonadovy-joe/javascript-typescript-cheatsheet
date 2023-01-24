# javascript-typescript-cheatsheet


## Contents
- [Type narrowing](#type-narrowing)
  - [What does the type narrowing mean ?](#type-narrowing)
  - [In operator narrowing](#in-operator-narrowing)
    - [Use case: discriminated union](#use-case-discriminated-union)
    - [Optional properties](#optional-properties)
- [FAQ Typescript](#faq-typescript)
  - [How can i change readonly property in TS?](#how-can-i-change-readonly-property-in-ts)
  - [Property 'x' has no initializer and is not assigned in the ctor?](#property-'x'-has-no-initializer-and-is-not-assigned-in-the-ctor)  
  - [How to use setter in ctors?](#how-to-use-setters-in-ctors)
- [Functional programming using FP-TS](#Functional-programming-using-FP-TS)
  - [Reader monad](#Reader-monad)


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









