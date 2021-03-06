# Chrome Extension Message Wrapper

## What is the Chrome Extension Message Wrapper?

The Chrome Extension Message Wrapper is a package which should make developing Chrome Extensions easier, by providing a wrapper for the message passing api.

I often found myself using some really verbose switch statements on the background script, just to trigger simple functions. I also had a problem with the callback-based response style. Therefore, I wrote a simple wrapper which allows you to define an object of functions in the background script, which you can call with exactly the same arguments in the content / popup script and will return a promise with the result of the function.

You don't have to worry about calling `chrome.runtime.sendMessage` again soon.

I plan to extend this package in the future.

## How to install

```
  npm i chrome-extension-message-wrapper
```

or...

```
  yarn add chrome-extension-message-wrapper
```

## How to use

To make the wrapper work, you have to call two functions.

On the background script:

```javascript
import { setupMessageListener } from "chrome-extension-message-wrapper";

const someFunction = name => {
  return `Hey ${name}, whats up?`;
};

chrome.runtime.onMessage.addListener(
  setupMessageListener({
    someFunction
  })
);
```

On the content / popup script:

```javascript
import { initBGFunctions } from "chrome-extension-message-wrapper";

initBGFunctions(chrome).then(backgroundFunctions => {
  const { someFunction } = backgroundFunctions;

  someFunction("ContentScript").then(response => {
    console.log(response); // => "Hey ContentScript, whats up?"
  });
});
```

## Documentation

### setupMessageListener

To be called on the background script.

**Params:**

- `Object` backgroundFunctions: Functions which will be accessable from the background script. ( Can be a nested object )
- `Object` options: Configure the listener to do logging or use a custom message handler.

**Returns:**

- `Function`: Takes `request`, `sender`, `sendResponse` as params. Should be passend to chrome.runtime.onMessage.addListener

**Usage**

```javascript
import { setupMessageListener } from "chrome-extension-message-wrapper";
import axios from "axios";

const syncCall = name => {
  return `Hey ${name}, whats up?`;
};

const asyncCall = url => {
  return axios.get(url).then(res => res.data);
};

const nested = () => {
  return "Inside a nested object";
};

// Every function automaticaly gets passed the request ({ request, sender }) as the last argument.
const removeTab = request => {
  chrome.tabs.remove(request.sender.id);
};

const backgroundFunctions = {
  // Responds with return value
  syncCall,

  // Responds with the result of the axios call
  asyncCall,

  // Can be called on the content script with backgroundFunctions.nestedObject.nested()
  nestedObject: {
    nested
  }
};

const options = {
  // Enables logging when a message was called from the content / popup script
  verbose: true,

  // Enables custom log messages instead of default (`Got request to call a function: ${req}`)
  logRequest: req => `Custom logging of function request: ${req}`,

  // Function which gets called when you send a message using the chrome.runtime.sendMessage
  customHandler: (request, sender, sendResponse) => {
    //...code
    return "From Custom Handler";
  }
};

chrome.runtime.onMessage.addListener(
  setupMessageListener(backgroundFunctions, options)
);
```

### initBGFunctions

To be called on the content / popup script

**Params**

- `Object` chrome: Chrome instance

**Returns**

- `Promise` bgFunctions: Object of all background functions which were passed to setupMessageListener + default sendMessage().

**Usage**

```javascript
import { initBGFunctions } from "chrome-extension-message-wrapper";

// Using the functions from the setupMessageListener example
initBGFunctions(chrome).then(async bgFuncs => {
  const syncRes = await bgFuncs.syncCall("ContentScript");
  console.log(syncRes); // => "Hey ContentScript, whats up?"

  const asyncRes = await bgFuncs.asyncCall(
    "https://jsonplaceholder.typicode.com/posts/1"
  );
  console.log(asyncRes); // => {userId: ..., title: ... }

  const nestedRes = await bgFuncs.nestedObject.nested();
  console.log(nestedRes); // => "Inside a nested object"

  const res = await bgFuncs.send({ command: "Other", data: { foo: "bar" } });
  console.log(res); // => "From Custom Handler"
});
```

## Contribute

If you find a bug or have an idea for a new feature feel free to open a new issue or create a new pull request.

I will keep track of the todos on the issue page.

## License

MIT
