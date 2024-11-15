---
title: "Looking at the Prototype"
description: "We are going to look at how the prototype and not ignore it"
tags: [prototype, JavaScript, Prototype Chain, Prototype Chain, __proto__]
categories: [Blogging, Tutorial]
excerpt_image: https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/pexels-anna-shvets-3746197.jpg
---


## Why I care about the prototype

To work with a lot of the cool flashy js frameworks, I was told to understand JavaScript first. Other than the scope of “this” one other concept would confuse me. The idea of prototypes would pop up, and people would always say “Don't worry about that. It is important but not really”. So for the longest, I just ignored the prototype. I could get what I needed and not really have to worry about it. After taking a real dive into es6 and understanding classes and inheritance.

## The breakdown

I was noticing that JavaScript was playing by different rules than C# or Java. Side note: I do Java sometimes when needed but not as I used to back in college. Back in my day, Java was the golden child, and we implemented in it. The C++ religion was dying down, and this took its place. It is interesting to see the same pattern happen with python. My professor Dr. Hang Chen (a very sharp man who took little bs. Then there was me, and I had a ton to give) at the time had the view that using Java was making us softer coders and real coders used C++ or C. It is just funny to see this same idea come up in a new era with js and python versus other compiled languages. Okay back to prototypes. I was attacking this with the mindset of a Java/C# coder.

## The What

![Image of a model prototype chain](https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/Proto.jpg "A model of the prototype chain")

Prototypical languages are just different. Now that I have a better understanding of them, I think they are kind of cool. So step one was understanding classical then prototypal inheritance. It took a second to understand that inheritance is different in this language and how simple it is. In JavaScript, all objects including functions have a prototype property. In the diagram above. The property is just a reference to another object we called proto. It would be an object that could stand on its own and could be independent if needed. So when you are calling prop 2 it is not on the object but it is actually on the object’s prototype. That prototype object can also point to another object. Each object can have its prototype. Let's use some code to see an example of the prototype chain. The prototype chain deals with where we have access to a property or method amid a sequence of objects. Those connected by this prototype property, here we are calling proto. The JavaScript engine will do the work for us by searching the prototype chain. We do not have to be explicit with our calls (Ex: we can just say object.prop2 and not object.proto.prop2) Let's see an example of this.


## The Code 

Here we created two objects with default values and a default function. The second object will have two properties with no function.

```js
// Let's create a car object with some Default values
var car = {
make: 'Default',
model: 'Default',
getCarInfo: function() {
return this.make + ' ' + this.model; 
}
}

var Volvo = {
make: 'Volvo',
model: 'S80'
}
```

Don't do this in real life. Learn about the [__proto__](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) here. We read docs around here. Now Volvo inherits from car. So when calling a method that doesn't exist on Volvo it will go to Car to find it

```js
Volvo.__proto__ = car;

// Now we can use the getCarInfo()
console.log(Volvo.getCarInfo());
```

We get the value of "Volvo" because of the prototype chain. It first looks on the Volvo object for that property, finds it then stops. JavaScript engine starts at the top of the chain, works its way down till it finds the needed value.

```js
console.log(Volvo.make);
```


This will return "Honda Default" as the object has a value for make but not for model. So the prototype is car and finds the model value of Default

```js
var Honda = {
    make: 'Honda'   
}

Honda.__proto__ = car;


console.log(Honda.getCarInfo());
```

Here we will crate a new function on the car object to hammer home the idea.

```js
car.getCarFullInfo = function() {
    return this.model + ', ' + this.make;   
}

console.log(Volvo.getCarFullInfo());
console.log(Honda.getCarFullInfo());
```

The value you would get here would be "S80, Volvo" and "Default, Honda" because of how prototypes work. [Get the full code here](https://github.com/KamRon-67/Proto-Example).