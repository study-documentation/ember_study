# Ember.js

An angular-flavored single page app framework. What better way to start learning than to write oneself a textbook.

## Prerequisites

In order to build an Ember project the following items must be on your machine
Required:

- Node.js
- npm
- ember cli (`npm install -g ember-cli`)

Optional:

- watchman (`brew install watchman` - do not use the watchman package available via NPM)
  - or - (`arch -arm64 brew install watchman` for arm based Macs)

## CLI

`ember serve` === `npm start`
`ember g c <name of component>` include `-gc` to generate a js file for the component.
`ember g route <name of route>`
`ember g util <name of utility>`
`ember g helper <name of helper>`

# Working With Ember Ocatane

Do stuff fast and so forth.
Ember does not release new features at major releases.

.hbs <-- Handlebars, a markup superset

Contents:

- Components and Helpers
- Basic Routing
- Component State and Actions
- Application State
- Nested Routes and Async Data
- Component Architecture
- Performance

## Components & Helpers

The contents of components in Ember is set in .hbs files. It is convention to do this in the templates folder where app.hbs in the entry point and a subdirectory called components is created to house all remaining component files. Granular components create semantic app.hbs files.

### Arguments - Parameterizing Components

An `@` inside a template is a sign that an immutable piece of data is being passed into the template from the outside.
The `{{}}` notation is a call to a piece of information or function. When calling a fuction in a template the syntax will look like this `{{<name of function> <parameter>}}`. Note that there is no delimiter.
The code in the component .hbs will look something like this...

```
<h3 class="text-grey-darkest mb-1 font-extrabold channel-header__title">
        <span aria-hidden="true">#</span>
        {{ @title }}
      </h3>
```

Back in the app.hbs, the corrsponding coding will look like this...

```
<ChannelHeader
    @title="Frontend Chat"
    @description="CSS and JS chat"/>
```

These statements prepended by an `@` are refered to as arguments.

### Utilities and Helpers

Plain JavaScript functions wrapped in Ember are ideal for unit testing and reusibility. Functions used broadly across an application are housed in the `app` subdirectory called `util`. These `utils` are intended to have broad application. Helper functions contained with in `app > helpers` can take outputs from a utility and transform it for specific scenarios.

For example, a utility might convert a date to a string and a helper might take that string and change the format or extract certain pieces of information given the context.

Ane xample of a helper function is...

```
import { helper } from '@ember/component/helper';
import { dateToString } from 'shlack/utils/date';

export default helper(function formatTimestamp(params/*, hash*/) {
  const [ dateIsh ] = params;
  return dateToString(dateIsh);
});
```

In the params of a generated helper there is (param, hash) hash is for key/value pair objects.

### Unit Testing

Anytime utility(unit) or helper(integration) is generated Ember produces a unit test as well. Unit tests are accessed in the browser at an application endpoint. During local development tests might be accessed at `localhost:4200/tests`

A qunit test looks something like this...

```
import { dateToString } from 'shlack/utils/date';
import { module, test } from 'qunit';

module('Unit | Utility | date', function() {
  // Replace this with your real tests.
  test('string inputs', function(assert) {
    assert.equal(
      dateToString('04/05/1983'),
      'Apr 5, 1983 00:00.00 AM',
      'MM/DD/YYYY'
    );
    assert.equal(
      dateToString('4/5/1983'),
      'Apr 5, 1983 00:00.00 AM',
      'M/D/YYYY'
    );
    assert.equal(
      dateToString('26 June 2010 13:14'),
      'Jun 26, 2010 01:14.00 PM',
      '26 June 2010 13:14'
    );
  });

  test('empty and invalid inputs', function(assert) {
    // @ts-ignore
    assert.equal(dateToString(), null);
    // @ts-ignore
    assert.equal(dateToString(null), null);
    // @ts-ignore
    assert.equal(dateToString([]), null);
    // @ts-ignore
    assert.equal(dateToString({}), null);
  });
});
```

### Integration Testing

Note that in integration tests in Ember the syntax used in the `await` statement is precisely the same as in an .hbs file. Intgration tests are going to use realistic data to observe potential interactions that will take place in a deployed app.

```
import { module, test } from 'qunit';
import { setupRenderingTest } from 'ember-qunit';
import { render } from '@ember/test-helpers';
import hbs from 'htmlbars-inline-precompile';

module('Integration | Helper | format-timestamp', function(hooks) {
  setupRenderingTest(hooks);

  // Replace this with your real tests.
  test('it renders', async function(assert) {
    this.set('myDate', '05-18-1995');

    await render(hbs`{{format-timestamp myDate}}`);

    assert.equal(this.element.textContent.trim(), 'May 18, 1995 00:00.00 AM');
  });
});
```

## Basic Routing

Creating URL specific content in an app.

### Setting Up New Routes

After running `ember g route <name of route>` in the terminal, unit tests will appear `tests > unit > routes`, . hbs templates for those components will appear in the templates folder and .js files will appear in `app > routes`.

As usual the .hbs file is the locale of the base content. Subcomponents are housed in `templates > components`.

Once routes are constructed the concept of `{{outlet}}` becomes important. Outlets are dynamic calls to other components that will be displayed within a component at a lower level. It is one of the ways that ember achieves the nested hierarchy typical of template drive SPA frames works.

### Linking Routes with linkTo

Hey, guess what? You dont actually link to new pages in a SPA. You simply create the illustion of page loads with loading bars and spinny bits. To that end, the `<a></a>` of static HTML is not a useful tool.

`<LinkTo>` replaces `<button>` in ember. It requires a few flags to work properly. It must be provided a route, model, models or query such that `<LinkTo @<name of provided component>="<name of component to route to>"`.<br>
LinkTo will default as an `<a>` this can be customized by adding another flag. `<LinkTo @<name of provided component>="<name of component to route to>" @tagName="<type of tag>"`

In practice this might look like... <br>
`<LinkTo @route="login" @tagName="button"`

### Acceptance Testing

Acceptence tests are designed to mimic user input.<br>

`ember g acceptance-test <name of test>`

If this is the first acceptance test being created, a new subdirectory called `acceptance` will appear in the `tests` directory. When generating the test ember assumes that the name given to the test is the name of a URL that should be tested. This is done via the ember test helpers library which holds numerous methods that allow for asynchronous testing. Ember will pause tests while data is fetched or features load, etc

Here is a test that achieves the following ends...

- visit teams page
- assert that URL has been reached
- click on logout
- assert that reroute to login happened

```
import { module, test } from 'qunit';
import { visit, currentURL, click } from '@ember/test-helpers';
import { setupApplicationTest } from 'ember-qunit';

module('Acceptance | logging out', function(hooks) {
  setupApplicationTest(hooks);

  test('visiting teams and clicking logout', async function(assert) {
    await visit('/teams'); //goto the /teams URl

    assert.equal(currentURL(), '/teams');

    await click('.team-sidebar__logout-button');

    assert.equal(currentURL(), '/login');

  });
});
```

`visit` and `click` are built-in to the ember `test-helpers` library. Note the structure of these tests. First, a method is called that will perform some task (i.e. click on a button), then an assertion is made that assumes the resultant behavior will match the programmers expectations. This is similar to unit testing. However, where acceptance tests differ from unit tests is that the acceptance test above is not testing one isolated action. Acceptance test performs a series of actions in an order that is likely going to be a part of the user experience.

Note also the argument in the click method. Here one must enter a unique identifier to for the button to be clicked so that ember clicks that correct button on the page. This identifying information is found in the HTML.... <br>

```
<button id="ember123" class="text-white rounded bg-grey-dark hover:bg
-red-darker p-2 team-sidebar__logout-button">Logout</button>
```

This is how the `LinkTo` is rendered in the browser. Much of the contents in the tag are boilerplate and, therefore, not valuable for testing. However, at the end of the class statement is `team-sidebar__logout-button`. This is an ideal candidate for a testing identifier. It must be entered in the test as `.team-sidebar__logout-button`. The `.` denoting that it is nested in other information. Another important note is that `id="ember123"` seems like the obvious choice for test id. Well, its not. Doesn't work. No idea why, just doesn't.

These acceptance tests are written as an await function. The coupled with inserting a `debugger;` or `pauseTest();` between the awaits or between them and the assserts will enable the pinpointing of errors. `pauseTest();` is best written as `this.pauseTest();` so there is not an import hanging out. The benefit of `pauseTest` is that the app is interactive. The cost of `pauseTest` is that you cannot see a stack frame.

## Component State & Actions

### Event Handling with the `on` Modifier

Handle all actions in components whenever possible. Complexity should exist in the component layer. 
`ember g component login-form` <-- this generates a handlebars template file, an integration test and a component js file.
{{@title}} <-- helper (comes in from the outside)
{{on}} <-- modifier (comes from the component)

An example of a modifier in action is...   
```
<form 
      {{on "submit" this.onLoginFormSubmit}}
      class="bg-grey-light shadow-md rounded px-8 pt-6 pb-8 mb-4">
```
Note that the function call begins with this. Anytime the contents of a template begin with this, it is a clear signal that it is referring to something that is defined with the Javascript file rendered for the component.

### Decorators and Action Decorator

Decorators are read bottom to top when stacked.

Decorators are recognized as class-build time and establish a relationship between the object and propName. `@action` binds a method to a component instance. This reduces Javascripts `this` weirdness. Java feels. Spring Boot feels. All is right in the world. `this` will always be the component when using `@action`


### Integration Testing
Integration tests determine if the contents of the html match expectations.
Here is a handy template for writing integration tests...   
```
assert.deepEqual(
      this.element.textContent
        .trim()
        .replace(/\s*\n+\s*/g, '\n')
        .split('\n'),
        [
          "Login",
          "Select a user",
          "Testy Testerson",
          "Sample McData",
          "A validation message"
        ]
    );
```
`deepEqual` is used to make a more thorough check; it reads contents. `equal` checks to the same extent as `===`.   
`trim` will remove white space that could lead to false failures.
`.replace(/\s*\n+\s*/g, '\n')` <-- this bit of regex will remove tabs and extra lines and turn them into a singular new line. With this bit implemented an edit later that adds a bunch of new lines or tabs to the content will not break a valid test.
`split('\n')` <-- takes the content that was just seperated into new lines and creates an array split upon those lines. It is against this array that the test will run.
Integration tests do not include dynamic elements such as `{{this.userId}}`.
Finally, The expected array in entered as the second argument in the assertion.

### Stateful Components

Adding state to a component requires edits to both the `.hbs` template and `.js` object. In the object it will be as simple as...
`userId = null;`<br>
<br>
In the `.hbs` file...  
```
<select
            class="block appearance-none w-full bg-white border border-grey-light hover:border-grey px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline">
            <option selected={{ not this.userId }} value="">Select a user</option>
            <option selected={{ eq this.userId "1" }} value="1">Testy Testerson</option>
            <option selected={{ eq this.userId "2" }} value="2">Sample McData</option>
          </select>
```
The `userId` is initialized to `null`. That is changed by the user via the `.hbs`. `selected` is a standard HTML jobby.
{{not}} is ember stuff that is equivalent to `!this.userID` in JS. `value=""` is the expected value from the given line of HTML.
{{eq}} means equal. Again there are no JS `this` shenanigans becuase the `@action` decorator tethers a given object and method at build time. Also note again that a space is used to delimit between eq, the method call and the argument.

### Using Track Properties

Ember Octane does not rely on two-way binding. This is done for the sake of creating more performant apps.

Ember Octane's reliance on one-way binding means one must opt-in to tracking infomation. This is done via the `@tracked` annotation. This is imported from `import { tracked } from '@glimmer/tracking';`. This only applies to properties of templates.

Seen in a code snippet...   
```
export default class LoginFormComponent extends Component {
  @tracked
  userId = null;
```
Everything else is the same as the code in the last section.

- Listen for a dom event
- When that dom event is fired we use vanilla javascript give some property a new value.

#### Enabling and Disabling 

Observing null state can be done with vanilla JS getters.
```
get isDisabled(){
    return !this.userId;
  }
```
This returns true is userId is false.
This does not need to be tracked.

### `if` Helper

Up to this point the validation message below the user selection drop down was visible even when a user was not selected. Thats dumb. It should be displayed given a condition. This is done via the `if` helper.
```
<p class="text-blue text-xs italic my-4">
          {{#if (not this.isDisabled)}}
            Logging in with User ID: {{this.userId}}
          {{/if}}
        </p>
```  
With these lines of code the user id confirmation line will only appear if `userId` is not `null`.

The `if` helper can also be used inline...
```
<input class="{{if this.isDisabled "bg-grey" "bg-teal"}} text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
            disabled={{this.isDisabled}}
            value="Sign In" type="submit" />
```

