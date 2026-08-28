# AngularJS Notes — Basics to Advanced

> Covers **AngularJS (1.x)** — the original Google framework (`ng-app`, `$scope`, `angular.module`).
> Note: Angular 2+ ("Angular") is a complete rewrite in TypeScript with a different syntax — if you meant that, ask separately.

---

## Table of Contents
1. Introduction & Core Concepts
2. Setup & First App
3. Directives
4. Expressions & Data Binding
5. Modules & Controllers
6. Scope (`$scope`)
7. Filters
8. Services & Dependency Injection
9. Custom Directives
10. Routing (`ngRoute` / `ui-router`)
11. Forms & Validation
12. `$http` and Promises
13. Component-Based Architecture (`.component()`)
14. Interceptors
15. Animations (`ngAnimate`)
16. Testing Basics
17. Best Practices & Interview Q&A

---

## 1. Introduction & Core Concepts

**Definition:** AngularJS is an open-source, client-side **JavaScript MVC (Model-View-Controller)** framework developed by Google (2010) for building **Single Page Applications (SPAs)**. It extends HTML with new attributes (directives) and binds data between the Model and the View automatically.

### Key Features
| Feature | Description |
|---|---|
| **Two-Way Data Binding** | Automatic sync between Model (JS data) and View (HTML DOM) |
| **MVC Architecture** | Separates Model, View, and Controller for maintainability |
| **Directives** | Extend HTML with custom attributes/elements (`ng-model`, `ng-repeat`, etc.) |
| **Dependency Injection (DI)** | Built-in DI system for services, factories, controllers |
| **Testability** | Designed with unit testing (Jasmine/Karma) in mind |
| **Templating** | HTML is the templating language itself |
| **Routing** | Enables SPA navigation without full page reloads |

### MVC in AngularJS
- **Model** → Data (JavaScript objects on `$scope`)
- **View** → HTML template rendered in the browser
- **Controller** → JS function that sets up `$scope` data/behavior

---

## 2. Setup & First App

### Include via CDN
```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular.min.js"></script>
</head>
<body ng-app="myFirstApp">
  <div ng-controller="HelloController">
    <input type="text" ng-model="name" placeholder="Enter your name">
    <h1>Hello, {{ name }}!</h1>
  </div>

  <script>
    var app = angular.module('myFirstApp', []);
    app.controller('HelloController', function ($scope) {
      $scope.name = 'World';
    });
  </script>
</body>
</html>
```

**Explanation:**
- `ng-app="myFirstApp"` — bootstraps/initializes the AngularJS application on that DOM element.
- `ng-controller="HelloController"` — attaches a controller (and its `$scope`) to that section of the DOM.
- `{{ name }}` — interpolation/expression binding that displays the scope variable.
- `ng-model="name"` — binds the input value to `$scope.name` (two-way binding).

---

## 3. Directives

**Definition:** Directives are markers on DOM elements (attributes, tags, CSS classes) that tell AngularJS to attach specific behavior or transform the DOM.

### Common Built-in Directives

| Directive | Purpose | Example |
|---|---|---|
| `ng-app` | Bootstraps the app | `<html ng-app="app">` |
| `ng-controller` | Attaches a controller | `<div ng-controller="Ctrl">` |
| `ng-model` | Two-way binds form input to scope | `<input ng-model="user">` |
| `ng-bind` | One-way binds text (like `{{ }}`) | `<span ng-bind="name"></span>` |
| `ng-repeat` | Loops over an array/collection | see below |
| `ng-if` | Conditionally adds/removes DOM element | `<p ng-if="isVisible">Hi</p>` |
| `ng-show` / `ng-hide` | Toggles CSS `display` (element stays in DOM) | `<p ng-show="loggedIn">` |
| `ng-class` | Conditionally applies CSS classes | `<div ng-class="{active: isActive}">` |
| `ng-click` | Binds a click event handler | `<button ng-click="save()">` |
| `ng-disabled` | Conditionally disables an element | `<button ng-disabled="isLoading">` |
| `ng-src` | Safe binding for `src` attribute | `<img ng-src="{{imgUrl}}">` |

### `ng-repeat` Example
```html
<ul>
  <li ng-repeat="student in students track by student.id">
    {{ $index + 1 }}. {{ student.name }} — {{ student.marks }}
  </li>
</ul>
```
```js
app.controller('ListController', function ($scope) {
  $scope.students = [
    { id: 1, name: 'Aakash', marks: 88 },
    { id: 2, name: 'Harshada', marks: 92 },
    { id: 3, name: 'Rohan', marks: 76 }
  ];
});
```
`ng-repeat` exposes special variables inside each iteration: `$index`, `$first`, `$last`, `$middle`, `$even`, `$odd`.

### `ng-if` vs `ng-show`
- `ng-if` — **removes/adds** the element from the DOM entirely (creates/destroys scope). Better for expensive elements.
- `ng-show`/`ng-hide` — element **stays in the DOM**, only `display: none` is toggled via CSS. Faster for frequent toggling.

---

## 4. Expressions & Data Binding

**Definition:** Expressions are JavaScript-like code snippets written inside `{{ }}` that AngularJS evaluates against the current `$scope`.

```html
<p>{{ 5 + 5 }}</p>              <!-- 10 -->
<p>{{ 'Hello ' + name }}</p>    <!-- string concatenation -->
<p>{{ user.age > 18 ? 'Adult' : 'Minor' }}</p>
```

### Two-Way Data Binding
**Definition:** Changes in the View (UI) automatically update the Model (`$scope`), and changes in the Model automatically update the View — kept in sync via Angular's **digest cycle** and **dirty checking**.

```html
<input type="text" ng-model="price">
<p>Total with tax: {{ price * 1.18 | number:2 }}</p>
```

**How it works internally:** AngularJS maintains a **watch list**. On every event (click, input, HTTP response), it triggers a **`$digest` loop** that re-evaluates all watched expressions and updates the DOM only where values changed ("dirty checking").

---

## 5. Modules & Controllers

**Definition (Module):** A module is a container for the different parts of an app (controllers, services, directives, filters, config). It's the entry point that groups related functionality.

```js
// Defining a module (2nd argument = array of dependencies)
var app = angular.module('schoolApp', ['ngRoute', 'ngAnimate']);

// Retrieving an already-defined module (no array argument)
var sameApp = angular.module('schoolApp');
```

**Definition (Controller):** A JS constructor function that initializes `$scope` with data and behavior for a specific view/section.

```js
app.controller('StudentController', function ($scope) {
  $scope.title = 'Student Dashboard';
  $scope.marksList = [70, 85, 90];

  $scope.average = function () {
    var sum = $scope.marksList.reduce(function (a, b) { return a + b; }, 0);
    return (sum / $scope.marksList.length).toFixed(2);
  };
});
```

### `controller as` Syntax (recommended — avoids `$scope` pollution)
```html
<div ng-controller="StudentController as vm">
  <h2>{{ vm.title }}</h2>
  <p>Average: {{ vm.average() }}</p>
</div>
```
```js
app.controller('StudentController', function () {
  var vm = this;
  vm.title = 'Student Dashboard';
  vm.marksList = [70, 85, 90];
  vm.average = function () {
    var sum = vm.marksList.reduce(function (a, b) { return a + b; }, 0);
    return (sum / vm.marksList.length).toFixed(2);
  };
});
```

---

## 6. Scope (`$scope`)

**Definition:** `$scope` is an object that refers to the application model. It acts as the **glue between the Controller and the View** — anything attached to `$scope` becomes accessible in the template.

### Scope Hierarchy
- AngularJS scopes are arranged in a **hierarchical tree** mirroring the DOM.
- Child scopes **prototypically inherit** properties from parent scopes (unless shadowed).

```html
<div ng-controller="ParentController">
  <p>{{ message }}</p>          <!-- Parent's message -->
  <div ng-controller="ChildController">
    <p>{{ message }}</p>        <!-- Inherits parent's message unless overridden -->
  </div>
</div>
```

### `$rootScope`
The topmost scope of the application; every other scope is a descendant of it. Useful for app-wide values, but overuse is an anti-pattern (creates hidden global state).

```js
app.run(function ($rootScope) {
  $rootScope.appName = 'EduConnect Portal';
});
```

### `$watch`, `$digest`, `$apply`
```js
$scope.$watch('username', function (newVal, oldVal) {
  console.log('Changed from', oldVal, 'to', newVal);
});
```
- `$digest` — re-evaluates all watchers in the current scope + children.
- `$apply` — used to manually bring external/async code (e.g., jQuery, `setTimeout`) into Angular's digest cycle.

```js
setTimeout(function () {
  $scope.$apply(function () {
    $scope.message = 'Updated from outside Angular';
  });
}, 1000);
```

---

## 7. Filters

**Definition:** Filters format the value of an expression for display without changing the underlying data.

```html
<p>{{ 'angularjs' | uppercase }}</p>          <!-- ANGULARJS -->
<p>{{ price | currency:'₹' }}</p>             <!-- ₹1,234.00 -->
<p>{{ today | date:'dd-MM-yyyy' }}</p>
<p>{{ marks | number:1 }}</p>
<input ng-model="search">
<li ng-repeat="s in students | filter:search | orderBy:'name'">{{ s.name }}</li>
```

### Custom Filter
```js
app.filter('capitalize', function () {
  return function (input) {
    if (!input) return '';
    return input.charAt(0).toUpperCase() + input.slice(1);
  };
});
```
```html
<p>{{ 'aakash' | capitalize }}</p>  <!-- Aakash -->
```

---

## 8. Services & Dependency Injection

**Definition (DI):** Dependency Injection is a design pattern where components (controllers, services) declare their dependencies, and Angular's **injector** supplies them at runtime instead of the component creating them itself.

```js
app.controller('DemoController', function ($scope, $http, $timeout) {
  // $scope, $http, $timeout are injected automatically by name
});
```

### Built-in Services
| Service | Purpose |
|---|---|
| `$http` | Makes AJAX/HTTP requests |
| `$q` | Promise library |
| `$timeout` | Angular-aware `setTimeout` |
| `$interval` | Angular-aware `setInterval` |
| `$location` | Reads/manipulates browser URL |
| `$route` | Route configuration info |

### Creating a Custom Service (`.service()`)
```js
app.service('StudentService', function ($http) {
  this.getStudents = function () {
    return $http.get('/api/students');
  };
});
```

### `.factory()` — more flexible, returns any object/function
```js
app.factory('MathFactory', function () {
  return {
    square: function (x) { return x * x; },
    cube: function (x) { return x * x * x; }
  };
});
```

### `.value()` and `.constant()`
```js
app.value('appVersion', '1.0.0');
app.constant('API_BASE_URL', 'https://api.example.com'); // available even in .config()
```

**`service` vs `factory` vs `value`/`constant`:**
- `service` — instantiated with `new`; good for class-like objects.
- `factory` — returns whatever the function returns; most flexible/common.
- `value`/`constant` — plain fixed values; `constant` can be injected into `.config()`, `value` cannot.

---

## 9. Custom Directives

**Definition:** Beyond built-ins, developers can create their **own reusable directives** to encapsulate DOM behavior/markup.

```js
app.directive('studentCard', function () {
  return {
    restrict: 'E',          // E = Element, A = Attribute, C = Class, M = Comment
    scope: {                // isolated scope — receives data via bindings
      student: '='          // two-way binding
    },
    template:
      '<div class="card">' +
        '<h3>{{ student.name }}</h3>' +
        '<p>Marks: {{ student.marks }}</p>' +
      '</div>'
  };
});
```
```html
<student-card student="s" ng-repeat="s in students"></student-card>
```

### Isolated Scope Binding Symbols
| Symbol | Meaning |
|---|---|
| `=` | Two-way data binding |
| `@` | One-way string binding (reads attribute as text/interpolated string) |
| `&` | Binds an expression/function from the parent scope |

```js
app.directive('greetBox', function () {
  return {
    restrict: 'E',
    scope: { name: '@', onGreet: '&' },
    template: '<button ng-click="onGreet()">Hello, {{ name }}</button>'
  };
});
```
```html
<greet-box name="Aakash" on-greet="sayHi()"></greet-box>
```

### `link` Function (direct DOM manipulation)
```js
app.directive('highlightOnHover', function () {
  return {
    restrict: 'A',
    link: function (scope, element, attrs) {
      element.on('mouseenter', function () {
        element.css('background-color', 'yellow');
      });
      element.on('mouseleave', function () {
        element.css('background-color', '');
      });
    }
  };
});
```

---

## 10. Routing (SPA Navigation)

**Definition:** Routing lets an AngularJS app switch between different "pages" (views/templates) without a full browser reload, by mapping URL paths to controller+template pairs.

### Using `ngRoute`
```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular-route.min.js"></script>
```
```js
var app = angular.module('routedApp', ['ngRoute']);

app.config(function ($routeProvider) {
  $routeProvider
    .when('/', {
      templateUrl: 'home.html',
      controller: 'HomeController'
    })
    .when('/students', {
      templateUrl: 'students.html',
      controller: 'StudentController'
    })
    .when('/students/:id', {
      templateUrl: 'student-detail.html',
      controller: 'StudentDetailController'
    })
    .otherwise({ redirectTo: '/' });
});
```
```html
<body>
  <a href="#!/">Home</a>
  <a href="#!/students">Students</a>
  <div ng-view></div>  <!-- matched template renders here -->
</body>
```

```js
app.controller('StudentDetailController', function ($scope, $routeParams) {
  $scope.studentId = $routeParams.id;  // reads the :id from URL
});
```

### `ui-router` (more powerful, state-based — common in real projects)
```js
var app = angular.module('routedApp', ['ui.router']);

app.config(function ($stateProvider, $urlRouterProvider) {
  $urlRouterProvider.otherwise('/');
  $stateProvider
    .state('home', { url: '/', templateUrl: 'home.html' })
    .state('students', { url: '/students', templateUrl: 'students.html', controller: 'StudentController' })
    .state('students.detail', { url: '/:id', templateUrl: 'detail.html' }); // nested state
});
```

---

## 11. Forms & Validation

**Definition:** AngularJS enhances native HTML forms with live validation state tracking via `FormController` and `NgModelController`.

```html
<form name="regForm" ng-submit="submitForm()" novalidate>
  <input type="text" name="username" ng-model="user.name" required minlength="3">
  <span ng-show="regForm.username.$error.required && regForm.username.$touched">
    Name is required.
  </span>
  <span ng-show="regForm.username.$error.minlength">
    Minimum 3 characters.
  </span>

  <input type="email" name="email" ng-model="user.email" required>
  <span ng-show="regForm.email.$error.email">Invalid email.</span>

  <button type="submit" ng-disabled="regForm.$invalid">Register</button>
</form>
```

### Useful Form/Field States
| Property | Meaning |
|---|---|
| `$valid` / `$invalid` | Is the form/field currently valid? |
| `$pristine` / `$dirty` | Has the user not touched / touched the field? |
| `$touched` / `$untouched` | Has the field lost focus at least once? |
| `$error` | Object containing active validation errors, e.g. `$error.required` |

---

## 12. `$http` and Promises

**Definition:** `$http` is Angular's built-in service for making asynchronous HTTP requests (AJAX) to a backend/REST API; it returns a **promise**.

```js
app.controller('ApiController', function ($scope, $http) {
  $http.get('https://api.example.com/students')
    .then(function (response) {
      $scope.students = response.data;
    })
    .catch(function (error) {
      console.error('Request failed:', error.status);
    })
    .finally(function () {
      $scope.loading = false;
    });
});
```

### POST Example
```js
$http.post('/api/students', { name: 'New Student', marks: 80 })
  .then(function (res) { console.log('Created:', res.data); });
```

### `$q` — Promise Service (for custom async logic)
```js
app.service('DelayService', function ($q, $timeout) {
  this.wait = function (ms) {
    var deferred = $q.defer();
    $timeout(function () {
      deferred.resolve('Done waiting ' + ms + 'ms');
    }, ms);
    return deferred.promise;
  };
});
```

---

## 13. Component-Based Architecture (`.component()`)

**Definition:** Introduced in Angular 1.5+, `.component()` is a simplified API on top of directives, designed to be closer to Angular 2+'s component model — encouraging modular, reusable UI pieces with clear inputs/outputs.

```js
app.component('studentList', {
  templateUrl: 'student-list.html',
  bindings: {
    students: '<',       // one-way (recommended over '=')
    onSelect: '&'
  },
  controller: function () {
    var $ctrl = this;
    $ctrl.select = function (student) {
      $ctrl.onSelect({ selected: student });
    };
  }
});
```
```html
<student-list students="vm.students" on-select="vm.handleSelect(selected)"></student-list>
```

**Why components over directives for UI pieces:** simpler API, controller `this` instead of `$scope`, built-in lifecycle hooks (`$onInit`, `$onChanges`, `$onDestroy`), one-way (`<`) binding by default which discourages hidden mutation.

---

## 14. Interceptors

**Definition:** Interceptors are services that can globally hook into every `$http` request/response — commonly used for attaching auth tokens, logging, or global error handling.

```js
app.factory('AuthInterceptor', function ($q, $window) {
  return {
    request: function (config) {
      var token = $window.localStorage.getItem('token');
      if (token) config.headers.Authorization = 'Bearer ' + token;
      return config;
    },
    responseError: function (rejection) {
      if (rejection.status === 401) {
        console.warn('Unauthorized — redirect to login');
      }
      return $q.reject(rejection);
    }
  };
});

app.config(function ($httpProvider) {
  $httpProvider.interceptors.push('AuthInterceptor');
});
```

---

## 15. Animations (`ngAnimate`)

**Definition:** `ngAnimate` adds/removes CSS classes during Angular's DOM transitions (`ng-if`, `ng-repeat`, `ng-show`, etc.), which CSS transitions/keyframes can then hook into.

```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular-animate.min.js"></script>
```
```js
var app = angular.module('app', ['ngAnimate']);
```
```css
.fade.ng-enter { transition: 0.4s; opacity: 0; }
.fade.ng-enter-active { opacity: 1; }
.fade.ng-leave { transition: 0.4s; opacity: 1; }
.fade.ng-leave-active { opacity: 0; }
```
```html
<div class="fade" ng-if="showBox">Animated content</div>
```

---

## 16. Testing Basics

**Definition:** AngularJS was built with testability as a first-class concern; **Jasmine** (assertions) + **Karma** (test runner) is the standard combination, using Angular's `ngMock` module.

```js
describe('StudentController', function () {
  var $controller, $scope;

  beforeEach(angular.mock.module('schoolApp'));

  beforeEach(inject(function (_$controller_, $rootScope) {
    $controller = _$controller_;
    $scope = $rootScope.$new();
  }));

  it('should calculate average marks correctly', function () {
    var ctrl = $controller('StudentController', { $scope: $scope });
    $scope.marksList = [80, 90, 100];
    expect($scope.average()).toBe('90.00');
  });
});
```

---

## 17. Best Practices & Interview Q&A

### Best Practices
- Use `controller as` syntax instead of injecting `$scope` directly (avoids scope inheritance bugs).
- Keep controllers thin — put business logic and API calls in **services**.
- Prefer `.component()` over `.directive()` for reusable UI pieces.
- Use one-way binding (`<`) over two-way (`=`) where possible — easier to reason about data flow.
- Avoid `$rootScope` for shared state; use a shared service instead.
- Use `ng-if` for expensive/rarely-shown content, `ng-show` for frequently toggled small elements.
- Always name modules/controllers with a consistent convention (e.g., `feature.module.js`, `FeatureController`).
- Use `track by` in `ng-repeat` for performance with large lists.

### Quick Interview Q&A
**Q: What is the digest cycle?**
A: The internal loop AngularJS runs to check all `$watch`ed expressions for changes ("dirty checking") and update the DOM accordingly. It runs repeatedly until no more changes are detected (or a max of 10 iterations, to prevent infinite loops).

**Q: Difference between `ng-if` and `ng-show`/`ng-hide`?**
A: `ng-if` adds/removes the element (and its scope) from the DOM; `ng-show`/`ng-hide` just toggles CSS `display`, keeping the element (and its scope/watchers) alive.

**Q: What is a directive's `restrict` option?**
A: Defines how the directive can be used in HTML: `E` (element `<my-dir>`), `A` (attribute `my-dir`), `C` (class `class="my-dir"`), `M` (comment). Attribute (`A`) is most common for custom behavior; Element (`E`) for reusable components.

**Q: Why is two-way binding sometimes considered a performance concern?**
A: Because every bound expression is watched, and the digest loop re-evaluates all watchers on every cycle — with very large numbers of watchers, this dirty-checking can slow the app down. This is one motivation behind Angular 2+'s different (unidirectional) change-detection model.

**Q: What's the difference between AngularJS and Angular (2+)?**
A: AngularJS (1.x) is JavaScript-based, uses `$scope`/two-way binding/dirty-checking, and MVC. Angular (2+) is a full rewrite in TypeScript, component-based, uses unidirectional data flow and a different change-detection strategy, and is a separate framework despite the similar name.

---

## Suggested Practice Path
1. Build a static-data to-do list using `ng-repeat`, `ng-model`, `ng-click`.
2. Add filters (search + sort) and custom filters.
3. Convert it to use a `service`/`factory` for data instead of hardcoded arrays.
4. Add routing (`ngRoute` or `ui-router`) with at least 2 views.
5. Hook it up to a real REST API using `$http` (try your own EduConnect backend endpoints).
6. Refactor reusable UI parts into `.component()`s.
7. Add form validation and an auth interceptor.
8. Write a few Jasmine/Karma unit tests for a controller and a service.
