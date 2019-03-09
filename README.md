# jeach-log4node

This is a fork of the [log4js-node](https://github.com/log4js-node/log4js-node) project.

I'm not trying to compete with the original library or anything like that. I just thought it was the closest thing to Log4J which I am accustomed to. 

The reason I forked it is because I believe that **method** and **line number** information in your logs are crucial. I can't figure out why they have yet to provide support out-of-the-box for it? But they don't. So I decided to simply add it myself and make it public for anyone that requires the same functionality.

So now, if you want to use the following pattern:

```
  "layout": {
    "type": "pattern",
    "pattern": "[%d][%-2p][%c][%M(%L)]> %m"
  }
```

Notice the `%M` and `%L`? This will provide logs similar to:

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
    // As long as the log4js library call stack doesn't change, this constant should work.
    const MAGIC_OFFSET = 14;
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

## Logging Levels

```
  ALL
  TRACE
  DEBUG
  INFO
  WARN
  ERROR
  FATAL
  MARK
  OFF
```    

## Pattern format


The pattern string can contain any characters, but sequences beginning with % will be replaced with values taken from the log event, and other environmental values. Format for specifiers is %[padding].[truncation][field]{[format]} - padding and truncation are optional, and format only applies to a few tokens (notably, date). e.g. %5.10p - left pad the log level by 5 characters, up to a max of 10
    
### Examples:
    
```    
  [%p]     --> [DEBUG]
  [%3.3p]  --> [DEB]
  [%1.1p]  --> [D]
```

### Fields can be any of:

```
  %M method name
  %L line number
  %r time in toLocaleTimeString format
  %p log level
  %c log category
  %h hostname
  %m log data
  %d date, formatted - default is ISO8601, format
  %% % - for when you want a literal % in your output
  %n newline
  %z process id (from process.pid)
  %x{<tokenname>} add dynamic tokens to your log. Tokens are specified in the tokens parameter.
  %X{<tokenname>} add values from the Logger context. Tokens are keys into the context values.
  %[ start a coloured block (colour will be taken from the log level, similar to colouredLayout)
  %] end a coloured block
```  
  
**Additional date (%d) options are:**
  
```
  ISO8601, ISO8601_WITH_TZ_OFFSET, ABSOLUTE, DATE, or any string compatible 
  with the date-format library. e.g. %d{DATE}, %d{yyyy/MM/dd-hh.mm.ss}
```

## Updates

As of version 2.0.1, you can now use the follwoing functions in order to determine your log level:

```
const log = package.getLogger();

log.level = 'debug';

console.log("Level    : " + log.level);

console.log("isAll    : " + log.isAll());
console.log("isTrace  : " + log.isTrace());
console.log("isDebug  : " + log.isDebug());
console.log("isInfo   : " + log.isInfo());
console.log("isWarn   : " + log.isWarn());
console.log("isError  : " + log.isError());
console.log("isFatal  : " + log.isFatal());
console.log("isMark   : " + log.isMark());
```

The above would return:

```
Level    : DEBUG
isAll    : false
isTrace  : false
isDebug  : true
isInfo   : false
isWarn   : false
isError  : false
isFatal  : false
isMark   : false
```
