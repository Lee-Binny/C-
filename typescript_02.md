**/back/types/passport-local.d.ts**
```typescript
declare module "passport-local" {
  import { Strategy as PassportStrategy } from 'passport';
  
  export interface IVerifyOptions {
    [key: string]: any;
  }
  export interface IStrategyOptions {
    usernameField: string;
    passwordField: string;
  }
  export interface Done {
    // 앞에 있는 매개변수가 optional이면 뒤에 있는 매개변수도 optional이다.
    (error: Error | null, user?: any, options?: IVerifyOptions): void;
  }
  export interface VerifyFunction {
    (username: string, password: string, done: Done): void;
  }
  export class Strategy extends PassportStrategy {
    constructor(options: IStrategyOptions, verify: VerifyFunction)
  }
}
```

### 오버로딩
**/back/types/passport-local.d.ts**
```typescript
declare module "passport-local" {
  import { Strategy as PassportStrategy } from 'passport';
  
  export interface IVerifyOptions {
    [key: string]: any;
  }
  export interface IStrategyOptions {
    usernameField: string;
    passwordField: string;
    session? boolean;
    passReqToCallback?: false;
  }
  export interface IStrategyOptionsWithRequest {
    usernameField: string;
    passwordField: string;
    session? boolean;
    passReqToCallback: true;
  }
  export interface Done {
    (error: Error | null, user?: any, options?: IVerifyOptions): void;
  }
  export interface VerifyFunction {
    // 앞에 req?: Request가 붙으면 뒤에 매개변수도 다 optional이 되는 문제가 발생한다.
    (username: string, password: string, done: Done): void;
  }
  // 오버로딩
  export interface VerifyFunctionWithRequest {
    (req: Request, username: string, password: string, done: Done): void;
  }
  export class Strategy extends PassportStrategy {
    // constructor(options: IStrategyOptions | IStrategyOptionsWithRequest, verify: VerifyFunction | VerifyFunctionWithRequest)
    constructor(options: IStrategyOptions, verify: VerifyFunction)
    constructor(options: IStrategyOptionsWithRequest, verify: VerifyFunctionWithRequest)
  }
}
```
