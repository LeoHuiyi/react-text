# react-text-translate

React text translation with dictionaries. An easy to use and flexible library to fullfill your translation needs:

```js
import Text from 'react-text-translate';
import dictionary from './dictionary';

export default () => (
  <Text language="ja" dictionary={dictionary}>
    <p><Text greetings /></p>
    <p><Text farewell name="Francisco" /></p>
  </Text>
);
// <p>こんにちは、世界！</p>
// <p>さよなら、FRANCISCOさん！</p>
```

Contents:

- [**Getting started**](#getting-started): Introduction to how to use `<Text>` with React.
- [**Dictionary**](#dictionary): define the translations and some basic transformations.
- [**Configuration**](#configuration): set the language and inject the dictionary.
- [**Translate**](#translate): use `<Text>` with a key to create translations with optional properties.
- [**Render**](#render): inject a translated string into a React component. Useful for `alt={}` and similar.
- [**Component**](#component): renders only for the right language.




## Getting started

First let's install the package with npm:

```bash
npm install react-text-translate
```

Then we define [a dictionary](#dictionary) of the text to translate:

```js
// ./dictionary.js
export default {
  greetings: {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ja: 'こんにちは、世界！'
  }
};
```

To use those, we need to nest `<Text>`. The first will be to [inject the dictionary](#configuration) and optionally the language, the second to create a translation:

```js
// ./Example.js
import Text from 'react-text-translate';
import dictionary from './dictionary';

export default () => (
  <Text language="es" dictionary={dictionary}>
    <p><Text greetings /></p>
  </Text>
);
// ~> ¡Hola mundo!
```






## Dictionary

The dictionary is defined as an object of objects. The first level (`greetings`) is what we call the `key`, the second is the language (`en`) and finally we have the values (`Hello world` and functions):

```js
// ./dictionary.js
export default {
  greetings: {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ja: 'こんにちは、世界！'
  },
  farewell: {
    en: ({ name = 'World' }) => `Hello ${name}!`,
    es: ({ name = 'Mundo'}) => `¡Adiós ${name}!`,
    ja: ({ name = '世界' }) => `さよなら、${name.toUpperCase()}さん！`
  }
};
```

All the languages must be the same in all the entries, otherwise it will throw an error. The order is important as well, since the first language will be considered the default one if it cannot be found otherwise.


We highly recommend that you normalize the dictionary on your side:

```js
import normalize from 'react-text-translate/normalize';

export default normalize({
  greetings: {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ja: 'こんにちは、世界！'
  }
});
```

If you normalize it, you have the advantage that you can use it anywhere as plain functions:

```js
import dictionary from './dictionary';

console.log(dictionary.greetings());             // Default language
console.log(dictionary.greetings('es'));         // Specify the language
console.log(dictionary.greetings('ja', props));  // Pass some properties
```






## Configuration

Once we have the dictionary, we have to determine how and where to inject it, as well as specifying the language. This will be done by creating a `<Text>` element **with children**:

```js
import Text from 'react-text-translate';
import dictionary from './dictionary';

export default () => (
  <Text language="en" dictionary={dictionary}>
    {/* Here the language will be English */}
  </Text>
);
```

They can be set at different levels, which is specially useful if you want to split the dictionary into different pages:

```js
import Text from 'react-text-translate';
import dictionaryA from './dictionaryA';
import dictionaryB from './dictionaryB';

export default () => (
  <Text language="en">
    <Text dictionary={dictionaryA}>
      {/* English for Dictionary A */}
    </Text>
    <div>
      <Text dictionary={dictionaryB}>
        {/* English for Dictionary B */}
      </Text>
    </div>
  </Text>
);
```

When nesting dictionaries **they will cascade** and the latter keys will override the previous keys.

```js
const dictA = {
  greetings: { en: 'Hello world' },
  farewell: { en: 'Goodbye world' }
};
const dictB = {
  greetings: { en: 'Hey world!' }
};

export default () => (
  <Text language="en" dictionary={dictA}>
    <p><Text greetings /></p>
    <Text dictionary={dictB}>
      <p><Text greetings /></p>
      <p><Text farewell /></p>
    </Text>
  </Text>
);
// <p>Hello world</p>
// <p>Hey world!</p>
// <p>Goodbye world</p>
```

The `language` key is optional and will default to the browser language, or to the first language in the dictionary otherwise. Normally the language would come from your own code:

```js
import Text from 'react-text-translate';
import dictionary from './dictionary';

export default ({ language = 'en' }) => (
  <Text language={language} dictionary={dictionary}>
    {/* Here the language will be English */}
  </Text>
);
```

The language can also be nested, and it will use the most specific (innermost):

```js
const dictionary = { greetings: {
  en: 'Hello world!',
  es: '¡Hola mundo!',
  ja: 'こんにちは、世界！'
}};

export default () => (
  <Text language="en" dictionary={dictionary}>
    <p><Text greetings /></p>
    <Text language="ja">
      <p><Text greetings /></p>
      <Text language="es">
        <p><Text greetings /></p>
      </Text>
    </Text>
  </Text>
);
// <p>Hello world!</p>
// <p>こんにちは、世界！</p>
// <p>¡Hola mundo!</p>
```

While nesting dictionaries is totally fine and expected, nesting languages might get messy and it's recommended to avoid it if possible. Use a global store like Redux to handle the language instead and inject it at the root level:

```js
// LanguagePicker.js
// Example implementation with Redux and an action creator
const setLanguage = payload => ({ type; 'SET_LANGUAGE', payload });
export default connect(({ language }) => ({ language }))(({ language, dispatch }) => {
  <Text language={language}>
    <div onClick={e => dispatch(setLanguage('en'))}>English</div>
    <div onClick={e => dispatch(setLanguage('es'))}>Spanish</div>
    <div onClick={e => dispatch(setLanguage('ja'))}>Japanese</div>
    Current language: {language}
  </Text>
});

// reducers/index.js
export default combineReducers({
  // ...
  language: (state = 'en', { type, payload }) => {
    return (type === 'SET_LANGUAGE') ? payload : state;
  }
});
```






## Translate

Now that we have our dictionary injected and language detected, we can use them with a **self-closing tag** and the right dictionary key:

```js
const dictionary = {
  greetings: {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ja: 'こんにちは、世界！'
  }
};

// Usage; the prop 'greetings' will correspond to the dictionary key 'greetings'
export default () => (
  <Text language="ja" dictionary={dictionary}>
    <p><Text greetings /></p>
  </Text>
);
// ~> <p>こんにちは、世界！</p>
```

**Valid key names**: any string except [`children`](#configuration), [`render`](#render) and [`component`](#component) since these have special meaning in React-Text-Translate. Click on those keywords to see how they are used. The keys are case-sensitive.

The dictionary can also be a function, which will be called when rendering. The advantage is that it will receive any prop that you pass to the element. You can then localize the text properly depending on the language, and even provide defaults easily:

```js
const dictionary = {
  greetings: {
    en: ({ name = 'World' }) => `Hello ${name}!`,
    es: ({ name = 'Mundo' }) => `¡Hola ${name}!`,
    ja: ({ name = '世界' }) => `こんにちは、${name.toUpperCase()}さん！`
  }
};

// The prop passed as `name` will be received in the dictionary
export default () => (
  <Text language="ja" dictionary={dictionary}>
    <p><Text greetings name="Francisco" /></p>
  </Text>
);
// ~> こんにちは、FRANCISCOさん！
```

> Note: the props that you can pass can be either strings or numbers, but right now you cannot pass a boolean like `<Text greetings isUser={true} />`. We might lift this limitation in the future.






## Render

Injects the plain text into a function. Useful for those times when you can only pass plain text and not a component:

```js
// These both render to the exact same thing:
<p><Text greetings /></p>
<Text greetings render={text => <p>{text}</p>} />
// ~> <p>Hello world</p>
```

However, the next example can only be achieved with `render()` since it will pass the plain representation as specified in the dictionary:

```js
<Text greetings render={text => <img alt={text} />} />
// ~> <img alt="Hello world">
```

If you try to do the same with `<Text />` you will get an unexpected result, since `<Text />` renders a React component:

```js
// ERROR - this does not work as expected
<img alt={<Text greetings />} />
// ~> <img alt="[Object object]">
```






## Component

When trying to do a switch between more complex fragments, or display one part only for one language, we can do so by using the `component` prop:

```js
<div>
  <Text en component={(
    <section>
      {/* Large block of text in English here */}
    </section>
  )} />
  <Text es component={(
    <section>
      {/* Large block of text in Spanish here */}
    </section>
  )} />
  <Text ja component={(
    <section>
      {/* Large block of text in Japanese here */}
    </section>
  )} />
</div>
```
