
## Introduction
Consume a WSDL file and output an ECMAScript library. 

## Git it:
```
sudo npm install -g ivory 
```

## How do I use it?
This will generate a folder called [serviceName] in the current directory ready to be require'd and used:
```
ivory [serviceName] [/local/path/to/wsdl]
```

## Requirements for using the generated code
```
npm install request xml2json
```

## Generated code structure
```
./[ServiceName]/
|   // This holds one file per defined Element within the WSDL
├── Element
│   ├── SomeDefinedElement.js
|   └── ...
|   // This is the main file which handles requests, JSON->XML->JSON, etc
├── index.js
|   // This is where mock data goes from [myService].Settings.createMock
├── Mocks
│   ├── WsdlOperationName.js
|   └── ...
|   // This library provides strong typing, it's used in each Element/Type
├── Modeler.js
|   // This file defines the top level functionality found within the WSDL
├── ServiceDefinition.js
|   // This holds one file per defined Type within the WSDL
└── Type
    ├── SomeDefinedType.js
    └── ...
```

## Using the generated code for the weather service
```javascript
var service = require("./[serviceName]");
```
This is how we create a new request:
```javascript
var someRequest = new service.[WSDL-Binding-Name].[WSDL-Operation-Name]();
```
Setting basic properties is trivial
```javascript
someRequest.someSimpleProperty = 1;
```
Most requests consist of several complex types, they are all found within our service object:
```javascript
someRequest.someElementProperty = new Service.Elements.[WSDL-Element-Name]();
someRequest.someTypeProperty = new Service.Types.[WSDL-Type-Name]();
```
Populating Requests/Elements/Types can be done one at a time:
```javascript
someRequest.someNumber = 1;
someRequest.someString = "1";
```
We can also populate directly from a JSON object:
```javascript
var json = { someNumber: 1, someString: "1" };
someRequest = new Service.TestRequest(json);
// someRequest.someNumber == 1
// someRequest.someString == "1"
```
Trying to set a property's value to an invalid type will be discarded:
```javascript
someRequest.PersonElement = null;
someRequest.PersonElement = new SomeRandomObject();
// someRequest.PersonElement == null;
```
If we have an array of objects there's a helper function to save typing:
```javascript
someRequest.PeopleList = new Service.Types.ArrayOfPeople;
someRequest.PeopleList.newChild({ firstname: "Oli", age: 24 });
// is the equivalent of:
someRequest.PeopleList = new Service.Types.ArrayOfPeople;
var newPerson = new Service.Types.Person();
newPerson.firstname = "Oli";
newPerson.age = 24;
someRequest.PeopleList.push(newPerson);
```
Making the request is trivial:
```javascript
someRequest.request(function(err, response) {
  // 'response' is a modeled object, it WILL conform to the WSDL.
  //... w00p!
});
```
Once we have a request and we want to edit it by adding properties not found in the WDSL, we must first extract the data from the response:
```javascript
someRequest.request(function(err, response) {
  response.SomeInvalidProperty = "testing";
  // response.SomeInvalidProperty == null
  var myResponse = response.extract();
  myResponse.SomeInvalidProperty = "testing";
  // myResponse.SomeInvalidProperty == "testing"
});
```

## Runtime Settings and Debugging
```javascript
var Service = require("./[serviceName]");

// This next statement will enable debugging for ALL soap requests
// It prints to stdout JSON objects, XML documents, etc
// default: false
Service.Settings.debugSoap = true;

// This next statement will enable benchmarking for ALL soap requests
// It prints to stdout the name of each request and its duration in ms
// default: true
Service.Settings.benchmark = true;

// This next statement will store the most recent request of each type to file
// It outputs to [/path/to/generated/code]/Mocks/[request-name]
// default: false
Service.Settings.createMock = true;

// This next statement will use saved mock requests instead of real requests
// default: false
Service.Settings.useMock = true;

// We can debug single SOAP requests by using the .debug() function, which is
// a property of every request and response object
var additionRequest = new Service.MathService.AdditionFunction(json);
additionRequest.debug(); // Watch your console output
additionRequest.request(function(err, response) {
  response.debug(); // Watch your console output
  if (err || !response) {
    return callback(err || "No response?");
  }
});
```

