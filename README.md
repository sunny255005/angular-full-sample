# Getting Started

Full sample application built with Angular 4 that follows all best practices. This app contains:

1. Application of Lazy Loading Modules.
2. Api configurations by environments ("dev", "prod" or "mock").
3. Integration with translation module.
4. Mock system defined with "json-server" and "faker.js" library.
5. Bootstrap and Font-awsone integration.
6. Dynamic nav bar definition.
7. Http interceptor system.
8. Login component configurable with different authentication service implementation.
9. Custom service validation already available (es. creditCardValidator, emailValidator, passwordValidator ...).
10. Customizable logger system.
11. Incorporated automatic handle errors.
11. TSLint integration for quality.
12. Unit and Functional test with Jasmine, Karma and Protractor.

## Core Module
1. Spinner component to be displayed before to load any resource and hidden at the end.
2. Base "Header" and "Footer" component to be overriden.
3. Include a nav customizable bar with language selector that allow the creation of multi-language apps.
4. Automatic handle errors to display different messages and formats depending of the error type.
5. Http Interceptor system to configure extra things on each http call.
6. Multiple guards (auth, login, module-import) to protect routes and module loading.
7. Custom authentication service implementation. Include an "Oauth2 client" service to integrate your login page with OAuth 2.0 service and other service (SkypAuthService) to by pass the login automatically and redirect to the home page.
8. Validation service included to define new custom validations to be used in any form validation (es. creditCardValidator, emailValidator, passwordValidator ...).
9. Include logger customizable service in the core module to logs anything in the app.

## Shared Module:
1. Control message component created in the shared module to display custom messages in input controls depending of the validators and messages configured.
2. Login component displayed depending of the authentication service configured.
3. Not found component displayed when invalid routes.

## Get the Code
```
git clone https://github.com/yduartep/angular-full-sample.git
cd angular-full-sample
npm i
```

### Just in Time (JiT) Compilation

Runs the TypeScript compiler and launches the app

```
npm start
```

### Ahead of Time (AoT) Compilation 

Runs the Angular AoT compiler, rollup, uglify for an optimized bundle, then launches the app

```
npm run start-aot
```

### AoT + gzip 

Runs AoT plus gzips and launches the app 

```
gulp copy-aot-gzip
npm run aot
npm run rollup
http-server
```

Notes:
- Use your favorite server in place of `http-server`
- This could be scripted, obviously
- `lite-server` does not launch gzipped files by default.

# Functionalities

## 1. How create new lazy module
1. Install agular-cli globally: `npm install -g @angular/cli`
2. Create new module from the root of the app using command: `ng g module heroes`
3. Create new routing module in the folder of the module created before: `ng g module heroes-routing`
   - Define the routes and import them into the module as child routes: 
      ```
      export const heroesRoutes: Routes = <Routes>[{
        path: '',
        component: HeroesComponent,
        children: [
          { path: '', component: HeroListComponent },
          { path: 'detail/:id', component: HeroDetailComponent },
          { path: 'create', component: HeroCreateComponent }
        ]
      }];
      @NgModule({
        imports: [RouterModule.forChild(heroesRoutes)],
        exports: [RouterModule]
      })
      ```
   - Export the module `export class HeroesRoutingModule {}`
   - Export the list of components used `export const heroesRoutedComponents = [ MyComponent, ...];`.
4. Import the routing module and the routed components in the first module created:
   ```
   @NgModule({
      imports: [HeroesRoutingModule, ...],
      declarations: [heroesRoutedComponents]
   })
   export class HeroesModule { }
   ```
5. Add the new route /heroes to the list of menus defined in the file /assets/data/menu.data
   ```
   [{ "title": "Heroes", "action": "/heroes", "icon": "fa-flash"}, ... ]
   ```
6. Add the new route /heroes to the main routes defined in app-routing.module:
   ```
   const routes: Routes = [..., {path: 'heroes', loadChildren: 'app/heroes/heroes.module#HeroesModule', canActivate: [AuthGuard]},...];
   ```
More Info: https://angular-2-training-book.rangle.io/handout/modules/lazy-loading-module.html

## 2. How create new api config environment
1. Go to the folder environments and create a new file configuration: `environment.qa.ts`
2. Define the configuration to be used in this new environment:
   ```
   export const environment = {
      production: false,
      envName: 'qa',
      apiConfig: {
        apiEnv: 'qa',
        timeExpired: 5,
        credentials: {
          clientId: 'userClientId',
          clientSecret: 'pwdClient'
        },
        apiUrls: [
          { id: 'HEROES_SERVICE_URL', url: 'http://127.0.0.1:3000/api/heroes' },
          ...
          { id: 'OAUTH_SERVICE_URL', url: 'http://localhost:3000/api/oauth/token' }
        ],
      }
    };
   ```
   Note: If you want to use the OAuth Client during authentication, the ID of the url should be OAUTH_SERVICE_URL. Also, if your services use the basic authentication, define the property credentials like in the example below.
3. Edit the file .angular-cli.json, go to the environments property and add the new configuration:
   ```
   {
      "apps": [
         "environments": {
            ...,
            "qa": "environments/environment.qa.ts"
         }
      ]
   }
   ```
4. Serve the application using the new environment: `ng serve --environment=qa`

More Info: http://tattoocoder.com/angular-cli-using-the-environment-option/

## 3. How translate elements in a page
The application use the module ngx-translate for translation. The configuration of the TranslateModule is defined in the app.module.ts and exported from the SharedModule, so always be sure the module who expose the component to be translated import the SharedModule. The json files used during translation are stored in the folder /assets/i18n/ with the code of the specific language of translation (es. es.json, en.json, it.json ...).
   ```
   // app.translate.factory.ts
   export function createTranslateLoader(http: Http) {
      return new TranslateHttpLoader(http, '/assets/i18n/', '.json');
   }
   
   // app.module.ts
   @NgModule({
    imports: [..., 
      TranslateModule.forRoot({
        loader: {
          provide: TranslateLoader,
          useFactory: (createTranslateLoader),
          deps: [Http]
        }
      })
     ], ...
   })
   export class AppModule { }
   
   // shared/shared.module
   @NgModule({
      imports: [..., TranslateModule, ...],
      exports: [..., TranslateModule, ...]
   })
   export class SharedModule { }
   
   // heroes/heroes.module
   @NgModule({ imports: [SharedModule, ...] })
    export class HeroesModule { }
   ```
### i. Language initialization
When the app.component is instantiated set the following translation settings:
- Set the language to be used as a fallback when a translation isn't found in the current language based on the property 'defaultLanguage' defined in the current environment: `translate.setDefaultLang(defaultLanguage);`.
- Set the language to use, if the lang isn't available. First take the language defined in the localStorage and if no value was defined will use the default language defined before: `translate.use(localStorage['language'] || defaultLanguage);`

### ii. Word translations
Now to translate our words in the HTML view:
1. Create a pipe that we can use to translate our words: `<h2>{{ 'heroesList' | translate }}</h2>`
2. Add the translation ID used in our [language].json files:
   ```
   // assets/i18n/en.json
   { ..., "heroesList": "List of Super heroes", ...}
   
   // assets/i18n/es.json
   { ..., "heroesList": "Lista de Super heroes", ...}
   ```
3. Each time you change the language, that title will change.

### iii. How to use the language selector
In the core module, there is a 'language-selector' component that could be displayed anywhere to select the language for translation. On this sample application, was added in the header component.

The 'language-selector' component could be initialized in two ways:
1. With a list of custom languages: `<app-language-selector [languages]="languages"></app-language-selector>`
2. Without any predefined language: `<app-language-selector></app-language-selector>`

In the second case, the languages will loaded from the file 'assets/data/languages.json'. The format of a language is:
```
{
    "id": "en",
    "title": "English",    
    "icon": "en-EN.png"
}
```
The icon should be stored in the 'assets/images/flags' folder to be displayed correctly in the component.

### iv. How to define new language
1. Add new json file in the folder 'assets/i18n' and copy the content from other json file already defined. Example: `assets/i18n/fr.json`
2. Translate all the properties of the new file 'fr.json'.
3. Add new entry in the json file 'assets/data/languages.json':
   ```
   {
        "id": "fr",
        "title": "French",
        "icon": "fr-FR.png"
   }
   ```
4. Download and store the flag icon for French in the folder 'assets/images/flags'.
5. Restart the application

More Info: https://github.com/ngx-translate/core/blob/master/README.md

## 4. How to mock services
1. Go to the prototype folder out of the app and define the list of elements for the new api request (/heroes) in the file 'prototype/apiMocks.js'. You can also use the faker.js module to generate massive amounts of fake data:

   ```
   module.exports = function () {
    var faker = require("faker");
    return {
      heroes: [...]
    }
   }
   ```
2. If the path of the request is different as the default (default: http://localhost:3000/token, expected: http://localhost:3000/oauth/token), redefine the route in the file routes.json:
   ```
   { ..., "/oauth/token": "/token", ... }
   ```
3. Start the mock server using the command: `npm run server:mocks`

More Info:
https://github.com/typicode/json-server
https://scotch.io/tutorials/json-server-as-a-fake-rest-api-in-frontend-development
https://github.com/marak/Faker.js/

## 5. Bootstrap and Font-awsone integration
The application has already installed the library 'font-awesome' and 'bootstrap', so you can create new responsive components with a pack of pre-defined icons. The application also incorporate the library 'ngx-bootstrap' that contains many modules like accordion, alerts, datepicker, progressbar, etc, that could be imported separately in the case you need it. See how to use it from http://valor-software.com/ngx-bootstrap/#/.

## 6. Dynamic nav bar
The application contains a 'nav' component in the core module that could be initialized in two ways:
1. With a list of custom menu items: `<app-nav [items]="menuData"></app-nav>`
2. Without any predefined menu: `<app-nav></app-nav>`

In the second case, the menus will loaded from the file 'assets/data/menu.json'. The format of a menu is:
```
{
    "title": "Heroes",
    "action": "/heroes",
    "icon": "fa-flash"
}
```
The icon is a font-awsone icon. See some example from http://fontawesome.io/examples/.

## 7. Http interceptor system
The application include an 'Http Interceptor' used to capture HTTP errors, authentication and show loading after any HTTP requests. The class is defined as a provider of the shared module (/shared/http.interceptor.ts), in that way, will be imported for each new module automatically. 

## 8. Login component configurable with different authentication service
The application include a login component that could be integrated with any authentication service that implements the interface AuthService present in the core module. The application already includes:

- OAuth Service client to be integrated with an OAuth2 server (/core/services/oauth.service.ts).
- Skyp Service that allow to display the application without to use the login component (/core/services/skyp-auth.service.ts).

### How to create new Authentication service
1. Create new class that implement the interface AuthService. The BaseAuthService is an abstract class that contains some util functionalities like adding or removing user and token info from and to the cookie.
    ```
    @Injectable()
    export class LDAPService extends BaseAuthService implements AuthService {...}
    ```
 2. Go to the authFactory function defined in the core module in '/core/factories/auth.factory.ts' and replace the instance created using the new class:
    ```
    export function authFactory(http: Http): AuthService {
      return new LDAPService(http, ...);
    }
    ```
 3. Restart the application
