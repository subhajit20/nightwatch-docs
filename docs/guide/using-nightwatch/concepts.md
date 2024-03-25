<div class="page-header"><h1>Core Concepts</h1></div>

### Defining Test Environments

It is likely you will run your tests against multiple environments and/or different browsers. Nightwatch provides a concept for defining different _environments_, in which you can set specific test settings.

The environments are located under the `"test_settings"` dictionary in the configuration file. A `default` environment is always required from which the other environments inherit the settings. You can overwrite any test setting for each environment as needed.

The configuration file which is auto-generated by Nightwatch (`nightwatch.conf.js`) contains several pre-defined test environments for running tests against several different browsers (Firefox, Chrome, Safari), and also for running tests using Selenium Server or popular cloud testing provider Browserstack.com.

Here’s an extract:

<div class="sample-test">

<pre><code class="language-javascript">module.exports = {
  src_folders: [],

  test_settings: {
    default: {
      launch_url: 'https://nightwatchjs.org'
    },

    safari: {
      desiredCapabilities : {
        browserName : 'safari',
        alwaysMatch: {
          acceptInsecureCerts: false
        }
      },
      webdriver: {
        port: 4445,
        start_process: true,
        server_path: '/usr/bin/safaridriver'
      }
    },

    firefox: {
      desiredCapabilities : {
        browserName : 'firefox'
      },

      webdriver: {
        start_process: true,
        port: 4444,
        server_path: require('geckodriver').path
      }
    }
  }
}</code></pre>

</div>

Considering this setup, to run tests, for instance, against Safari, we would run the following the command-line:

<pre><code class="language-bash">nightwatch --env safari</code></pre>

Refer to the [Running Tests][1] documentation section to learn more about how to use the Nightwatch test runner.

### Extending Test Environments

In some cases, you will need to extend another environment which is not the `default` one. A common scenario for this use case is the situation when you are using the Selenium Server or a cloud-based testing service (such as [Sauce Labs][2] or [Browserstack][3]).

In this case you will most likely need a base environment for common settings (such as `host` or `port`) and several sub-environments for each target browser (e.g. Firefox, Chrome).

Going back to the auto-generated configuration file, `nightwatch.conf.js`, we have provided an example structure for using the Selenium Server with Chrome and Firefox.

Here's the relevant extract. Notice the "`extends`" property:

<div class="sample-test">

<pre><code class="language-javascript">module.exports = {
  src_folders: [],

  test_settings: {
    default: {
      launch_url: 'https://nightwatchjs.org'
    },

    selenium: {
      // Selenium Server is running locally and is managed by Nightwatch
      selenium: {
        start_process: true,
        port: 4444,
        server_path: require('selenium-server').path,
        cli_args: {
          'webdriver.gecko.driver': require('geckodriver').path,
          'webdriver.chrome.driver': require('chromedriver').path
        }
      },
      webdriver: {
        start_process: false
      }
    },

    'selenium.chrome': {
      extends: 'selenium',
      desiredCapabilities: {
        browserName: 'chrome',
        chromeOptions : {
        }
      }
    },

    'selenium.firefox': {
      extends: 'selenium',
      desiredCapabilities: {
        browserName: 'firefox'
      }
    }
  }
}</code></pre>

</div>

With this setup, you can run the tests using Selenium Server and Firefox with this command:

<pre><code class="language-bash">nightwatch --env selenium.firefox</code></pre>

## Using Test Globals

Another useful concept that Nightwatch provides is test globals. In its most simple form, it is a dictionary of name-value pairs which is defined in your configuration file.

Globals can be defined either as a `"globals"` property or as an external file which is specified as the `"globals_path"` property.

Here's an example definition using the `"globals"` property in `nightwatch.json`:

<div class="sample-test">

<pre data-language="javascript"><code class="language-javascript">{
  "src_folders": [],

  "test_settings": {
    "default": {
      "launch_url": "https://nightwatchjs.org",

      "globals": {
        "myGlobalVar" : "some value",
        "otherGlobal" : "some other value"
      }
    }
  }
}</code></pre>

</div>

Like the `launch_url` property, the `globals` object is made available directly on the Nightwatch api which is passed to the tests.

<div class="sample-test">

<pre data-language="javascript"><code class="language-javascript">module.exports = {
  'Demo test' : function (browser) {
    console.log(browser.globals.myGlobalVar); // myGlobalVar == "some value"
  }
};</code></pre>

</div>

### Pre-defined Globals

The following global properties can be used to control the behaviour of the test runner and are defined with the following default values:

<div class="sample-test">

<pre data-language="javascript"><code class="language-javascript">module.exports = {
  globals: {
    // this controls whether to abort the test execution when an assertion failed and skip the rest
    // it's being used in waitFor commands and expect assertions
    abortOnAssertionFailure: true,

    // this will overwrite the default polling interval (currently 500ms) for waitFor commands
    // and expect assertions that use retry
    waitForConditionPollInterval: 500,

    // default timeout value in milliseconds for waitFor commands and implicit waitFor value for
    // expect assertions
    waitForConditionTimeout : 5000,

    // since 1.4.0 – this controls whether to abort the test execution when an element cannot be located; an error
    // is logged in all cases, but this also enables skipping the rest of the testcase;
    // it's being used in element commands such as .click() or .getText()
    abortOnElementLocateError: false,

    // this will cause waitFor commands on elements to throw an error if multiple
    // elements are found using the given locate strategy and selector
    throwOnMultipleElementsReturned: false,

    // By default a warning is printed if multiple elements are found using the given locate strategy
    // and selector; set this to true to suppress those warnings
    suppressWarningsOnMultipleElementsReturned: false,

    // controls the timeout value for async hooks. Expects the done() callback to be invoked within this time
    // or an error is thrown
    asyncHookTimeout : 10000,

    // controls the timeout value for when running async unit tests. Expects the done() callback to be invoked within this time
    // or an error is thrown
    unitTestsTimeout : 2000,

    // controls the timeout value for when executing the global async reporter. Expects the done() callback to be
    // invoked within this time or an error is thrown
    customReporterCallbackTimeout: 20000,

    // Automatically retrying failed assertions - You can tell Nightwatch to automatically retry failed assertions
    // until a given timeout is reached, before the test runner gives up and fails the test.
    retryAssertionTimeout: 5000,

    // Custom reporter
    reporter: function(results, done) {
      // do something with the results
      done(results);
    }
  }
}
</code></pre>

</div>

### Environment Specific Globals

Like other test settings, globals have the ability to be overwritten per test environment. Consider this configuration:

<div class="sample-test">

<pre data-language="javascript"><code class="language-javascript">{
  "src_folders": [],

  "test_settings": {
    "default": {
      "launch_url": "https://nightwatchjs.org",

      "globals": {
        "myGlobalVar" : "some value",
        "otherGlobal" : "some other value"
      }
    },

    "integration": {
      "globals": {
        "myGlobalVar" : "integrated global"
      }
    }
  }
}</code></pre>

</div>

If we still pass the `--env integration` option to the runner, then our globals object will look like below:

<pre><code class="language-bash">nightwatch --env integration</code></pre>

<div class="sample-test">

<pre data-language="javascript"><code class="language-javascript">module.exports = {
  'Demo test' : function (browser) {
    console.log(browser.globals.myGlobalVar); // myGlobalVar == "integrated global"
  }
};</code></pre>

</div>

### External Test Globals

Test globals can also be defined in an external file, specified by the `"globals_path"` settings in your configuration file.

The external globals file can also contain global test hooks, a custom reporter and other test specific settings. Refer to the [External Globals](/guide/using-nightwatch/external-globals.html) section for more details.

[1]:    /guide/running-tests/
[2]:    https://saucelabs.com
[3]:    https://browserstack.com