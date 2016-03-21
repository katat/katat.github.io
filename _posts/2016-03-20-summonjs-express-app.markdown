---
layout:     post
title:      "SummonJS - a minimal framework to manage dependencies on Node.JS"
subtitle:   "Create An Easy to Test Node.JS Express APP by injecting dependencies via SummonJS"
date:       2016-03-20 12:00:00
author:     "Katat Choi"
header-img: "img/post-bg-01.jpg"
---

# SummonJS - a minimal framework to manage dependencies on Node.JS

Coming from the Java world, I appreciate how the Java spring ease the dependency management, using its XML configuration to define the dependencies for the application, making the whole application configurable. This way of managing the dependencies help keep the code clean, and easy to swap the dependencies when needed in certain situations, such as unit tests.

> Dependency injection is a software design pattern in which one or more dependencies (or services) are injected, or passed by reference, into a dependent object.

When writing unit tests, it is very common to get to a point that there are dependencies not easy to mock up. Even there are always ways to decouple the code to into smaller chucks to make the unit testable, but many times I doubt if it is really worthwhile to do so.

Then I started looking around for the similar solutions to Java Spring in Node.JS world. I found [dependable](https://github.com/idottv/dependable), which state itself as:

> A minimalist dependency injection framework for node.js.

As it described it is a minimalist dependency injection framework, its feature is far simpler from what Java Spring has. But the dependency injection is enough for many use cases. Basically, `dependable` allow dependencies to be defined with `names` in the code instead of via XML or JSON, and automatically inject the dependency objects as dependent module's function arguments, names of which are the same as the defined dependencies.

I was using `dependable` for a while and putting all the dependency definitions in a file JS file. Overtime, the JS dependency definition file got bigger and bigger with many boilerplate codes. I thought it would be better to put all the JS dependency code simply into a JSON file. Then I created an open source project call `SummonJS` to wrap up the `dependable` framework into a way that makes the code cleaner and easier to use, also added a `hook` feature allow adding pre/post hook callback functions to a dependency's object function.

In this post, I will introduce how the [SummonJS](https://github.com/katat/summonjs) can help manage the Node.JS dependencies by creating a simple Node.JS express API, which is for creating custom forms.

### Dependency Definition
Suppose the backend app has the following SummonJS dependency definition:

```javascript
{
  "dependency": {
    "Configs": "./config/dev.json",
    "FormDB": "./dbs/form",
    "FormModel": "./models/form",
    "FormController": "./controllers/form",
    "FormRoute": "./routes/form",
    "mongoose": {
      "path": "mongoose",
      "shim": true
    }
  }
}
```

This JSON file defines all the dependencies needed for the app. The property names are the dependency names for the source code path as the corresponding values. The another format of the definition is like how the `mongoose` dependency is defined. The `path` is the value of where the dependency will be required from, and the interesting part is the `shim` property that tells `SummonJS` to require `mongoose` without any other dependencies to be passed as arguments when initializing the `mongoose` instance.

This is because when `var mongoose = require("mongoose")`, the variable `mongoose` is actually a function with arguments. If we don't define it with `shim`, SummonJS will trying to figure the corresponding dependencies based on the arguments' names and it could create issues. With `shim`, it tells SummonJS not to inject the dependencies for `mongoose` function argument names. You should be able to get a better understanding about this later in dependency injection section.

### Dependency Injection
In the `FormDB` dependency, its source code in the `./dbs/form` file is as below:

```javascript
module.exports = function(mongoose, Configs) {
    mongoose.connect(Configs.dbHost, function(err) {
        if(err) {
            console.error(err, 'Make sure you have set the correct database connection string for dbHost in the config file.');
        }
    });
    return mongoose;
};
```

Yes, there is no `require` needed. In this dependent object, it depends on the `mongoose` and `Configs`. Both of these dependencies will be injected automatically by SummonJS as they are matched to names in the JSON definition file.

Also take a look at the `FormModel` dependent object, which depends on the `FormDB` and the `mongoose`:

```javascript
module.exports = function(mongoose, FormDB) {
    var schema = {
        name: String,
        fields: mongoose.Schema.Types.Mixed
    };
    var FormModel = FormDB.model('FormModel', schema);
    return FormModel;
};
```

Here is how the express route dependent looks like:

```javascript
module.exports = function(app, FormController) {
    app.route('/form/:name?')
    	.post(FormController.create)
        .get(FormController.get);
};
```

> Note the `app` is not defined in the configuration file. That is because this dependency is only available after initialized the express server instance. So it needs to be injected during the bootstrap process. We will see how it works in the next section.

### Bootstrap App
For this sample express APP, it will need to start the app with http server listening port. Durning this bootstrap process, it will require the integration with the SummonJS in order to make the dependency injection works. Below is the code bootstrap the app.

```javascript
//init summonjs
var depConfigs = require('./depend.json');
var summon = require('summonjs')({
    configs: depConfigs
});
app.summon = summon;

app.setup = function(callback) {
    var targets = Object.keys(depConfigs.dependency).map(function(name){
        if(name.toLowerCase().indexOf('route') !== -1) {
            return name;
        }
    }).filter(function(name){
        return name? true: false;
    });
    summon.invoke({
        override: {app: function(){return app;}},
        targets: targets
    });
    var server = app.listen(app.summon.get('Configs').apiPort || 5000, function() {
        var port = server.address().port;
        callback && callback();
        console.log('Server up and listening at %s', port);
    });
};
```
Here is how it works in order:

 1. Initializes the `SummonJS` with the dependency configuration JSON. At other place during the startup will call the `app.setup` function.

 2. Looks up the express `route` dependency names in the configuration JSON by name convention `route`, and save them in the `targets` variable.

 3. Now the express instance `app` is ready for use, call `summon.invoke` to pass in `app` to the override property, and named it as `app`. Also passed in dependency name `targets`as target parameter, all of which will be called and initialized at this point, so all the `route` dependencies will be setup and initialize the whole dependency tree, from route, to FormController, to FormModel, to FormDB.

### Unit Tests
Since we are using `SummonJS` to manage the dependencies. Let's take a look first at how easy we could build the mock.

```javascript
beforeEach(function () {
    app.summon.get('mockgoose').reset();
});

var formDefine = {
    name: 'newForm',
    fields: [
        {name: 'First Name', type: 'text'},
        {name: 'Last Name', type: 'text'},
        {name: 'Email', type: 'email'},
        {name: 'Introduction', type: 'textarea'}
    ]
};
it('should create new form type', function (done) {
    request(app)
        .post('/form')
        .send(formDefine)
        .expect(201)
        .end(function(err, res) {
            assert.ifError(err);
            var created = res.body;
            assert(created._id);
            assert.deepEqual(formDefine.fields, created.fields);
            done();
        });
});
```

In the `beforeEach`, it uses the `mockgoose` to reset the mongodb database for each unit test. But there is another simple script to swap the original mongoose dependency with mockgoose.

```javascript
global.app = require('../app');
var configs = require('../app/config/test.json');
global.app.summon.register('Configs', configs);

var mockgoose = require('mockgoose');
mockgoose(app.summon.get('mongoose'));
app.summon.register('mockgoose', function(){
    return mockgoose;
});
app.setup();
```

What above code does is replace the test dependency configs, and register it in run time to override the one in JSON config file. Then it use the mockgoose to intercept the origin mongoose by `mockgoose(app.summon.get('mongoose'))`, avoiding testing against the real database.

> Note that we don't need to register mongoose to override the origin mongoose after modified by mockgoose, since all the dependencies returned by summon are singleton instances.

So you can see, SummonJS is capable to swap the dependencies for the any parts of the dependency tree, easing the mocks for unit tests.

### Summary
Hopefully this post demonstrates what SummonJS can help in easing the dependencies management, leading to a cleaner code base with minimal boilerplate codes, and give clues on what you can do with it in real world Node.JS applications. For the example app, you take a look at the source code [flexform-api](https://github.com/katat/flexform-api).

There is another interesting feature in SummonJS, called `hook`, enable adding pre/post hooks to a dependency object's function. This is useful for scenarios such as caching results return in function's async callback or pre-processing the inputs before passing them to the main function. I will talk about this feature in another post.
