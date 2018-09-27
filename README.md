# jeach-log4node

This is a fork of the [log4js-node](https://github.com/log4js-node/log4js-node) project.

I'm not trying to compete with the original library or anything like that. I just thought it was the closest thing to Log4J which I am accustomed to. The real reason I forked it is because I believe that method and line number tracking in log prints are crucial. I can't figure out why they have yet support for it, but they don't. So I decided to simply add it and make it public for anyone that requires the same.

So now, if you want to use the following pattern:

```
  "layout": {
    "type": "pattern",
    "pattern": "[%d][%-2p][%c][%M(%L)]> %m"
  }
```

It will provide logs as follows:

```
[2018-09-27T02:55:04.618][DEBUG][jeach-abc][methodThree(7)]> Lorem ipsum dolor sit amet ...
[2018-09-27T02:55:04.618][INFO][jeach-xyz][methodOne(18)]> Lorem ipsum dolor sit amet ...
[2018-09-27T02:55:04.637][DEBUG][jeach-abc][methodTwo(26)]> Lorem ipsum dolor sit amet ...
[2018-09-27T02:55:04.648][INFO][jeach-xyz][methodOne(8)]> Lorem ipsum dolor sit amet ...
[2018-09-27T02:55:04.653][DEBUG][jeach-abc][methodThree(12)]> Lorem ipsum dolor sit amet ...
```

Other than adding support for **Method (%M)** and **Line Number (%L)** formats, nothing else was added. So you can essentially consult the **log4j-node** documentation.

If you are curious at the changes I have made, simply look at the [layout.js](https://github.com/Jeach/log4node/blob/master/lib/layouts.js) file and search for **@author Christian Jean**. I have added my name to all the code additions and changes that I have made.

Here is a summary of the changes in **layout.js**:

On line 128, I added the 'M' and 'L' characters to the REGEX.

```javascript
  const regex = /%(-?[0-9]+)?(\.?[0-9]+)?([[\]MLcdhmnprzxXy%])(\{([^}]+)\})?|([^%]+)/;
```
Next, on line 144, I added the **getTrace(caller)** and **prepareStackTrace(error, structuredStackTrace)** functions which allows for the generation of a stack trace. This code was ripped off from the `log4js-extend.js` file.

```javascript
  function getTrace(caller) {
    var original = Error.prepareStackTrace, error = {};
    Error.prepareStackTrace = prepareStackTrace;
    Error.captureStackTrace(error, caller || getTrace);
    var stack = error.stack;
    Error.prepareStackTrace = original;
    return stack;
  }

  function prepareStackTrace(error, structuredStackTrace) {
    const MAGIC_OFFSET = 14;  // As long as the log4js library call stack doesn't change, this constant should work.
    var trace = structuredStackTrace[MAGIC_OFFSET];
   
    return {
      name: trace.getMethodName() || trace.getFunctionName() || "<anonymous>",
      file: trace.getFileName(),
      line: trace.getLineNumber(),
      column: trace.getColumnNumber()
    };
  }
```
On line 180, I've added the **formatMethod(loggingEvent, specifier)** and **formatLineNumber(loggingEvent, specifier)** methods.

```javascript
  function formatMethod(loggingEvent, specifier) {
    var trace = getTrace();
    return trace.name;
  }

  function formatLineNumber(loggingEvent, specifier) {
    var trace = getTrace();
    return trace.line;
  }
```

And lastly, on line 276, I've added the two (2) formatters.

```javascript
  /* eslint quote-props:0 */
  const replacers = {
    'M': formatMethod,      // @author Christian Jean
    'L': formatLineNumber,  // @author Christian Jean
    'c': categoryName,
    ...
```

That's pretty much it! Nothing more than that...

Although, you may have noticed that on line 162, there is something which you may have found suspicious?

```javascript
const MAGIC_OFFSET = 14;  // As long as the log4js library call stack doesn't change, this constant should work.
```

It is essentially an offset into the stack trace. When I first altered the `log4js-node` library (I think version 2.x.x), the `MAGIC_OFFSET` was 9 and it worked like that. For this version (3.0.5), the magic number is now 14. Although I haven't tested it thoroughly, as long as the code doesn't change that much, or more specifically, the call-stack doesn't change, this number should work perfectly. If I ever need to update the code with any fixes, this is easy to check and change to make it work again.

That's about it ... now I don't depend on another library on what I considered important. 

I also intend to add a few additional features in order to easily change the log level for any of my modules from a single config file and not within the modules as they recommend.
