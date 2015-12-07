# Helpers

Helpers is a core concept of CodeceptJS. Helper is a wrapper around various libraries providing unified interface around them.
Methods of Helper class will be available in tests in `I` object. This abstracts test scenarios from the implementation and allows easy switching between backends. 
Functionality of CodeceptJS should be extended by writing a custom helpers.

You can either access core Helpers (and underlying libraries) or create a new from scratch. 

## Development

Helpers can be created by running a generator command:

```bash
codeceptjs gh
```

*(or `generate helper`)*

Helpers are ES6 classes inherited from [corresponding abstract class](https://github.com/Codeception/CodeceptJS/blob/master/lib/helper.js).
Generated Helper will be added to `codecept.json` config. It should look like this:

```js
'use strict';

let Helper = require('codeceptjs/helper');

class MyHelper extends Helper {

  // before/after hooks
  _before() {
    // remove if not used  
  }
  
  _after() {
    // remove if not used
  }
  
  // add custom methods here  
  // If you need to access other helpers
  // use: this.helpers['helperName']    
  
}

module.exports = MyHelper;
```

All methods except those starting from `_` will be added to `I` object and trated as test actions.
Every method should return a value in order to be appended into promise chain.

## WebDriverIO Example  

Next example demontrates how to use WebDriverIO library to create your own test action.
Method `seeAuthenication` will use `client` instance of WebDriverIO to get access to cookies.
Standard NodeJS assertion library will be used.

```js
'use strict';
let Helper = require('codeceptjs/helper');

// use any assertion library you like
let assert = require('assert');

class MyHelper extends Helper {  
  /**
   * checks that authentication cookie is set 
   */
  seeAuthentication() {    
    // access current client of WebDriverIO helper
    let client = this.helpers['WebDriverIO'].browser;
    
    // get all cookies according to http://webdriver.io/api/protocol/cookie.html
    // any helper method should return a value in order to be added to promise chain
    return client.cookie(function(err, res) {
      // get values
      let cookies = res.value;
      for (let k in cookies) {
        // check for a cookie
        if (cookies[k].name != 'logged_in') continue;
        assert.equal(cookies[k].value, 'yes');
        return;        
      }
      assert.fail(cookies, 'logged_in', "Auth cookie not set");      
    });
  }
}
```

## Initialization

Helpers can be configured in `codecept.json` and config values are passed into constructor.
By default config values will be stored in `this.config`. You can redefine constructor to provide custom initialization and customization.

```js
constructor(config) {
  config.defaultValue = '42';
  super(config);
}
```

## Hooks

Helpers may contain several hooks you can use to handle events of a test.
Implement corresponding methods to them.  

* `_init` - before all tests
* `_before` - before a test
* `_beforeStep` - before each step
* `_afterStep` - after each step

Each implemented method should return a value as they will be added to global promise chain as well.

### done()