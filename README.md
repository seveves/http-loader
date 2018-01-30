# @ngx-translate/http-loader [![Build Status](https://travis-ci.org/ngx-translate/http-loader.svg?branch=master)](https://travis-ci.org/ngx-translate/http-loader) [![npm version](https://img.shields.io/npm/v/@ngx-translate/http-loader.svg)](https://www.npmjs.com/package/@ngx-translate/http-loader)

A loader for [ngx-translate](https://github.com/ngx-translate/core) that loads translations using http.

Get the complete changelog here: https://github.com/ngx-translate/http-loader/releases

* [Installation](#installation)
* [Usage](#usage)

## Installation

We assume that you already installed [ngx-translate](https://github.com/ngx-translate/core).

Now you need to install the npm module for `TranslateHttpLoader`:

```sh
npm install @ngx-translate/http-loader --save
```

**NB: if you're still on Angular <4.3, please use Http from @angular/http with http-loader@0.1.0.**

## Usage
#### 1. Setup the `TranslateModule` to use the `TranslateHttpLoader`:

The `TranslateHttpLoader` uses HttpClient to load translations, which means that you have to import the HttpClientModule from `@angular/common/http` before the `TranslateModule`:

```ts
import {NgModule} from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
import {HttpClientModule, HttpClient} from '@angular/common/http';
import {TranslateModule, TranslateLoader} from '@ngx-translate/core';
import {TranslateHttpLoader} from '@ngx-translate/http-loader';
import {AppComponent} from "./app";

// AoT requires an exported function for factories
export function HttpLoaderFactory(http: HttpClient) {
    return new TranslateHttpLoader(http);
}

@NgModule({
    imports: [
        BrowserModule,
        HttpClientModule,
        TranslateModule.forRoot({
            loader: {
                provide: TranslateLoader,
                useFactory: HttpLoaderFactory,
                deps: [HttpClient]
            }
        })
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

The `TranslateHttpLoader` also has two optional parameters:
- prefix: string = "/assets/i18n/"
- suffix: string = ".json"

By using those default parameters, it will load your translations files for the lang "en" from: `/assets/i18n/en.json`.

You can change those in the `HttpLoaderFactory` method that we just defined. For example if you want to load the "en" translations from `/public/lang-files/en-lang.json` you would use:

```ts
export function HttpLoaderFactory(http: HttpClient) {
    return new TranslateHttpLoader(http, "/public/lang-files/", "-lang.json");
}
```

For now this loader only support the json format.

## Custom TranslateLoader strategies
### HashTranslateLoader - Hashing translation files with angular-cli, webpack and SystemJS
Allow caching of translation files by the browser could help to speed up the initial load of your app but with the default setup your translation files could miss the latest changes because the cache doesn't have the latest version of your translation file yet.

When using the angular-cli (uses webpack under the hood) you can write your own `TranslateLoader` that always loads the latest translation file available during your application build.

```typescript
// webpack-translate-loader.ts
import { TranslateLoader } from '@ngx-translate/core';
import { Observable } from 'rxjs/Observable';

export class HashTranslateLoader implements TranslateLoader {
  getTranslation(lang: string): Observable<any> {
    return Observable.fromPromise(System.import(`../assets/i18n/${lang}.json`));
  }
}
```

When your project cannot find `System` then adding this to your `typings.d.ts` file helps:
```typescript
declare var System: System;
interface System {
  import(request: string): Promise<any>;
}
```

Now you can use the `HashTranslateLoader` with your `app.module`:
```typescript
@NgModule({
  bootstrap: [AppComponent],
  imports: [
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useClass: HashTranslateLoader
      }
    })
  ]
})
export class AppModule { }
```

One disadvantage of this solution is that you have to rebuild your application when there are only changes inside your language files.
