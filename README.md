[![npm version](https://badge.fury.io/js/ember-message-bus.svg)](https://badge.fury.io/js/ember-message-bus)
[![Build Status](https://travis-ci.org/patience-tema-baron/ember-message-bus.svg?branch=master)](https://travis-ci.org/patience-tema-baron/ember-message-bus)

# ember-message-bus

Explore new patterns of event-driven communication with `ember-message-bus`. This addon adds a simple service to your application, `message-bus`, which you can use to facilitate loosely coupled communications between services, components, and other Ember objects.

## Installation

`ember install ember-message-bus`

## Usage

First, inject the `message-bus` service into your object:

```js
import Ember from 'ember';

export default Ember.Component.extend({
  messageBus: Ember.inject.service('message-bus')
});
```

Once the `message-bus` has been injected, you can `subscribe` to events through it:

```js
import Ember from 'ember';

export default Ember.Component.extend({
  messageBus: Ember.inject.service('message-bus'),

  init() {
    this._super(...arguments);

    this.get('messageBus').subscribe('my-event', this, this.doSomething);
  },

  doSomething(arg1, arg2) {
    console.log(arg1 + arg2);
  }
});
```

`subscribe` expects three arguments: 1) the name of the event, 2) the context, typically `this`, and 3) the callback.

Finally, you can `publish` events to trigger the subscription:

```js
import Ember from 'ember';

export default Ember.Component.extend({
  messageBus: Ember.inject.service('message-bus'),

  action: {
    click() {
      this.get('messageBus').publish('my-event', 1, 2);
    }
  }
});
```

`publish` expects one or more arguments. The first argument must be the name of the event. After that, you can pass as many arguments as you like into `publish`. These arguments will be handed to the subscribing callback.

### Testing

It's easy to test if a message is published. First, run `initializeQUnitAssertions`:

```js
import { initializeQUnitAssertions } from 'ember-message-bus';

moduleForComponent('my-component', 'Integration | Component | my component', {
  integration: true,

  beforeEach() {
    const appInstance = getOwner(this);

    initializeQUnitAssertions(appInstance);
  }
});
```

If you'd like to ensure that events are published, use the Qunit assertion `willPublish` like so:

```js
assert.willPublish('shouldBeTriggered', '`shouldBeTriggered` was triggered');
assert.willPublish('shouldReceiveArgs', ['foo', arg2], '`shouldReceiveArgs` received the correct args');
assert.willPublish('shouldCallback', (param1, param2) => return param2 === arg2, '`shouldCallback` tested with callback');
```

Note that if you want to test that args are published, the expected args should be passed in as an array. Alternatively, you can provide a callback that will receive the args as their params.

On the other end, if you want to confirm that a message was not published, you can use `willNotPublish`:

```js
assert.willNotPublish('thisMessageShouldNotBePublished', '`thisMessageShouldNotBePublished` was not published');
```

Note that in both cases, these assertions should be made _before_ the message is actually published. So in a component integration test:

```js
const foo = { bar: 'baz' };

this.set('foo', foo);

assert.willPublish('shouldBeTriggered', [foo], '`shouldBeTriggered` was triggered');

this.render(hbs`{{my-component foo=foo}}`);
```
