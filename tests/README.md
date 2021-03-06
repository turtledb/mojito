Mojito Arrow Tests
==================

Mojito has traditionally had a built-in `mojito test` command which is being
migrated to leverage Arrow, a Yahoo!-developed multi-type testing harness.

The files in this directory tree represent Arrow tests constructed for Mojito.
The unit and func directories contain 'mirrors' of the source directory from
Mojito so that unit tests for a component found in `source/lib/app/autoload` are
found in `unit/lib/app/autoload`. Functional tests are under the `func` directory.

Test files begin with a `test-` prefix but otherwise match the name of the file
they are testing. For example, unit tests for the `mojit-proxy.client.js` file are
found `unit/lib/app/autoload/test-mojit-proxy.client.js`.

The `base` subdirectory here contains files which provide baseline functionality
which is leveraged in specific unit and functional tests. Files in this
directory are described in detail later in this document.


Test Descriptors
================

Arrow uses test descriptor files in JSON format to describe unit tests and their
parameters. For Mojito we leverage a top-level descriptor file in each of the
unit and func subdirectories which contains references to all the tests within
that subdirectory. A portion of a test_descriptor file is shown below.

    [
        {
            "settings": [ "master" ],
            "name" : "mojito-unit-tests",
            "config" : {
            },
            "dataprovider" : {
                "mojito-client.client" : {
                    "params" : {
                        "page": "../base/mojito-test.html",
                        "lib": "../../source/lib/app/autoload/mojito-client.client.js",
                        "test" : "./lib/app/autoload/test-mojito-client.client.js"
                    }, 
                    "group" : "fw,unit"
                },
                "mojit-proxy.client" : {
                    "params" : {
                        "page": "../base/mojito-test.html",
                        "lib": "../../source/lib/app/autoload/mojit-proxy.client.js",
                        "test" : "./lib/app/autoload/test-mojit-proxy.client.js"
                    }, 
                    "group" : "fw,unit"
                }
            }
        },
        {
            "settings": [ "environment:development" ]
        }
    ]

From a test configuration perspective the most interesting sections are the
"config" and "dataprovider" objects which provide common parameters and test
definitions respectively.

Each test description within the dataprovider object is named by convention to
match the name of the file-under-test. In the event of a file with an identical
name in different portions of the tree a prefix of the directory path sufficient
to make the name unique should be used. For example, the `lib/app/autoload` file
mojito-client.client.js is tested with a test name of `mojito-client.client`
while the `app/mojits/HTMLFrameMojit/controller.server.js` file is tested using a
test name of `HTMLFrameMojit/controller.server`.

Individual tests are described using two common properties: `params` and `group`.

The `group` property is used to categorize the test into one or more types. For
our purposes the common types are 'unit' or 'func' combined with 'fw', 'app', or
'mojit'. Typically group settings should combine two group names separated with
a comma and _no space_ as shown in the example above.

The params property is an object containing specific parameters. The three of
interest to us are `"page"`, `"lib"`, and `"test"`.

The `"page"` setting for each unit test refers to the `mojito-test.html` file. The
`mojito-test.html` file is a pure HTML file which loads common libraries (`yui`,
`yui-test`, and `mojito-test`) which ensure that each file-under-test has an
isolated context in which to execute. [NOTE we're investigating how to properly
default to a common page to 'dry' this up].

The "lib" property examples point simply to the individual file-under-test,
usually found in the `source/lib` tree. It is a convention that there SHOULD BE NO
OTHER LIB FILES necessary to unit test a particular file. The need for any other
lib (or commonlib) file indicates an unmocked dependency or a bug in the file
(such as failure to declare all 'requires').

The `"test"` property should point to the test file itself, typically found in a
set of parallel subdirectories to the file-under-test.


Test Files
==========

A unit test file will follow a consistent pattern of the form:

    /*
     * Copyright (c) 2011-2013, Yahoo! Inc.  All rights reserved.
     * Copyrights licensed under the New BSD License.
     * See the accompanying LICENSE file for terms.
     */
    
    
    /*jslint anon:true, sloppy:true, nomen:true, node:true*/
    /*global YUI*/
    
    
    /*
     * Test suite for the mojito-client.client.js file functionality.
     */
    YUI({useBrowserConsole: true}).use(
        'mojito-client',
        'test',
        function(Y, NAME) {
    
            var suite = new Y.Test.Suite("mojito-client.client tests"),
                A = Y.Assert;
    
            suite.add(new Y.Test.Case({
    
                setUp: function() {
                    this.mojitoClient = new Y.mojito.Client();
                },
    
                "test constructor": function() {
                    var client = this.mojitoClient;
                    A.isObject(client);
                }
            }));
    
            Y.Test.Runner.add(suite);
        }
    );

The copyright and jslint directives should be retained and not altered. Yes,
test files should not contain lint, they're still code :).

The `YUI.use()` function is leveraged to `require` the file-under-test's module
content and define the `test` operation via the function in the third parameter.

A common test should create a `Y.Test.Suite` instance, optionally define shared
variables (the example defines `A` as an alias for `Y.Assert`), populate the
suite with one or more tests, and finally add the suite to the `Y.Test.Runner`.


Test Execution
==============

See [Mojito Framework's Unit and Functional Tests](https://github.com/yahoo/mojito/wiki/Mojito-Framework's-Unit-and-Functional-Tests) for instructions on how to run the functional tests found in the `tests` directory.

