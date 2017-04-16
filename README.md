# api documentation for  [edge (v6.5.1)](https://github.com/tjanczuk/edge)  [![npm package](https://img.shields.io/npm/v/npmdoc-edge.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-edge) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-edge.svg)](https://travis-ci.org/npmdoc/node-npmdoc-edge)
#### Edge.js: run .NET and Node.js in-process on Windows, Mac OS, and Linux

[![NPM](https://nodei.co/npm/edge.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/edge)

[![apidoc](https://npmdoc.github.io/node-npmdoc-edge/build/screenCapture.buildCi.browser.%252Ftmp%252Fbuild%252Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-edge/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-edge/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-edge/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Tomasz Janczuk",
        "url": "http://tomasz.janczuk.org"
    },
    "bugs": {
        "url": "http://github.com/tjanczuk/edge/issues"
    },
    "dependencies": {
        "edge-cs": "1.2.1",
        "nan": "^2.0.9"
    },
    "description": "Edge.js: run .NET and Node.js in-process on Windows, Mac OS, and Linux",
    "devDependencies": {
        "jshint": "1.1.0",
        "mocha": "2.5.3"
    },
    "directories": {},
    "dist": {
        "shasum": "d3ebb02d0a7c044cb80bdfd4275ba64df9da57bb",
        "tarball": "https://registry.npmjs.org/edge/-/edge-6.5.1.tgz"
    },
    "engines": {
        "node": ">= 0.8"
    },
    "gitHead": "636f6cf4f91eb7b1111e84d8927e116464a0a1c7",
    "homepage": "https://github.com/tjanczuk/edge",
    "licenses": [
        {
            "type": "Apache",
            "url": "http://www.apache.org/licenses/LICENSE-2.0"
        }
    ],
    "main": "./lib/edge.js",
    "maintainers": [
        {
            "name": "tjanczuk"
        }
    ],
    "name": "edge",
    "optionalDependencies": {},
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/tjanczuk/edge.git"
    },
    "scripts": {
        "install": "node tools/install.js",
        "jshint": "node ./tools/runJsHint.js",
        "test": "node tools/test.js"
    },
    "tags": [
        "owin",
        "edge",
        "net",
        "clr",
        "coreclr",
        "c#",
        "mono",
        "managed",
        ".net"
    ],
    "version": "6.5.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module edge](#apidoc.module.edge)
1.  [function <span class="apidocSignatureSpan">edge.</span>func (language, options)](#apidoc.element.edge.func)



# <a name="apidoc.module.edge"></a>[module edge](#apidoc.module.edge)

#### <a name="apidoc.element.edge.func"></a>[function <span class="apidocSignatureSpan">edge.</span>func (language, options)](#apidoc.element.edge.func)
- description and source-code
```javascript
func = function (language, options) {
    if (!options) {
        options = language;
        language = 'cs';
    }

    if (typeof options === 'string') {
        if (options.match(/\.dll$/i)) {
            options = { assemblyFile: options };
        }
        else {
            options = { source: options };
        }
    }
    else if (typeof options === 'function') {
        var originalPrepareStackTrace = Error.prepareStackTrace;
        var stack;
        try {
            Error.prepareStackTrace = function(error, stack) {
                return stack;
            };
            stack = new Error().stack;
        }
        finally
        {
            Error.prepareStackTrace = originalPrepareStackTrace;
        }

        options = { source: options, jsFileName: stack[1].getFileName(), jsLineNumber: stack[1].getLineNumber() };
    }
    else if (typeof options !== 'object') {
        throw new Error('Specify the source code as string or provide an options object.');
    }

    if (typeof language !== 'string') {
        throw new Error('The first argument must be a string identifying the language compiler to use.');
    }
    else if (!options.assemblyFile) {
        var compilerName = 'edge-' + language.toLowerCase();
        var compiler;
        try {
            compiler = require(compilerName);
        }
        catch (e) {
            throw new Error("Unsupported language '" + language + "'. To compile script in language '" + language +
                "' an npm module '" + compilerName + "' must be installed.");
        }

        try {
            options.compiler = compiler.getCompiler();
        }
        catch (e) {
            throw new Error("The '" + compilerName + "' module required to compile the '" + language + "' language " +
                "does not contain getCompiler() function.");
        }

        if (typeof options.compiler !== 'string') {
            throw new Error("The '" + compilerName + "' module required to compile the '" + language + "' language " +
                "did not specify correct compiler package name or assembly.");
        }

        if (process.env.EDGE_USE_CORECLR) {
            options.bootstrapDependencyManifest = compiler.getBootstrapDependencyManifest();
        }
    }

    if (!options.assemblyFile && !options.source) {
        throw new Error('Provide DLL or source file name or .NET script literal as a string parmeter, or specify an options object
 '+
            'with assemblyFile or source string property.');
    }
    else if (options.assemblyFile && options.source) {
        throw new Error('Provide either an asseblyFile or source property, but not both.');
    }

    if (typeof options.source === 'function') {
        var match = options.source.toString().match(/[^]*\/\*([^]*)\*\/\s*\}$/);
        if (match) {
            options.source = match[1];
        }
        else {
            throw new Error('If .NET source is provided as JavaScript function, function body must be a /* ... */ comment.');
        }
    }

    if (options.references !== undefined) {
        if (!Array.isArray(options.references)) {
            throw new Error('The references property must be an array of strings.');
        }

        options.references.forEach(function (ref) {
            if (typeof ref !== 'string') {
                throw new Error('The references property must be an array of strings.');
            }
        });
    }

    if (options.assemblyFile) {
        if (!options.typeName) {
            var matched = options.assemblyFile.match(/([^\\\/]+)\.dll$/i);
            if (!matched) {
                throw new Error('Unable to determine the namespace name based on assembly file name. ' +
                    'Specify typeName parameter as a namespace qualified CLR type name of the application class.');
            }

            options.typeName = matched[1] + '.Startup';
        }
    }
    else if (!options.typeName) {
        options.typeName = "Startup";
    }

    if (!options.methodName) {
        options.methodName = 'Invoke';
    }

    return edge.initi ...
```
- example usage
```shell
...

You can script C# from a Node.js process:

**ES5**
'''javascript
var edge = require('edge');

var helloWorld = edge.func(function () {/*
async (input) => {
    return ".NET Welcomes " + input.ToString();
}
*/});

helloWorld('JavaScript', function (error, result) {
if (error) throw error;
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
