### passport 라이브러리 커스텀
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

### axios 라이브러리 커스텀
**/axios/tsconfig.json**
```json
{
  "compilerOptions": {
    "strict": true,
    "declaration": true,
    "lib": ["dom", "es2020"],
  }
}
```

**/axios/package.json**
```json
{
  ...
  "types": "./index.d.ts",
  ...
}
```

**/axios/index.ts**
```typescript   
export interface AxiosConfig {
  headers?: {
    [key: string]: string;
  }
  withCredentials?: boolean;
}

export interface AxiosData {
  [key: string]: any;
}

export interface AxiosResult {
  data: any;
  status: number;
  statusText: string;
}

function isAxiosData(data: any): data is AxiosData {
  if (data !== null) return false;
  if (data instanceof FormData) return false;
  return typeof data === 'object';
}

export interface Axios {
  defaults: {
    baseUrl: string;
    headers: {
      [key: string]: string;
    }
  }
  get(url: string, config?: AxiosConfig): Promise<AxiosResult>;

  put(url: string, data?: string | AxiosData | FormData, config?: AxiosConfig): Promise<AxiosResult>;

  patch(url: string, data?: string | AxiosData | FormData, config?: AxiosConfig): Promise<AxiosResult>;

  post(url: string, data?: string | AxiosData | FormData, config?: AxiosConfig): Promise<AxiosResult>;

  delete(url: string, config?: AxiosConfig): Promise<AxiosResult>;

  options(url: string, config?: AxiosConfig): Promise<AxiosResult>;

  head(url: string, config?: AxiosConfig): Promise<AxiosResult>;
}

const axios: Axios = {
  defaults: {
    baseUrl: '',
    headers: {},
  },
  get(url, config) {
    // Prmoise 타입으로 반환하기
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('GET', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      xhr.send();
    });
  },
  put(url, data, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('PUT', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      // send에는 json 타입이 가능해서 Object를 stiringify 해야 함
      if (isAxiosData(data)) {
        xhr.send(JSON.stringify(data));
      } else {
        xhr.send(data);
      }
    });
  },
  patch(url, data, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('PATCH', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      if (isAxiosData(data)) {
        xhr.send(JSON.stringify(data));
      } else {
        xhr.send(data);
      }
    });
  },
  post(url, data, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('POST', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      if (isAxiosData(data)) {
        xhr.send(JSON.stringify(data));
      } else {
        xhr.send(data);
      }
    });
  },
  delete(url, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('DELETE', axios.defaults.baseUrl + url);
      const headers: { [key: string]: string } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      xhr.send();
    });
  },
  options(url, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        }
      };
      xhr.onerror = function () {
        reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('OPTIONS', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      xhr.send();
    });
  },
  head(url, config) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.onload = function () {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve({
            data: xhr.responseText,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
           reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
        }
      };
      xhr.onerror = function () {
         reject({
          data: xhr.responseText,
          status: xhr.status,
          statusText: xhr.statusText,
        });
      };
      xhr.open('HEAD', axios.defaults.baseUrl + url);
      const headers: { [key: string]: any } = { ...axios.defaults.headers, ...config?.headers };
      Object.keys(headers).map((key) => {
        xhr.setRequestHeader(key, headers[key]);
      });
      xhr.withCredentials = config?.withCredentials || false;
      xhr.send();
    });
  },
};

export default axios;
```

**/axios/index.html**
```
<script src="./index.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.16/require.min.js"></script>
<script>
  requirejs.config({
    paths: {
      axios: './index.js'
    }
  });

  requirejs(['axios'], function (axios) {
    axios.get('https://api.github.com/users/mzabriskie')
      .then(function (user) {
        document.getElementById('useravatar').src = user.data.avatar_url;
        document.getElementById('username').innerHTML = user.data.name;
      });
  });
</script>
<script>
  console.log(axios);
  axios.defaults.baseUrl = 'https://jsonplaceholder.typicode.com/todos/1';
  axios.get('/').then(function(result) {
    console.log(result);
  });
</script>
```

- `npm publish` 명령어를 입력한다.
 
