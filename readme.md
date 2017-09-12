profane - performance analyzer
----------------

*for those of us who find flame charts confusing...*

#### premise:
tagging system which is an alternative to a flame graph, to be used in tests as it will slow down production systems. Profane wraps nodes require and checks for start/end tags in methods, it will generate a copy of the actual module file and require that with code that logs the average method execution time, and require the modified module

NB - this is a quickie, so dont expect predictable results if you dont tag properly...

## quickstart:

#### 1. install and save dev

```bash

npm i happner-profane --save-dev

```

#### 2. tag your code

profane uses special tags to denote start and end for average execution time logging, NB - make sure your tags "key" matches the method name and start and end keys match, in async methods, also be sure to specify the "self" argument in the tag if you are using an external context

```javascript

function testAnalyzerModule1(){

  this.__events = new EventEmitter();
}

//example sync function:

testAnalyzerModule1.prototype.testMethod2 = function(){

  //[start:{"key":"testMethod2"}:start]

  var i = 0;

  while(i < 100000){
    i++;
  }

  //[end:{"key":"testMethod2"}:end]
};

//example async function:

testAnalyzerModule1.prototype.testPromiseMethod = function(str, int){

  var _this = this;

  //[start:{"key":"testPromiseMethod"}:start]

  return new Promise(function(resolve){

    setTimeout(function(){

      //notice the "self" argument is named _this (from outside of promise above)
      //[end:{"key":"testPromiseMethod", "self":"_this"}:end]
      resolve();
    }, 2000)

  });
};

module.exports = testAnalyzerModule1;

```

#### 3. run your tests, and check the analysis output:

```javascript

//instantiate
var analyzer = require('happner-profane').create();

//require a module (can be any module you have in your project)
var TestModule1 = analyzer.require('./fixtures/testAnalyzerModule1', true);

var testModule1 = new TestModule1();

//require another module
var TestModule2 = analyzer.require('./fixtures/testAnalyzerModule2', true);

var testModule2 = new TestModule2();

//execute some sync methods
testModule1.testMethod1('test', 1);

testModule1.testMethod2('test', 2);

testModule2.testMethod1('test', 1);

testModule2.testMethod2('test', 2);

testModule1.testMethod3(function(){

//execute some promises

  testModule1.testPromiseMethod('promise', 1)

    .then(function(){

      testModule2.testPromiseMethod('promise', 2)

        .then(function(){

          //output analysis
          var analysis = analyzer.getAnalysis();

          //should output a results object with average execution times for your class and methods
          /*
            {
              "testAnalyzerModule1": {
                "testMethod1": 0,
                "testMethod2": 1,
                "testMethod3": 2001,
                "testPromiseMethod": 2007,
                "some_event": 1
              },
              "testAnalyzerModule2": {
                "testMethod1": 0,
                "testMethod2": 0,
                "testPromiseMethod": 2000
              }
            }
          */

          //cleanup the the generated modified modules
          analyzer.cleanup();
        })
    });
});
```

#### supported node versions:

v0.10 - v8