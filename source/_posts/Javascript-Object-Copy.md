---
title: Javascript Object Copy
date: 2016-06-22 17:21:07
tags: Javascript
---

## Prerequisite: Understanding objects assignment in Javascript

As you know, the assignment does not copy an object, it only assign a reference to it, therefore the following code:

```
var object = { a: 1, b: 2 } ;  
var copy = object ;  
object.a = 3 ;  
console.log( copy.a ) ;
```

... will output 3 rather than 1.

The two variables object & copy reference the same object, so whatever the variable used to modify it, you will get the same result.

## How to perform a deep copy of an object in Javascript

### Resolution 1:

A variant using Object.keys() can be used if we want to copy only owned and enumerable properties:

```
function cloneObject(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
 
    var temp = obj.constructor(); // give temp the original obj's constructor
    for (var key in obj) {
        temp[key] = cloneObject(obj[key]);
    }
 
    return temp;
}
```

### Resolution 2: if it is the json object.

```
var originalObj = {
    name: "Bob",
    age: 32,
    address:{
    	city: "CQ"",
    	Road: "JinShan"
    }
};

var targetObj = JSON.parse(JSON.stringify(originalObj))
```

### Resolution 3: Using jQuery’s [$.extend()](http://api.jquery.com/jQuery.extend/)

```
var originalObj = {
    name: "Bob",
    age: 32,
    address:{
    	city: "CQ"",
    	Road: "JinShan"
    }
};
 
var targetObj = $.extend(true, {}, originalObj);
```