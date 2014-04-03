# ZenInjector

A simple library for dependency injection with promises.

# Usage

```javascript
var container = new require('zeninjector');

// register a module
container.register('config', function() {
  return {
    db: {url: 'mongodb://localhost:27017/myapp'}
  }
});

// or, if the module has no dependency, you can also
// do that
container.registerAndExport('config', {
  db: { url: 'mongodb://localhost:27017/myapp' }
});

// get the module
container.resolve('config').then(function(config) {
  var dbUrl = config.db.url;
  // do something with it
});
```

## Registering a module
A module is an object with a unique `name`, an optional array of `dependencies`, and a function `define` which returns the module. The dependencies are lazily loaded, so `define` will only be called when it is needed. It is similar to [angularJS DI system](http://docs.angularjs.org/guide/di) or [require.js](http://requirejs.org/) syntax.

**More complex example**

```javascript
var MongoClient = require('mongodb').MongoClient;

container.register('db', function(config) {
  var connect = Promise.promisify(MongoClient.connect, MongoClient);
  return connect(config.db.url);
});

container.resolve('db').then(function(db) { // connect to the db here
  // ... use the db object here
}, function onError(err) {
  console.error('cannot connect to db', err);
});
```

If you prefer, dependencies can also be explicitely written out:

```javascript
container.register('db', ['config', function(config) {
  var connect = Promise.promisify(MongoClient.connect, MongoClient);
  return connect(config.db.url);
}]);
```

Sometimes, you want to add an already existing object to the container, for example
an NPM module or something from another library. There is a shortcut for this:

```javascript
var fs = container.registerAndExport('fs', require('fs'));
```

## With generators
`resolve` returns a promise so it can easily be used in coroutines. Below is the 'complex' example above rewritten using coroutines.

```javascript
var Promise = require('bluebird');
Promise.spawn(function* () {
  var MongoClient = require('mongodb').MongoClient;
  container.register('db', function(config) {
    var connect = Promise.promisify(MongoClient.connect, MongoClient);
    return connect(config.db.url);
  });

  try {
    var db = yield container.register('db');
    // use the db here
  } catch(err) {
    console.error('cannot connect to the db', err);
  }
});
```

# Run tests
`npm test`, or `mocha --ui tdd --reporter spec` if you have mocha installed as a global module.

# License
MIT
