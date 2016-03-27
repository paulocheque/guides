_Understanding this tutorial requires prior knowledge of JavaScript basics._

In this article, I will introduce the basics of Test-Driven Development (TDD) by walking through the creation and development of a small project in a test-driven way. 

The first part of this tutorial focuses on unit tests, and the last part focuses on code coverage.

If you want to experiment with the project yourself, there is a [repository hosted on GitHub](https://github.com/peterolson/Introduction-to-Test-Driven-Development-in-Javascript) with all of the code from this article.

Through this article, I hope to demonstrate how TDD works. As such, please feel free to skim through the nitty-gritty particulars if you would like to understand the big picture.

## What is test-driven development?

Traditionally, the software development workflow comprises the following steps:

 1. Think about the task (design).
 2. Write the code to accomplish the task (implementation).
 3. Test the code to see if it works (testing).

Often, writing a successful program requires multiple loops or repeated testing. As a result, testing and revising code comprehensively can take a rather long time.

**Test-driven development changes this workflow process**. TDD involves writing automated tests *before* we write the code. Here is our new workflow:

 1. Think about the task (design).
 2. Write tests specifying what you expect your code to do (test setup).
 3. Write the code to accomplish the task (implementation).
 4. See if the code works by running it against the tests you wrote (testing).

So we start writing some test code that specifies the expected output before we write the function or method. This is called *unit testing*. 

For a simple add(int,int) function that sums two integers, a unit test might look something like this:

```javascript
expect(add(1,1)).toEqual(2);
expect(add(5,7)).toEqual(12);
expect(add(-4,5)).toEqual(1);
```
and so on for as many test examples as we like. After creating these tests, we will write the actual add function. When we're finished, we can run the test code, and it will tell us whether our function passes all the tests:

> 2 of 3 tests passed. 1 test failed:  
> Expected `add(-4,5)` to return `1`, but got `-9`.

OK, so we failed 1 of the tests. We'll revise the code to try to correct the mistake, and then we'll run the tests again:

> 1 of 3 tests passed. 2 tests failed:  
> Expected `add(1,1)` to return `2`, but got `0`.  
> Expected `add(5,7)` to return `12`, but got `-2`.

Oops, we fixed one thing but broke other things at the same time. We'll revise the code again and see if there are still any problems:

> 3 of 3 tests passed. Yay!

Of course, just because our code passed the tests it doesn't mean the code works in general, but **it does give us a little more confidence about its correctness**. And if we later find a bug that our tests missed, we can always modify our tests for better coverage.

At this point, you might object, "*But what's the point of that? Isn't this procedure just an extra step?*"

Admittedly, setting up the testing environment and understanding how to unit write tests often takes some effort. In the short run, it's faster to just do things the traditional way. But in the long run, TDD can save time that would otherwise be wasted by repeated manual testing. And there are a number of other benefits to unit testing:

 - **Automatic regression detection** --- Sometimes coders mistakenly create bugs that make their programs defective, or they reintroduce old bugs that had previously been fixed. Such mistakes are called *regressions*. Regressions might sneak by unnoticed for a long time if you don't use automated testing. If you create a unit test for each bug that you fix, any code that passes your unit tests afterwards is guaranteed to be free of any old bugs. As a result, if you encounter errors in the program later on, you can be sure that regression is not the issue. 

 - **Bold refactoring** --- Code can get messy pretty quickly whenever we add workarounds to accomplish edge cases and unforeseen tasks. And cleaning up your code could lead to regressions. If you have unit tests covering specifically nasty cases, however, you will know when you have broken something _as well as where your mistake occurred_. As a result, TDD facilitates code cleanup.

 - **Documentation** --- If another developer cannot figure out how to use your code, they can look at your unit tests to see how the code was designed to be used. Unit tests aren't a replacement for real documentation, of course, but they are certainly better than no documentation at all. Unfortunately, having little to no documentation is common because tedious documentation work rarely makes it up a programmers to-do list.

 - **Robustness** --- Without automated testing, it is easy for the code to a complex application to feel very fragile. You might have a nagging anxiety that the slightest unexpected action from the user or the slightest future modification to the code will cause everything to crash and burn. Knowing that your code passes a set of unit tests is more reassuring than knowing that your code seemed to work when you manually tested it with a handful of examples the other day.

Now let's see how TDD works in practice.

# Practical example

In this example, we will go through the process of developing a simple date library in a test-driven way. For each part of the library, we will first write unit tests specifying how we want the date library to behave. Then we will write code to implement that behavior. If a particular test fails, we know that the corresponding implementation does not match its specification.

This date library is simply for testing purposes. It is _not_ intended to be a feature-complete library. If you are looking for a way to create a complete date library, I recommend [Moment.js](http://momentjs.com/).

## Setting up the testing environment

First, install a testing library. [QUnit](https://qunitjs.com/), [Mocha](https://mochajs.org/), and [Jasmine](http://jasmine.github.io/2.4/introduction.html) are the most popular ones today. In my opinion, they both essentially do the same thing, so I've arbitrarily chosen to use Jasmine. 

Here's how I set things up:

 1. Create a folder for this project with a subfolder named `test`.
 2. Download the [Jasmine standalone zip](https://github.com/jasmine/jasmine/releases) (I used version 2.4.1) and extract it in your `test` folder.
 3. If you open the `SpecRunner.html` file, you should see something like this: 
 ![enter image description here](http://i.stack.imgur.com/ZPBiJ.png)
 4. In the parent folder, create a file named `DateTime.js`, and in the `test/spec` folder create a file named `DateTimeSpec.js`. Delete the other files in the `spec` folder; we don't need them anymore.
 5. Edit the head of the `SpecRunner.html` file to reference our scripts. It should look something like this:

```html
<script src="lib/jasmine-2.4.1/jasmine.js"></script>
<script src="lib/jasmine-2.4.1/jasmine-html.js"></script>
<script src="lib/jasmine-2.4.1/boot.js"></script>
 
<!-- include source files here... -->
<script src="../DateTime.js" data-cover></script>
 
<!-- include spec files here... -->
<script src="spec/DateTimeSpec.js"></script>
```

If you refresh `SpecRunner.html` now, it should say "No specs found" since we haven't written anything yet.

With that out of the way, now we can start building our library.

## DateTime.js API

Feel free to quickly skim through this section to just get a basic idea of what our date library will do. There's no need to remember every detail in here; you can always refer back to this section if you are confused about the code's intended behavior.

`DateTime` is a function that constructs dates in one of the following ways:

 - `DateTime()`, called with no arguments, creates an object representing the current date/time.
   
    ```javascript
    var d = DateTime(); // d represents the current date/time.
    ```
   
 - `DateTime(date)`, called with one argument `date`, a native JavaScript `Date` object, creates an object representing the date/time corresponding to `date`.

    ```javascript
    var d = DateTime(new Date(0)); // d represents 1 Jan 1970 00:00:00 GMT
    ```

 - `DateTime(dateString, formatString)`, called with two arguments `dateString` and `formatString`,  returns an object representing the date/time encoded in `dateString`, which is interpreted using the format specified in `formatString`.

    ```javascript
    var d = DateTime("1/5/2012", "D/M/YYYY");
    ```

The object returned by `DateTime` will have the following method

 - `toString(formatString?)` - returns a string representation of the date, using the optional `formatString` argument to specify how the output should be formatted. If no `formatString` is provided, it will default to `"YYYY-M-D H:m:s"`.

    ```javascript
    > DateTime().toString()
    "2016-2-22 15:06:42"
    > DateTime().toString("MMMM Do, YYYY")
    "February 22nd, 2016"
    ```
        
and the following properties

 - `year` - the full (usually 4-digit) year (e.g. 1997). Represented in a format string as `YYYY`. Readable/writable.
 - `monthName` - the name of the month (e.g. December). Represented in a format string as `MMMM`. Readable/writable.
 - `month` - the number of the month (e.g. `12` for December). Represented in a format string as `M`. Readable/writable.
 - `day` - the name of the weekday (e.g. Tuesday). Represented in a format string as `dddd`. Readonly.
 - `date` - the date of the month (e.g. 22). Represented in a format string as `D`. Readable/writable.
 - `ordinalDate` - the ordinal date of the month (e.g. 22nd). Represented in a format string as `Do`. Readable/writable.
 - `hours` - a number from 0 to 23, corresponding to the hour. Represented in a format string as `H`. Readable/writable.
 - `hours12` - a number from 1 to 12, corresponding to the hour in the 12-hour system. Represented in a format string as `h`. Readable/writable.
 - `minutes` - a number from 0 to 59, corresponding to the minute. Represented in a format string as `m`. Readable/writable.
 - `seconds` - a number from 0 to 59, corresponding to the minute. Represented in a format string as `s`. Readable/writable.
 - `ampm` - a string, either `"am"` or `"pm"`. Represented in a format string as `a`. Readable/writable.
 - `offset` - returns the Unix offset, that is, the number of milliseconds since January 1st, 1970, 00:00:00. Readable/writable.

## Getting started

Here is a brief explanation of some Jasmine functions. You can read more details from the [Jasmine docs](http://jasmine.github.io/2.4/introduction.html).

 - `describe("DateTime", ...)` - this creates a test group (or "suite") named "DateTime". We will put any tests specifying the behavior of `DateTime` inside of this test suite.
 - `it(testName, ...)` - this creates a test (or "spec") with the given name. As a general rule, the spec name should read like the predicate of a sentence with the name of the test group as an implied subject (in our case, `DateTime`). For this reason, the function is named `it` to encourage you to read the spec name like a sentence starting with the word "it".
   - Good name: [It] returns the current time when called with no arguments
   - Bad name: [It] no arguments returns current time
 - `expect` - this specifies what the spec expects to happen. If all the expectations in a spec are met, then the spec passes. If at least one expectation is not met, or an unhandled exception is thrown, then the spec fails.

The first thing we need to do is to decide on some minimal subset of the API to implement, and then build on top of that until we're finished. We will start by writing a unit test that specifies what we expect `DateTime` to do. 

In`DateTimeSpec.js`, create the first few unit tests. 

```javascript
describe("DateTime", function () {
    it("returns the current time when called with no arguments", function () {
        var lowerLimit = new Date().getTime(),
            offset = DateTime().offset,
            upperLimit = new Date().getTime();
        expect(offset).not.toBeLessThan(lowerLimit);
        expect(offset).not.toBeGreaterThan(upperLimit);
    });
});
```

Now open `SpecRunner.html` and click on "Spec List". You should see 1 failing test:

![1 spec, 1 failure; DateTime uses the current time when called with no arguments](http://i.stack.imgur.com/zNO6v.png)

If you click "Failures" you will see some message about a ReferenceError because DateTime is not defined. _That is exactly what should happen, since we haven't written any code defining `DateTime` yet._

   In this case, we want to see whether `DateTime()` actually returns the current time. In the test code above, I recorded the time before and after we called `DateTime()`, and then tested if the time returned by `DateTime` was between these two limits.

Now that we've finished writing our first test, we can write code to implement the features we are testing. In the `DateTime.js` file, paste the following code that **creates our DateTime object**:

```javascript
"use strict";
var DateTime = (function () {

    function createDateTime(date) {
        return {
            get offset() {
                return date.getTime();
            }
        };
    }

    return function () {
        return createDateTime(new Date());
    };
})();
```

When we open `SpecRunner.html` our test should pass because DateTime is now a defined object:

![1 spec, 0 failures](http://i.stack.imgur.com/KHMPk.png)

Great, now we've completed our first development iteration.

A reasonable next step is to implement the `DateTime(date)` constructor. First, we write a unit test for it:

```javascript
it("matches the passed in Date when called with one argument", function () {
    var dates = [new Date(), new Date(0), new Date(864e13), new Date(-864e13)];
    for (var i = 0; i < dates.length; i++) {
        expect(DateTime(dates[i]).offset).toEqual(dates[i].getTime());
    }
});
```

I've chosen four dates to test: the current date, and three dates that are potential edge cases: 

 - the epoch (January 1, 1970), 
 - [the maximum valid JavaScript date](http://stackoverflow.com/q/12666127/546661) (September 13, 275760, i.e. 100 million days after the epoch), and
 -  [the minimum valid JavaScript date](http://stackoverflow.com/q/12666127/546661) (April 20, 271822 B.C., i.e. 100 million days before the epoch).

Testing all of these may seem a little superfluous. After all, we're only writing a wrapper around the native `Date` object and there's not any complicated logic going on. Nonetheless, as a general rule, it's a good idea to use potential edge cases in your tests to increase the chances of finding bugs sooner.

There are still two important issues we haven't specified anything about yet in our tests.

 1. What should happen when we pass in a single argument to `DateTime` that is not a `Date` object?
 2. What should happen when we pass in a `Date` object that is invalid, such as `new Date(1e99)` or `new Date("Invalid string")`?

There are lots of possible answers to these two questions depending on how you choose to handle your errors. I typically choose one of the following strategies for dealing with these issues:

 1. Throw an error.
 2. Continue without throwing an error. All the native `Date` methods we use will return `NaN` in this situation, so our `DateTime` properties and methods will propagate this behavior automatically.

Regardless of what behavior we might decide on, we should write tests that codify that behavior:

```javascript
it("throws an error when called with a single non-Date argument", function () {
    var nonDates = [0, NaN, Infinity, "", "not a date", null, /regex/, {}, []];
    for (var i = 0; i < nonDates.length; i++) {
        expect(DateTime.bind(null, nonDates[i])).toThrow();
    }
});
it("returns a NaN offset when an invalid date is passed in", function () {
    var invalidDates = [new Date(864e13 + 1), new Date(-1e99), new Date("xyz")];
    for (var i = 0; i < invalidDates.length; i++) {
        expect(isNaN(DateTime(invalidDates[i]).offset)).toBe(true);
    }
});
```

This is fairly straightforward, except for the `DateTime.bind` part. The [function binding](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind) is used because (1) `.toThrow()` assumes a function was passed to `expect`, and (2) [creating a function inside of a loop in the straightforward way behaves somewhat counter-intuitively in JavaScript](http://stackoverflow.com/q/750486/546661).

When we open `SpecRunner.html` now we should see that the three specs we just wrote all failed. This is an important thing to check: **if a spec passes before we write the implementation code, that usually means we made a mistake while writing the spec.**

Now we can implement the code:

```javascript
// ...
return function (date) {
    if (date !== undefined) {
        if (date instanceof Date) {
            return createDateTime(date);
        }
        throw new Error(String(date) + " is not a Date object.");
    }
    return createDateTime(new Date());
};
```

At this point, all of the tests should pass.

## Getters

The easiest next step is to implement the property getters. Writing tests for all of them is straightforward, although a little tedious. Simply loop through some test dates and make sure that all the property getters return the expected values.

When choosing test dates it's a good idea to include both typical dates as well as edge cases.

```javascript
var testDates = [
    "0001-01-01T00:00:00", // Monday
    "0021-02-03T07:06:07", // Wednesday
    "0321-03-06T14:12:14", // Sunday
    "1776-04-09T21:18:21", // Tuesday
    "1900-05-12T04:24:28", // Saturday
    "1901-06-15T11:30:35", // Saturday
    "1970-07-18T18:36:42", // Saturday
    "2000-08-21T01:42:49", // Monday
    "2008-09-24T08:48:56", // Wednesday
    "2016-10-27T15:54:03", // Thursday
    "2111-11-30T22:01:10", // Monday
    "9999-12-31T12:07:17", // Friday
    864e13, // max date (Sat 13 Sep 275760 00:00:00) 
    -864e13, // min date (Tue 20 Apr -271821 00:00:00)
    -623e11 // single-digit negative year (Tue 17 Oct -5 04:26:40)
].map(function (x) {
    return DateTime(new Date(x));
});

var expectedValues = {
    year: [1, 21, 321, 1776, 1900, 1901, 1970, 2000, 2008, 2016, 2111, 9999, 275760, -271821, -5],
    monthName: ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December", "September", "April", "October"],
    month: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 9, 4, 10],
    day: ["Monday", "Wednesday", "Sunday", "Tuesday", "Saturday", "Saturday", "Saturday", "Monday", "Wednesday", "Thursday", "Monday", "Friday", "Saturday", "Tuesday", "Tuesday"],
    date: [1, 3, 6, 9, 12, 15, 18, 21, 24, 27, 30, 31, 13, 20, 17],
    ordinalDate: ["1st", "3rd", "6th", "9th", "12th", "15th", "18th", "21st", "24th", "27th", "30th", "31st", "13th", "20th", "17th"],
    hours: [0, 7, 14, 21, 4, 11, 18, 1, 8, 15, 22, 12, 0, 0, 4],
    hours12: [12, 7, 2, 9, 4, 11, 6, 1, 8, 3, 10, 12, 12, 12, 4],
    minutes: [0, 6, 12, 18, 24, 30, 36, 42, 48, 54, 1, 7, 0, 0, 26],
    seconds: [0, 7, 14, 21, 28, 35, 42, 49, 56, 3, 10, 17, 0, 0, 40],
    ampm: ["am", "am", "pm", "pm", "am", "am", "pm", "am", "am", "pm", "pm", "pm", "am", "am", "am"],
    offset: [-62135596800000, -61501568033000, -52031843266000, -6113414499000, -2197654532000, -2163155365000, 17174202000, 966822169000, 1222246136000, 1477583643000, 4478364070000, 253402258037000, 8640000000000000, -8640000000000000, -62300000000000]
};

describe("getter", function () {
    Object.keys(expectedValues).forEach(function (propertyName) {
        it("returns expected values for property '" + propertyName + "'", function () {
            testDates.forEach(function (testDate, i) {
                expect(testDate[propertyName]).toEqual(expectedValues[propertyName][i]);
            });
        });
    });
});
```

All the new tests we just wrote **should fail now**, except the one corresponding to the `offset` property, since we already implemented the getter for `offset`.

Now that we've written the tests we can write the implementation code.

```javascript
var monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"],
    dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

function createDateTime(date) {
    return {
        get year() {
            return date.getFullYear();
        },
        get monthName() {
            return monthNames[date.getMonth()];
        },
        get month() {
            return date.getMonth() + 1;
        },
        get day() {
            return dayNames[date.getDay()];
        },
        get date() {
            return date.getDate();
        },
        get ordinalDate() {
            var n = this.date;
            var suffix = "th";
            if (n < 4 || n > 20) {
                suffix = ["st", "nd", "rd"][n % 10 - 1] || suffix;
            }
            return n + suffix;
        },
        get hours() {
            return date.getHours();
        },
        get hours12() {
            return this.hours % 12 || 12;
        },
        get minutes() {
            return date.getMinutes();
        },
        get seconds() {
            return date.getSeconds();
        },
        get ampm() {
            return this.hours < 12 ? "am" : "pm";
        },
        get offset() {
            return date.getTime();
        }
    };
}
```
    
I made quite a few mistakes in the process of writing the code above that the tests helped me catch. While most of these errors were trivial and easy to spot, there is one subtle bug that I have intentionally left in the code above. 

When I run the tests, I see 8 failed specs. Here is one of my failed expectations:

> DateTime getter returns expected values for property 'monthName'  
> Expected 'December' to equal 'November'.

This is caused by the `2111-11-30T22:01:10` date. My `monthName` getter says this is December instead of November. This is because I live in the GMT+8 timezone, so something behind the scenes is converting the time from GMT into my timezone, resulting in `2111-12-01 06:01:10`. So we must consider time zone when creating our date library.

The solution to this problem is to use the `getUTCMonth` method instead of the `getMonth` method to prevent this conversion:
  
```javascript
get monthName() {
    return monthNames[date.getUTCMonth()];
},
```

The same logic applies to the other methods, like `getFullYear`/`getUTCFullYear`, `getDay`/`getUTCDay`, and so forth:

```javascript
return {
    get year() {
        return date.getUTCFullYear();
    },
    get monthName() {
        return monthNames[date.getUTCMonth()];
    },
    get month() {
        return date.getUTCMonth() + 1;
    },
    get day() {
        return dayNames[date.getUTCDay()];
    },
    get date() {
        return date.getUTCDate();
    },
    get ordinalDate() {
        var n = this.date;
        var suffix = "th";
        if (n < 4 || n > 20) {
            suffix = ["st", "nd", "rd"][n % 10 - 1] || suffix;
        }
        return n + suffix;
    },
    get hours() {
        return date.getUTCHours();
    },
    get hours12() {
        return this.hours % 12 || 12;
    },
    get minutes() {
        return date.getUTCMinutes();
    },
    get seconds() {
        return date.getUTCSeconds();
    },
    get ampm() {
        return this.hours < 12 ? "am" : "pm";
    },
    get offset() {
        return date.getTime();
    }
};
```

After these modifications, all the tests should pass now. This type of bug is a bit nasty because the unit tests might not catch it if your timezone is close to GMT. **But I'm not sure that I would have even noticed it if I hadn't written the unit tests.**

## Setters

Now that we've implemented all the getters, the obvious next step is to implement all the setters.

Fortunately, to write tests for the setters, we don't need to create any more test dates or expected values, we can just reverse the process we used for the getter tests. Before, we had some date, such as`2008-09-24T08:48:56`, and we were checking that the year property returned `2008`, the month property returned `9`, and so on. 

This time, we will create a date, set its year property to `2008`, set its month property to `9`, and so forth, and check that its offset is the same as the offset for `2008-09-24T08:48:56`. 

We have some overlapping properties, like `month` and `monthName` that set the same information, and `offset` which affects everything else, so we will do three passes to test all of the properties:

 - In the first pass we'll use the `year`, `month`, `date`, `hours`, `minutes`, and `seconds` properties.
 - In the second pass we'll use the `year`, `monthName`, `ordinalDate`, `ampm`, `hours12`, `minutes`, and `seconds` properties.
 - In the third pass we'll use the `offset` property.

Here is the code for the setter unit tests:

```javascript
describe("setter", function () {
    var settableProperties = [["seconds", "minutes", "hours", "date", "month", "year"],
        ["seconds", "minutes", "hours12", "ampm", "ordinalDate", "monthName", "year"],
        ["offset"]];
    it("can reconstruct a date using the property setters", function () {
        testDates.forEach(function (date, i) {
            settableProperties.forEach(function (properties) {
                var date = DateTime(new Date(0));
                properties.forEach(function (property) {
                    date[property] = expectedValues[property][i];
                });
                expect(date.offset).toEqual(expectedValues.offset[i]);
            });
        });
    });
});
```

As usual, when you run these tests, they should all fail since we haven't written the setter code yet.

Finally, the only property left is the `day` property, which is read-only. _In JavaScript, writing to read-only properties fails silently by default:_

```javascript
> var obj = { get readonlyProperty() { return 1; } };
> obj.readonlyProperty
1
> obj.readonlyProperty = 2;
> obj.readonlyProperty
1
```

In my opinion, this silent failure is bad design. Without a warning when you try to write to a property without a setter, you might waste time later trying to figure out why it didn't work. Instead, we should throw an error when an attempt is made to write to `day`. 

**This modification is another use case of TDD testing.** So let's specify that with a test:

```javascript
it("throws an error on attempt to write to property 'day'", function () {
    expect(function () {
        var date = DateTime();
        date.day = 4;
    }).toThrow();
});
```

Again, this test should fail since we haven't written the implementation yet.

Here is the code for the setters:

```javascript
set year(v) {
    date.setUTCFullYear(v);
},
set month(v) {
    date.setUTCMonth(v - 1);
},
set monthName(v) {
    var index = monthNames.indexOf(v);
    if (index < 0) {
        throw new Error("'" + v + "' is not a valid month name.");
    }
    date.setUTCMonth(index);
},
set day(v) {
    throw new Error("The property 'day' is readonly.");
},
set date(v) {
    date.setUTCDate(v);
},
set ordinalDate(v) {
    date.setUTCDate(+v.slice(0, -2));
},
set hours(v) {
    date.setUTCHours(v);
},
set hours12(v) {
    date.setUTCHours(v % 12);
},
set minutes(v) {
    date.setUTCMinutes(v);
},
set seconds(v) {
    date.setUTCSeconds(v);
},
set ampm(v) {
    if (!/^(am|pm)$/.test(v)) {
        throw new Error("'" + v + "' is not 'am' or 'pm'.");
    }
    if (v !== this.ampm) {
        date.setUTCHours((this.hours + 12) % 24);
    }
},
set offset(v) {
    date.setTime(v);
}
```

All our setter tests should pass now.

## Formatting and parsing

The only things left now are the `DateTime(dateString, formatString)` constructor and the `toString(formatString?)` method. The two of these are related, since they both involve a format string.

We can reuse the same test dates from before, but we need to specify what strings we expect from them given different formats:

```javascript
var expectedStrings = {
    "YYYY-M-D H:m:s": ["1-1-1 0:00:00", "21-2-3 7:06:07", "321-3-6 14:12:14", "1776-4-9 21:18:21", "1900-5-12 4:24:28", "1901-6-15 11:30:35", "1970-7-18 18:36:42", "2000-8-21 1:42:49", "2008-9-24 8:48:56", "2016-10-27 15:54:03", "2111-11-30 22:01:10", "9999-12-31 12:07:17", "275760-9-13 0:00:00", "-271821-4-20 0:00:00", "-5-10-17 4:26:40"],
    "dddd, MMMM Do YYYY h:m:s a": ["Monday, January 1st 1 12:00:00 am", "Wednesday, February 3rd 21 7:06:07 am", "Sunday, March 6th 321 2:12:14 pm", "Tuesday, April 9th 1776 9:18:21 pm", "Saturday, May 12th 1900 4:24:28 am", "Saturday, June 15th 1901 11:30:35 am", "Saturday, July 18th 1970 6:36:42 pm", "Monday, August 21st 2000 1:42:49 am", "Wednesday, September 24th 2008 8:48:56 am", "Thursday, October 27th 2016 3:54:03 pm", "Monday, November 30th 2111 10:01:10 pm", "Friday, December 31st 9999 12:07:17 pm", "Saturday, September 13th 275760 12:00:00 am", "Tuesday, April 20th -271821 12:00:00 am", "Tuesday, October 17th -5 4:26:40 am"],
    "YYYY.MMMM.M.dddd.D.Do.H.h.m.s.a": ["1.January.1.Monday.1.1st.0.12.00.00.am", "21.February.2.Wednesday.3.3rd.7.7.06.07.am", "321.March.3.Sunday.6.6th.14.2.12.14.pm", "1776.April.4.Tuesday.9.9th.21.9.18.21.pm", "1900.May.5.Saturday.12.12th.4.4.24.28.am", "1901.June.6.Saturday.15.15th.11.11.30.35.am", "1970.July.7.Saturday.18.18th.18.6.36.42.pm", "2000.August.8.Monday.21.21st.1.1.42.49.am", "2008.September.9.Wednesday.24.24th.8.8.48.56.am", "2016.October.10.Thursday.27.27th.15.3.54.03.pm", "2111.November.11.Monday.30.30th.22.10.01.10.pm", "9999.December.12.Friday.31.31st.12.12.07.17.pm", "275760.September.9.Saturday.13.13th.0.12.00.00.am", "-271821.April.4.Tuesday.20.20th.0.12.00.00.am", "-5.October.10.Tuesday.17.17th.4.4.26.40.am"]
};
```

Once we've constructed this object, writing the tests is straightforward:

 - For `toString`, we go through all the test dates (e.g. `1970-07-18T18:36:42`) and see if they return the expected string for each of the formats (e.g.  `"1970-7-18 18:36:42"` for `"YYYY-M-D H:m:s"`).
 - For the `DateTime(dateString, formatString)` constructor, we go through all the pairs of formats and date strings (e.g. `"1970-7-18 18:36:42"` and `"YYYY-M-D H:m:s"`) to make sure that each constructed object has the same offset as the corresponding test date.

Or, as expressed in code:

```javascript
describe("toString", function () {
    it("returns expected values", function () {
        testDates.forEach(function (date, i) {
            for (format in expectedStrings) {
                expect(date.toString(format)).toEqual(expectedStrings[format][i]);
            }
        });
    });
});

it("parses a string as a date when passed in a string and a format string", function () {
    for (format in expectedStrings) {
        expectedStrings[format].forEach(function (date, i) {
            expect(DateTime(date, format).offset).toEqual(testDates[i].offset);
        });
    }
});
```

As usual, these tests should fail if we run them now.

Here's the implementation code to add these features. **This is the most complicated part of the library**, so the code here is not as simple as the code we've written up to this point. Feel free to just skim through this code to get the big picture without analyzing the finer details.

```javascript
"use strict";
var DateTime = (function () {

    var monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"],
        dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

    function createDateTime(date) {
        return {
            // ... skipping the getters/setters to save space
            toString: function (formatString) {
                formatString = formatString || "YYYY-M-D H:m:s";
                return toString(this, formatString);
            }
        };
    }

    var formatAbbreviations = {
        YYYY: "year",
        YY: "shortYear",
        MMMM: "monthName",
        M: "month",
        dddd: "day",
        D: "date",
        Do: "ordinalDate",
        H: "hours",
        h: "hours12",
        m: "minutes",
        s: "seconds",
        a: "ampm"
    };
    var maxAbbreviationLength = 4;
    var formatPatterns = {
        year: /^-?[0-9]+/,
        monthName: /January|February|March|April|May|June|July|August|September|October|November|December/,
        month: /[1-9][0-9]?/,
        day: /Sunday|Monday|Tuesday|Wednesday|Thursday|Friday|Saturday/,
        date: /[1-9][0-9]?/,
        ordinalDate: /[1-9][0-9]?(st|nd|rd|th)/,
        hours: /[0-9]{1,2}/,
        hours12: /[0-9]{1,2}/,
        minutes: /[0-9]{1,2}/,
        seconds: /[0-9]{1,2}/,
        ampm: /am|pm/
    };


    function tokenize(formatString) {
        var tokens = [];
        for (var i = 0; i < formatString.length; i++) {
            var slice, propertyName;
            for (var j = maxAbbreviationLength; j > 0; j--) {
                slice = formatString.slice(i, i + j);
                propertyName = formatAbbreviations[slice];
                if (propertyName) {
                    tokens.push({ type: "property", value: propertyName });
                    i += j - 1;
                    break;
                }
            }
            if (!propertyName) {
                tokens.push({ type: "literal", value: slice });
            }
        }
        return tokens;
    }

    function toString(dateTime, formatString) {
        var tokens = tokenize(formatString);
        return tokens.map(function (token) {
            if (token.type === "property") {
                var value = dateTime[token.value];
                if (token.value === "minutes" || token.value === "seconds") {
                    value = ("00" + value).slice(-2);
                }
                return value;
            }
            return token.value;
        }).join("");
    }

    function parse(string, formatString) {
        var tokens = tokenize(formatString);
        var properties = {};
        tokens.forEach(function (token, i) {
            if (token.type === "literal") {
                var value = token.value,
                    slice = string.slice(0, value.length);
                if (slice !== value) {
                    throw new Error("String does not match format. Expected '" + slice + "' to equal '" + value + "'.");
                }
                string = string.slice(value.length);
            } else {
                var format = token.value,
                    pattern = formatPatterns[format],
                    match = string.match(pattern);
                if (!match || !match.length) {
                    throw new Error("String does not match format. Expected '" + string + "' to start with the pattern " + pattern + ".");
                }
                match = match[0];
                string = string.slice(match.length);
                properties[format] = match;
            }
        });
        var propertyOrder = ["seconds", "minutes", "hours12", "ampm", "hours", "ordinalDate", "date", "monthName", "month", "year"];
        var date = createDateTime(new Date(0));
        propertyOrder.forEach(function (property) {
            if (properties[property]) {
                date[property] = properties[property];
            }
        });
        return date;
    }

    return function (date, formatString) {
        if (date !== undefined) {
            if (date instanceof Date) {
                return createDateTime(date);
            }
            if (typeof formatString === "string") {
                return parse(date, formatString);
            }
            throw new Error(String(date) + " is not a Date object.");
        }
        return createDateTime(new Date());
    };
})();
```

Now all tests should pass. The unit tests were the most useful during this part; the pre-designed tests helped me **quickly** spot bugs that would otherwise get lost in the framework of my code. 

I haven't detailed all of the iterations of mistakes and fixes that I went through writing this, since they were mostly trivial and uninteresting mistakes. That said, I do want to point out how illuminating (and humbling) this process is: the number of mistakes you make while writing code can be surprisingly large.

It might seem like we're finished now, since we've written all of the features and all the tests pass, but there's one more step we should go through to see if our tests are thorough enough.

# Code coverage

Code coverage tools are used to help you find untested code. They attach counters to each statement in the code to **alert you of any statements that are never executed**. Code coverage is often expressed as a percentage; for example, 85% code coverage means that 85% of the statements in the code were executed.

**If you have low code coverage, it is usually a good indication that your tests are incomplete.** Of course, having 100% code coverage is no guarantee that your unit tests can catch every potential bug, but, in general, there are more defects in untested code than in tested code.

Code coverage is an especially useful tool when writing tests for large projects that lack pre-existing unit tests.

## Setting up code coverage

For this we will need to install [Node.js](https://nodejs.org/en/).

We will use [Karma](http://karma-runner.github.io/0.13/index.html) for running the code coverage tests. In the instructions below, I assume that you have Google Chrome, [but it's easy to modify which browser you use](http://karma-runner.github.io/0.10/config/browsers.html).

Create a file in your project called `package.json` with the following content:

```javascript
{
  "scripts": {
    "test": "node_modules/.bin/karma start my.conf.js"
  },
  "devDependencies": {
    "jasmine-core": "^2.3.4",
    "karma": "^0.13.21",
    "karma-coverage": "^0.5.3",
    "karma-jasmine": "^0.3.6",
    "karma-chrome-launcher": "~0.1"
  }
}
```

Then create another file named `my.conf.js` with the following content:

```javascript
module.exports = function(config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine'],
    files: [
      'DateTime.js',
      'test/spec/*.js'
    ],
    browsers: ['Chrome'],
    singleRun: true,
    preprocessors: { '*.js': ['coverage'] },
    reporters: ['progress', 'coverage']
  });
};
```

If you use Windows, open the Node.js command prompt. Otherwise, just open your terminal and navigate to your project folder. Then run `npm install`. 

Once it's done installing, you can run `npm test` whenever you want to run the coverage tests. It will create a `coverage` folder with a subfolder corresponding to your browser name. Open the `index.html` file in that folder to see the code coverage report.

## Going through the code coverage report

The code coverage highlights unexecuted code in red, 

![unexecuted line of code](http://i.stack.imgur.com/nWlVW.png) and unevaluated logical branches in yellow.

![unevaluated conditional branch](http://i.stack.imgur.com/RzoRZ.png)

At this point, the code coverage report shows that the unit tests cover 96% of the lines of code and 87% of the conditional branches. Going through the report and inspecting the highlighted code reveals what our unit tests are missing:

 - There are no tests that use the default format string in the `toString` method. We can add some to the `describe("toString", ...)` section:

    ```javascript
    it("uses YYYY-M-D H:m:s as the default format string", function () {
        testDates.forEach(function (date, i) {
            expect(date.toString()).toEqual(expectedStrings["YYYY-M-D H:m:s"][i]);
        });
    });
    ```
 - There are no tests that attempt setting `monthName` to an invalid month name. We can add some to the `describe("setter", ...)` section:

    ```javascript
    it("throws an error on attempt to set property `monthName` to an invalid value", function () {
        var invalidMonths = ["janury", "???", "", 5];
        invalidMonths.forEach(function (value) {
            expect(function () {
                var date = DateTime();
                date.monthName = value;
            }).toThrow();
        });
    });
    ```
 - There are no tests that try to set `ampm` to a value other than `am` or `pm`. We can add some to the `describe("setter", ...)` section:

    ```javascript
    it("throws an error on attempt to set property `ampm` to an invalid value", function () {
        var invalidAMPM = ["afternoon", "p", "a", 0];
        invalidAMPM.forEach(function (value) {
            expect(function () {
                var date = DateTime();
                date.ampm = value;
            }).toThrow();
        });
    });
    ```
 - There are no tests that try to parse an invalid date string. We can add some to the `describe("DateTime", ...)` section:

    ```javascript
    it("throws an error when passed in an invalid date string and a format string", function () {
        var invalidDates = ["1234!5+3 1:2:3", "tomorrow", "Monday, January 500th 2000 5:40:30 pm", 0];
        Object.keys(expectedStrings).forEach(function (format) {
            invalidDates.forEach(function (invalidDate) {
                expect(function () {
                    DateTime(invalidDate, format);
                }).toThrow();
            });
        });
    });
    ```

Now the tests should cover 100% of the lines and branches of the code.


# Conclusion

Congrats! If you've read through this far, you should understand

 - unit testing and its benefits,
 - the ways of writing basic unit tests,
 - code coverage,
 - running coverage tests.

These steps should be enough for you to get started with TDD in your own projects.

If you work on collaborative projects, especially open-source ones, I would also recommend that you read up on Continuous Integration (CI) testing. [Travis CI](https://travis-ci.org/) is a popular CI server that automatically runs tests after every push to GitHub, and [Coveralls](https://coveralls.io/) is a server that runs code coverage tests after every push to GitHub.

I hope this (long) explanation elucidated the advantages of test-driven development. If you have any questions for me, feel free to leave a comment below.