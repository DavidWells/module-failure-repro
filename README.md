# Module failure repro

Using `node v14.15.1`  with `next.js@10.0.7` package `tiny-cognito@0.0.8` fails with `SyntaxError: Cannot use import statement outside a module` error.

<details>
<summary>package.json contents</summary>

https://github.com/DavidWells/tiny-cognito/blob/536132683e4682b14af1fd5bee126686d3b89197/package.json

```json
{
  "name": "tiny-cognito",
  "version": "0.0.8",
  "description": "Get AWS Cognito creds to use with aws4fetch",
  "source": "src/index.js",
  "main": "dist/index.js",
  "exports": "./dist/index.modern.js",
  "module": "dist/index.module.js",
  "unpkg": "dist/index.umd.js",
  "scripts": {
    "build": "microbundle",
    "dev": "microbundle watch",
    "release:patch": "npm version patch && npm publish",
    "release:minor": "npm version minor && npm publish",
    "release:major": "npm version major && npm publish"
  },
  "author": "DavidWells",
  "license": "ISC",
  "homepage": "https://github.com/DavidWells/tiny-cognito#readme",
  "repository": {
    "type": "git",
    "url": "https://github.com/DavidWells/tiny-cognito"
  },
  "devDependencies": {
    "microbundle": "^0.13.0"
  },
  "dependencies": {
    "@aws-sdk/client-cognito-identity-browser": "0.1.0-preview.2"
  }
}
```

</details>

Run this repro to see in action

## 1. Set Node to version 14.4.

```
nvm use 14
```

## 2. Run next locally

```
npm run dev
```

## 3. Navigate to page

Then navigate to page.

Then you should see error:

```
event - compiled successfully
/Users/dir/scratch/module-failure-repro/node_modules/tiny-cognito/dist/index.modern.js:1
import{CognitoIdentityClient as t}from"@aws-sdk/client-cognito-identity-browser/CognitoIdentityClient";import{GetCredentialsForIdentityCommand as e}from"@aws-sdk/client-cognito-identity-browser/commands/GetCredentialsForIdentityCommand";import{GetIdCommand as n}from"@aws-sdk/client-cognito-identity-browser/commands/GetIdCommand";function r(){return(r=Object.assign||function(t){for(var e=1;e<arguments.length;e++){var n=arguments[e];for(var r in n)Object.prototype.hasOwnProperty.call(n,r)&&(t[r]=n[r])}return t}).apply(this,arguments)}async function o(o={}){const{COGNITO_REGION:w="us-east-1",COGNITO_ENDPOINT:u,IDENTITY_POOL_ID:g,getUserId:I=d,setUserId:y=l,getUserCredentials:f=c,setUserCredentials:m=s,getUserCredentialsKey:C=a,getUserIdKey:p=i,debug:O=!1}=o,h=await C(g),N=await p(g);let _=await I(N);const b=await async function(t,e,n){try{const r=await e(t),o=JSON.parse(r);if(new Date(o.Credentials.Expiration).getTime()>Date.now())return n&&console.log("creds cache hit"),o}catch(t){n&&console.log("getCredentials error",t)}return n&&console.log("creds have expired"),!1}(h,f,O);if(b&&b.Credentials)return b.Credentials;const S=new t(r({region:w,credentials:{}},u?{endpoint:u}:{}));if(!_)try{const t=new n({IdentityPoolId:g});_=(await S.send(t)).IdentityId,await y(N,_)}catch(t){console.error(t)}try{const t=new e({IdentityId:_}),n=await S.send(t);return await m(h,n),n.Credentials}catch(t){console.error(t)}return!1}function i(t){return`__identity-id.${t}`}function a(t){return`__identity-credentials.${t}`}function s(t,e){localStorage.setItem(t,JSON.stringify(e))}function c(t){return window.localStorage.getItem(t)}function d(t){return window.localStorage.getItem(t)}function l(t,e){return window.localStorage.setItem(t,e)}export default o;
^^^^^^

SyntaxError: Cannot use import statement outside a module
    at wrapSafe (internal/modules/cjs/loader.js:979:16)
    at Module._compile (internal/modules/cjs/loader.js:1027:27)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
    at Module.load (internal/modules/cjs/loader.js:928:32)
    at Function.Module._load (internal/modules/cjs/loader.js:769:14)
    at Module.require (internal/modules/cjs/loader.js:952:19)
    at require (internal/modules/cjs/helpers.js:88:18)
    at eval (webpack-internal:///tiny-cognito:1:18)
    at Object.tiny-cognito (/Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:148:1)
    at __webpack_require__ (/Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:23:31)
    at eval (webpack-internal:///./pages/index.js:9:70)
    at Module../pages/index.js (/Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:104:1)
    at __webpack_require__ (/Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:23:31)
    at /Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:91:18
    at Object.<anonymous> (/Users/dir/scratch/module-failure-repro/.next/server/pages/index.js:94:10)
    at Module._compile (internal/modules/cjs/loader.js:1063:30)
```

## Notes

This exact same code works in node 12 but not on node 14. ¯\_(ツ)_/¯