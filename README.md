[![npm version](https://badge.fury.io/js/ember-unused-components.svg)](https://badge.fury.io/js/ember-unused-components)
![ember-cli version](https://img.shields.io/badge/ember--cli-3.1.4-orange.svg)

ember-unused-components
==============================================================================

This addon searches for unused components in your Ember project. It has POD structure and whitelist support.

Installation
------------------------------------------------------------------------------

```
ember install ember-unused-components
```


Usage
------------------------------------------------------------------------------

Run in your app root directory:

```
ember unused:components
```

Expected output:

```
Searching for unused components:
 No. of components: 277
 No. of unused components: 2

 Unused components:
  - app/app-tab-panel
  - user/user-birthday
```

If you feel like there are too many components listed then check [Configuration Section](#configuration).

### Advanced usage

Typically the addon should realize if you are using [POD structure](https://ember-cli.com/user-guide/#pod-structure) or not and find its way to components directory.

If you have problems with that, consider:

#### Forcing POD

To force using POD use `--pods` argument (alias: `-p`). Like this:

```
ember unused:components --pods
```

Addon will use default directory of POD components: `app/modules/components`

#### Forcing POD with the custom directory

Your app should be configured to have `podModulePrefix` property in `config/environment.js` if you are using POD but if it somehow doesn't work you can specify it through `--pods-dir` (alias: `-pd`). Like this:

```
ember unused:components --pods --pods-dir="modules/components-repository"
```

Configuration
------------------------------------------------------------------------------

In typical use cases, it should work out of the box and you don't have to configure anything but you can consider following options:

### Whitelist

You can specify which components should not be treated as unused even if the addon couldn't find their usage occurrences. This happens when:
 - you know that the component will be used in the future and you don't want to remove it and being reminded of that
 - you use some kind of "dynamic name resolution" for your components
 
Add whitelist to your `config/environment.js` file:

```js
ENV['ember-unused-components'] = {
  whitelist: [
    'app/app-tab-panel' // we will use it again soon
  ]
};
```

------------------------------------------------------------------------------

For the "dynamic name resolution" scenario consider following example.
Typical dynamic component look like this:

**Template**
```hbs
{{component name car=car}}
```

**JavaScript**
```js
name: computed('car.type', function () {
  return `car-card-${this.get('car.type')}`;
});
```

**Result**

Which may result in having following components in use:
- car.type = 'suv' => `{{car-card-suv car=car}}`
- car.type = 'sport' => `{{car-card-sport car=car}}`
- car.type = 'sedan' =>  `{{car-card-sedan car=car}}`

Unfortunately, this static analysis tool doesn't understand it yet and doesn't know that your component `car-card-suv`
has been used anywhere.
You can whitelist these components from being marked as unused by referencing to them directly:
```js
ENV['ember-unused-components'] = {
  whitelist: [
    'car-card-suv',
    'car-card-sport',
    'car-card-sedan'
  ]
};
```
or by using wildcard:
```
ENV['ember-unused-components'] = {
  whitelist: ['car-card-*']
};
```

### Ignored files

A component might be used in some files like guidelines template (see [ember-freestyle](https://github.com/chrislopresto/ember-freestyle)) that in fact does not indicate that it is in use. Best practice is to ignore that file:

```js
ENV['ember-unused-components'] = {
  ignore: [
    'app/templates/freestyle.hbs' // this is our template with style guides
  ]
};
```

Removing components
------------------------------------------------------------------------------

Please raise an issue if you would like to see that functionality. It might be useful. But please consider that simple removal of:
 - `template.hbs`
 - `component.js`
 - `style.sass` (if you use `ember-component-css`)

might not remove everything you would like. Examples:

 - global CSS classes that are no longer used
 - 3rd party JS libraries used only in that component
 - translation keys that are no longer needed
 - assets (images) that are no longer used
 - local overwrites/adjustments for that component made by parent's CSS

So you'll still have some dead code because of unused components.

I encourage you to remove components from the list manually.

Best Practices
------------------------------------------------------------------------------

Once you delete unused components run the script once again :) You might find that now some other components are no longer used.
This happens when the component is used in the unused component.

Example:


```hbs
{{!-- users-list component --}}
{{#each users as |user|}}
  {{user-card user=user}}
{{/each}}
```

So `user-card` is being used but in *unused* component `users-list`. Once you will delete `users-list` component then `user-card`
will not be longer used.

Roadmap
------------------------------------------------------------------------------

Future Plans and Ideas for the lib:
 - address dependent "unused" component problem (check [Best Practices](#best-practices))
 - removal done by script
 - removal of component with all its dependencies 
 
If you feel like you need this functionality please raise an issue or event better [Contribute](#contributing)!

Contributing
------------------------------------------------------------------------------

### Installation

* `git clone <repository-url>`
* `cd ember-unused-components`
* `npm install`

### Linting

* `npm run lint:js`
* `npm run lint:js -- --fix`

### Running tests

* `ember test` – Runs the test suite on the current Ember version
* `ember test --server` – Runs the test suite in "watch mode"
* `ember try:each` – Runs the test suite against multiple Ember versions

### Running the dummy application

* `ember serve`
* Visit the dummy application at [http://localhost:4200](http://localhost:4200).

For more information on using ember-cli, visit [https://ember-cli.com/](https://ember-cli.com/).

License
------------------------------------------------------------------------------

This project is licensed under the [MIT License](LICENSE.md).
