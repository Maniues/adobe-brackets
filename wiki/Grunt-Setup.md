**This is a _subset_** of the steps needed to [generate a Brackets release build](https://github.com/adobe/brackets/wiki/Building-Brackets-Releases).


1. Install Node 0.8.x or newer http://nodejs.org/download/
2. Run ``npm install -g grunt-cli`` to install the GruntJS command line interface
3. Run ``npm install -g jasmine-node`` to install the jasmine-node test runner
4. Run ``npm install`` from the root of the brackets git repo
5. Run ``grunt``

# Development Task Details

* ``grunt jshint`` Run JSHINT on all ``/src`` and ``/test`` files as well as the ``Gruntfile.js``
* ``grunt jasmine`` Run headless Jasmine tests
* ``grunt jasmine-node`` to run the Node tests
* ``grunt test`` Run JSHINT and Jasmine if JSHINT completes without errors
* ``grunt watch`` Watch for file changes, then run JSHINT and Jasmine

# Adding Unit Tests

Non-integration tests (typically tests that don't require a full Brackets instance running) are candidates to run headless via [PhantomJS](http://phantomjs.org). The headless tests use a separate spec runner that is configured in [Gruntfile.js](https://github.com/adobe/brackets/blob/master/Gruntfile.js). To add new tests, modify the ``config`` object, find the ``specs`` property and add the path to the spec file (e.g. ``test/spec/MyFeature.js`` to the array of specs.

Node-based tests are also good candidates to run via Grunt. Look at the `jasmine-node` task in Gruntfile.js.

# Misc. Tasks

* ``grunt write-config`` Automatically run after ``npm install`` to update ``src/config.json``
* ``grunt install`` See ``write-config``