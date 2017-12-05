# AngularJS styleguide (ES2015)

### Up-to-date with AngularJS 1.6 best practices. Architecture, file structure, components, one-way dataflow, lifecycle hooks.

---

> Want an example structure as reference? Check out my [component based architecture 1.5 app](https://github.com/toddmotto/angular-1-5-components-app).

---

*A sensible styleguide for teams by [@toddmotto](//twitter.com/toddmotto)*

This architecture and styleguide has been rewritten from the ground up for ES2015, the changes in AngularJS 1.5+ for future-upgrading your application to Angular. This guide includes new best practices for one-way dataflow, event delegation, component architecture and component routing.

You can find the old styleguide [here](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), and the reasoning behind the new one [here](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Join the Ultimate AngularJS learning experience to fully master beginner and advanced AngularJS features to build real-world apps that are fast, and scale.

<a href="https://ultimateangular.com" target="_blank"><img src="https://ultimateangular.com/assets/img/banner.jpg"></a>

## Table of Contents

1. [Modular architecture](#modular-architecture)
    1. [Theory](#module-theory)
    1. [Root module](#root-module)
    1. [Framework module](#framework-module)
    1. [Common module](#common-module)
    1. [Feature modules](#feature-modules)
    1. [File naming conventions](#file-naming-conventions)
    1. [Scalable file structure](#scalable-file-structure)
1. [Components](#components)
    1. [Theory](#component-theory)
    1. [Supported properties](#supported-properties)
    1. [Namespacing](#namespacing)
    1. [Controllers](#controllers)
    1. [One-way dataflow and Events](#one-way-dataflow-and-events)
    1. [Stateful Components](#stateful-components)
    1. [Stateless Components](#stateless-components)
    1. [Routed Components](#routed-components)
1. [Directives](#directives)
    1. [Theory](#directive-theory)
    1. [Recommended properties](#recommended-properties)
    1. [Constants or Classes](#constants-or-classes)
1. [Services](#services)
    1. [Theory](#service-theory)
    1. [Classes for Service](#classes-for-service)
1. [Routes](#routes)
1. [Styles](#styles)
1. [ES2015 and Tooling](#es2015-and-tooling)
1. [State management](#state-management)
1. [Resources](#resources)
1. [Documentation](#documentation)
1. [Contributing](#contributing)

# Modular architecture

Each module in an Angular app is a module component. A module component is the root definition for that module that encapsulates the logic, templates, routing and child components.

### Module theory

The design in the modules maps directly to our folder structure, which keeps things maintainable and predictable. The root module defines the base module that bootstraps our app, and the corresponding template. We then import our framework, common, and feature modules into the root module to include our dependencies.

**[Back to top](#table-of-contents)**

### Root module

A root module begins with a root component that defines the base element for the entire application, with a routing outlet defined, example shown using `ui-view` from `ui-router`.

```js
// app.component.js
export const AppComponent = {
  template: `
    <header>
        Hello world
    </header>
    <div>
        <div ui-view></div>
    </div>
    <footer>
        Copyright MyApp 2016.
    </footer>
  `
};
```

A root module is then created, with `MyupmcComponent` imported and registered with `.component('myupmc', MyupmcComponent)`. Further imports for submodules (framework, common, and feature modules) are made to include all components relevant for the application.

```js
// myupmc.module.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { MyupmcComponent } from './myupmc.component';
import { FrameworkModule } from './_framework/framework.module';
import { CommonModule } from './_common/common.module';
import { AppointmentsModule } from './appointments/appointments.module';

export const MyupmcModule = angular
  .module('myupmc', [
    uiRouter,
    FrameworkModule,
    CommonModule,
    AppointmentsModule
  ])
  .component('myupmc', MyupmcComponent)
  .name;
```

**[Back to top](#table-of-contents)**

### Framework module

A Framework module is the container reference for all reusable components and services. Items in Framework should contain no business logic.  See above how we import `FrameworkModule` and inject them into the Root module, this gives us a single place to import all Framework components and services for the app. These modules we require are decoupled from all other modules and thus can be moved into any other application with ease.

```js
import angular from 'angular';
import { NotificationModule } from './notification/notification.module';
import { IconModule } from './icon/icon.module';

export const FrameworkModule = angular
  .module('myupmc.framework', [
    NotificationModule,
    IconModule
  ])
  .name;
```

**[Back to top](#table-of-contents)**

### Common module

The Common module is the container reference for all application specific components, that we don't want to use in another application. This can be things like layout, navigation and footers. See above how we import `CommonModule` and inject them into the Root module, this gives us a single place to import all common components for the app.

```js
import angular from 'angular';
import { AllergyModule } from './Allergy/Allergy.module';

export const CommonModule = angular
  .module('app.common', [
    AllergyModule
  ])
  .name;
```

**[Back to top](#table-of-contents)**

### Feature modules

Feature modules are modules that contain the logic for each feature block. These will each define a module, to be imported to a higher-level module, either to another feature module, or to the root module. Always remember to add the `.name` suffix to each `export` when creating a _new_ module, not when referencing one. You'll noticed routing definitions also exist here, we'll come onto this in later chapters in this guide.

```js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { AppointmentsComponent } from './appointments.component';
import { AppointmentsService } from './appointments.service';
import { AppointmentsRoute } from './appointments.route';

export const AppointmentsModule = angular
  .module('myupmc.appointments', [
    uiRouter
  ])
  .component('appointments', AppointmentsComponent)
  .service('appointmentsService', AppointmentsService)
  .config(AppointmentsRoute)
  .name;
```

**[Back to top](#table-of-contents)**

### File naming conventions

Keep it simple and lowerCamelCase, use the component name, e.g. `calendar.*.js*`, `calendarGrid.*.js` - with the name of the type of file in the middle. Use `*.module.js` for the module definition file, as it keeps it verbose and consistent with Angular.

```
calendar.module.js
calendar.component.js
calendar.component.spec.js
calendar.service.js
calendar.service.spec.js
calendar.directive.js
calendar.directive.spec.js
calendar.template.jade
calendar.route.js
calendar.scss
```

**[Back to top](#table-of-contents)**
### Scalable file structure

File structure is extremely important, this describes a scalable and predictable structure. An example file structure to illustrate a modular component architecture.

```
├── app/
│   ├── _common/
│   │  ├── allergy/
│   │  │  ├── allergy.module.js
│   │  │  ├── allergy.component.js
│   │  │  ├── allergy.component.spec.js
│   │  │  ├── allergy.service.js
│   │  │  ├── allergy.service.spec.js
│   │  │  ├── allergy.template.jade
│   │  │  ├── allergy.scss
│   │  │  └── allergy.scss
│   │  └── common.module.js
│   ├── _framework/
│   │  ├── components/
│   │  │     └── notification/
│   │  │        ├── notification.module.js
│   │  │        ├── notification.component.js
│   │  │        ├── notification.component.spec.js
│   │  │        ├── notification.service.js
│   │  │        ├── notification.service.spec.js
│   │  │        ├── notification.template.jade
│   │  │        └── notification.scss
│   │  ├── services/
│   │  │     └── cache/
│   │  │        ├── cache.module.js
│   │  │        ├── cache.service.js
│   │  │        └── cache.service.spec.js
│   │  └── framework.module.js
│   ├── appointments/
│   │  ├── make/
│   │  │     ├── make.module.js
│   │  │     ├── make.component.js
│   │  │     ├── make.component.spec.js
│   │  │     ├── make.service.js
│   │  │     ├── make.service.spec.js
│   │  │     ├── make.template.jade
│   │  │     ├── make.route.js
│   │  │     └── make.scss
│   │  ├── details/
│   │  │     ├── details.module.js
│   │  │     ├── details.component.spec.js
│   │  │     ├── details.component.js
│   │  │     ├── details.service.js
│   │  │     ├── details.service.spec.js
│   │  │     ├── details.template.jade
│   │  │     ├── details.route.js
│   │  │     └── details.scss
│   │  ├── appointments.module.js
│   │  ├── appointments.component.spec.js
│   │  ├── appointments.component.js
│   │  ├── appointments.service.js
│   │  ├── appointments.service.spec.js
│   │  ├── appointments.template.jade
│   │  ├── appointments.route.js
│   │  └── appointments.scss
│   ├── myupmc.module.js
│   ├── myupmc.component.spec.js
│   ├── myupmc.component.js
│   ├── myupmc.service.js
│   ├── myupmc.service.spec.js
│   ├── myupmc.template.jade
│   ├── myupmc.route.js
│   └── myupmc.scss
└── assets/
```

The high level folder structure `app/`, for all first-party scripts, and `assets/` for 3rd party scripts and other assets such as images.

**[Back to top](#table-of-contents)**

# Components

### Component theory

Components are essentially templates with a controller. They are _not_ Directives, nor should you replace Directives with Components, unless you are upgrading "template Directives" with controllers, which are best suited as a component. Components also contain bindings that define inputs and outputs for data and events, lifecycle hooks and the ability to use one-way data flow and event Objects to get data back up to a parent component. These are the new defacto standard in AngularJS 1.5 and above. Everything template and controller driven that we create will likely be a component, which may be a stateful, stateless or routed component. You can think of a "component" as a complete piece of code, not just the `.component()` definition Object. Let's explore some best practices and advisories for components, then dive into how you should be structuring them via stateful, stateless and routed component concepts.

**[Back to top](#table-of-contents)**

### Supported properties

These are the supported properties for `.component()` that you can/should use:

| Property | Support |
|---|---|
| bindings | Yes, use `'@'`, `'<'`, `'&'` only |
| controller | Yes |
| controllerAs | Yes, default is `$ctrl` |
| require | Yes (new Object syntax) |
| template | Yes |
| templateUrl | Yes |
| transclude | Yes |

**[Back to top](#table-of-contents)**

### Namespacing

Components need to be namespaced to prevent multiple components from having the same name.  All components should begin with `mgl-*`.  Components in feature modules should add additional namespacing.  For example, the `make` component in the `myupmc.appointments.make` module should be named `mgl-appts-make`, with `-appts-` being added since its module is a child of the `myupmc.appointments` module.

### Controllers

Controllers should only be used alongside components, never anywhere else. If you feel you need a controller, what you really need is likely a stateless component to manage that particular piece of behaviour.

Here are some advisories for using `Class` for controllers:

* Drop the name "Controller", i.e. use `controller: class TodoComponent {...}` to aid future Angular migration
* Always use the `constructor` for dependency injection purposes
* Use [ng-annotate](https://github.com/olov/ng-annotate)'s `'ngInject';` syntax for `$inject` annotations
* If you need to access the lexical scope, use arrow functions
* Alternatively to arrow functions, `let ctrl = this;` is also acceptable and may make more sense depending on the use case
* Bind all public functions directly to the `Class`
* Make use of the appropriate lifecycle hooks, `$onInit`, `$onChanges`, `$postLink` and `$onDestroy`
  * Note: `$onChanges` is called before `$onInit`, see [resources](#resources) section for articles detailing this in more depth
* Use `require` alongside `$onInit` to reference any inherited logic
* Do not override the default `$ctrl` alias for the `controllerAs` syntax, therefore do not use `controllerAs` anywhere

**[Back to top](#table-of-contents)**

### One-way dataflow and Events

One-way dataflow was introduced in AngularJS 1.5, and redefines component communication.

Here are some advisories for using one-way dataflow:

* In components that receive data, always use one-way databinding syntax `'<'`
* _Do not_ use `'='` two-way databinding syntax anymore, anywhere
* Components that have `bindings` should use `$onChanges` to clone the one-way binding data to break Objects passing by reference and updating the parent data
* Use `$event` as a function argument in the parent method (see stateful example below `$ctrl.addTodo($event)`)
* Pass an `$event: {}` Object back up from a stateless component (see stateless example below `this.onAddTodo`).
  * Bonus: Use an `EventEmitter` wrapper with `.value()` to mirror Angular, avoids manual `$event` Object creation
* Why? This mirrors Angular and keeps consistency inside every component. It also makes state predictable.

**[Back to top](#table-of-contents)**

### Stateful components

Let's define what we'd call a "stateful component".

* Fetches state, essentially communicating to a backend API through a service
* Does not directly mutate state
* Renders child components that mutate state
* Also referred to as smart/container components

An example of a stateful component, complete with its low-level module definition (this is only for demonstration, so some code has been omitted for brevity):

```js
/* ----- todo/todo.component.js ----- */
import templateUrl from './todo.html';

export const TodoComponent = {
  templateUrl,
  controller: class TodoComponent {
    constructor(TodoService) {
      'ngInject';
      this.todoService = TodoService;
    }
    $onInit() {
      this.newTodo = {
        title: '',
        selected: false
      };
      this.todos = [];
      this.todoService.getTodos().then(response => this.todos = response);
    }
    addTodo({ todo }) {
      if (!todo) return;
      this.todos.unshift(todo);
      this.newTodo = {
        title: '',
        selected: false
      };
    }
  }
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;
```

This example shows a stateful component, that fetches state inside the controller, through a service, and then passes it down into stateless child components. Notice how there are no Directives being used such as `ng-repeat` and friends inside the template. Instead, data and functions are delegated into `<todo-form>` and `<todo-list>` stateless components.

**[Back to top](#table-of-contents)**

### Stateless components

Let's define what we'd call a "stateless component".

* Has defined inputs and outputs using `bindings: {}`
* Data enters the component through attribute bindings (inputs)
* Data leaves the component through events (outputs)
* Mutates state, passes data back up on-demand (such as a click or submit event)
* Doesn't care where data comes from - it's stateless
* Are highly reusable components
* Also referred to as dumb/presentational components

An example of a stateless component (let's use `<todo-form>` as an example), complete with its low-level module definition (this is only for demonstration, so some code has been omitted for brevity):

```js
/* ----- todo/todo-form/todo-form.component.js ----- */
import templateUrl from './todo-form.html';

export const TodoFormComponent = {
  bindings: {
    todo: '<',
    onAddTodo: '&'
  },
  templateUrl,
  controller: class TodoFormComponent {
    constructor(EventEmitter) {
        'ngInject';
        this.EventEmitter = EventEmitter;
    }
    $onChanges(changes) {
      if (changes.todo) {
        this.todo = Object.assign({}, this.todo);
      }
    }
    onSubmit() {
      if (!this.todo.title) return;
      // with EventEmitter wrapper
      this.onAddTodo(
        this.EventEmitter({
          todo: this.todo
        })
      );
      // without EventEmitter wrapper
      this.onAddTodo({
        $event: {
          todo: this.todo
        }
      });
    }
  }
};

/* ----- todo/todo-form/todo-form.html ----- */
<form name="todoForm" ng-submit="$ctrl.onSubmit();">
  <input type="text" ng-model="$ctrl.todo.title">
  <button type="submit">Submit</button>
</form>

/* ----- todo/todo-form/todo-form.module.js ----- */
import angular from 'angular';
import { TodoFormComponent } from './todo-form.component';
import './todo-form.scss';

export const TodoFormModule = angular
  .module('todo.form', [])
  .component('todoForm', TodoFormComponent)
  .value('EventEmitter', payload => ({ $event: payload }))
  .name;
```

Note how the `<todo-form>` component fetches no state, it simply receives it, mutates an Object via the controller logic associated with it, and passes it back to the parent component through the property bindings. In this example, the `$onChanges` lifecycle hook makes a clone of the initial `this.todo` binding Object and reassigns it, which means the parent data is not affected until we submit the form, alongside one-way data flow new binding syntax `'<'`.

**[Back to top](#table-of-contents)**


# Directives

### Directive theory

Directives gives us `template`, `scope` bindings, `bindToController`, `link` and many other things. The usage of these should be carefully considered now that `.component()` exists. Directives should not declare templates and controllers anymore, or receive data through bindings. Directives should be used solely for decorating the DOM. By this, it means extending existing HTML - created with `.component()`. In a simple sense, if you need custom DOM events/APIs and logic, use a Directive and bind it to a template inside a component. If you need a sensible amount of DOM manipulation, there is also the `$postLink` lifecycle hook to consider, however this is not a place to migrate all your DOM manipulation to, use a Directive if you can for non-Angular things.

Here are some advisories for using Directives:

* Never use templates, scope, bindToController or controllers
* Always `restrict: 'A'` with Directives
* Use compile and link where necessary
* Remember to destroy and unbind event handlers inside `$scope.$on('$destroy', fn);`

**[Back to top](#table-of-contents)**

### Recommended properties

Due to the fact directives support most of what `.component()` does (template directives were the original component), I'm recommending limiting your directive Object definitions to only these properties, to avoid using directives incorrectly:

| Property | Use it? | Why |
|---|---|---|
| bindToController | No | Use `bindings` in components |
| compile | Yes | For pre-compile DOM manipulation/events |
| controller | No | Use a component |
| controllerAs | No | Use a component |
| link functions | Yes | For pre/post DOM manipulation/events |
| multiElement | Yes | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Yes | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | No | Use a component |
| restrict | Yes | Defines directive usage, always use `'A'` |
| scope | No | Use a component |
| template | No | Use a component |
| templateNamespace | Yes (if you must) | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | No | Use a component |
| transclude | No | Use a component |

**[Back to top](#table-of-contents)**

### Constants

There are a few ways to approach using ES2015 and directives, either with an arrow function and easier assignment, or using an ES2015 `Class`. We will use arrow functions.

Here's an example using a constant with an Arrow function an expression wrapper `() => ({})` returning an Object literal (note the usage differences inside `.directive()`):

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

export const TodoAutoFocus = ($timeout) => {
  'ngInject';
  return {
    restrict: 'A',
    link($scope, $element, $attrs) {
      $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
        if (!newValue) {
          return;
        }
        $timeout(() => $element[0].focus());
      });
    }
  }
};

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;
```

# Services

### Service theory

Services are essentially containers for business logic that our components shouldn't request directly. Services contain other built-in or external services such as `$http`, that we can then inject into component controllers elsewhere in our app. We have two ways of doing services, using `.service()` or `.factory()`. With ES2015 `Class`, we should only use `.service()`, complete with dependency injection annotation using `$inject`.

**[Back to top](#table-of-contents)**

### Classes for Service

Here's an example implementation for our `<todo>` app using ES2015 `Class`:

```js
/* ----- todo/todo.service.js ----- */
export class TodoService {
  constructor($http) {
    'ngInject';
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;
```

**[Back to top](#table-of-contents)**

# Routes

Each feature module should define a route, rather than defining all routes at the top-level feature module.  Like directives, the route should be exported as a fat arrow constant

```js
/* ----- appointments/appointments.route.js ----- */

export const AppointmentsRoute = ($stateProvider) => {
  'ngInject';

  $stateProvider.state('appointments', {
    url: '/appointments',
    component: 'appointments'
  });
};

/* ----- appointments/appointments.module.js ----- */

import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { AppointmentsComponent } from './appointments.component';
import { AppointmentsService } from './appointments.service';
import { AppointmentsRoute } from './appointments.route';

export const AppointmentsModule = angular
  .module('myupmc.appointments', [
    uiRouter
  ])
  .component('appointments', AppointmentsComponent)
  .service('appointmentsService', AppointmentsService)
  .config(AppointmentsRoute)
  .name;
```

# Styles

TODO

**[Back to top](#table-of-contents)**

# ES2015 and Tooling

##### ES2015

* Use [Babel](https://babeljs.io/) to compile your ES2015+ code and any polyfills
* Consider using [TypeScript](http://www.typescriptlang.org/) to make way for any Angular upgrades

##### Tooling
* Use `ui-router` [latest alpha](https://github.com/angular-ui/ui-router) (see the Readme) if you want to support component-routing
  * Otherwise you're stuck with `template: '<component>'` and no `bindings`/resolve mapping
* Consider preloading templates into `$templateCache` with `angular-templates` or `ngtemplate-loader`
  * [Gulp version](https://www.npmjs.com/package/gulp-angular-templatecache)
  * [Grunt version](https://www.npmjs.com/package/grunt-angular-templates)
  * [Webpack version](https://github.com/WearyMonkey/ngtemplate-loader)
* Consider using [Webpack](https://webpack.github.io/) for compiling your ES2015 code and styles
* Use [ngAnnotate](https://github.com/olov/ng-annotate) to automatically annotate `$inject` properties
* How to use [ngAnnotate with ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[Back to top](#table-of-contents)**

# State management

Consider using Redux with AngularJS 1.5 for data management.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Back to top](#table-of-contents)**

# Resources

* [Stateful and stateless components, the missing manual](https://toddmotto.com/stateful-stateless-components)
* [Understanding the .component() method](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Using "require" with $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Understanding all the lifecycle hooks, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Using "resolve" in routes](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux and Angular state management](http://blog.rangle.io/managing-state-redux-angular/)
* [Sample Application from Community](https://github.com/chihab/angular-styleguide-sample)

**[Back to top](#table-of-contents)**

# Documentation
For anything else, including API reference, check the [AngularJS documentation](//docs.angularjs.org/api).

# Contributing

Open an issue first to discuss potential changes/additions. Please don't open issues for questions.

## License

#### (The MIT License)

Copyright (c) 2016-2017 Todd Motto

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
