# Propob

Propob stands for Protractor Page Objects and is a Page Object Model [DSL][] for Protractor.

The *Model DSL* part is about providing a declarative and [DRY][] way to write your [Page Objects][PageObject]

## Goal

The ultimate goal of this project is to provide for [Protractor][] what [site_prism][] does for [Capybara][] which is:

> A simple, clean and semantic DSL for describing your site using the Page Object Model pattern.

One of the objectives is to stay [DRY][] and for that we needed some of [site_prism][] features:

## Features in site_prism to be mirrored

### Page and Iframes support

* a [Page class][site_prism_page] from which to inherit and provides Page Objects functionality out of the box plus is extensible. ![done][done-icon]
* has [iframe][site_prism_iframes] support so once you grab an iframe element the interaction should be [transparent][site_prism_interact_iframe] so you can manipulate the frame as just another [Page][site_prism_page] instance. ![pending][pending-icon]

### Routing support

* has [base URL][site_prism_set_url] relative to a common configurable base path. ![done][done-icon]
* has [parameterized URLs][site_prism_parameterized_url] support.
* ability to [navigating to the page][site_prism_navigating_to_page] programmatically if needed. ![done][done-icon]
* can assert if the current page is [the one expected][site_prism_page_to_be_displayed]. ![done][done-icon]
* can assert if current page [query params are correct][site_prism_params_assert]. ![pending][pending-icon]

Note similar effort has been done in Java through [LoadableComponent][LoadableComponent].

### Sections support

Pages are logically or visually divided in sections or "modules" given that not every single time they deserve a page object on their own.

* has [sections][site_prism_sections] support that can be reused if needed. ![done][done-icon]
* section has a base element to reflect the fact that the [DOM][] is a [tree structure][]. To implement this feature in propob we take advantage on the fact that [JS Object literals][js-object] are per-se a [tree structure][] too. ![done][done-icon]
* wait for a [section to be present][site_prism_section_present] or [become visible or invisible][site_prism_section_becomes_visible_or_opp]. ![done][done-icon]
* support for [sections within sections][site_prism_setions_within_sections]. ![done][done-icon]
* can have a [collection of sections][site_prism_sections_collections]. ![pending][pending-icon]

### Wait for things instead of failing fast

* wait for element to [become visible][site_prism_become_visible]. ![done][done-icon]
* wait for element to [become invisible][site_prism_become_invisible]. ![done][done-icon]
* assert all elements of a section or page [are there][site_prism_all_there] instead of having to manually iterate each. ![pending][pending-icon]
* support [excluding][site_prism_all_there_exclude] certain elements from the [all are there][site_prism_all_there] feature. ![pending][pending-icon]

## Features in propob non existing in site_prism

* Code less with [Protractor][] locator shortcuts like `$linkText('Sign in')` and so. ![done][done-icon]

## How to organize your test suite

### Global Namespace

You can either `require` your dependencies and instantiate the pages you need every time or you can write less, keep reading if so.

First of all you may realize that it actually makes little sense having the ability to instantiate page objects since most pages behave as singletons, e.g. you have just one Home page on your website so making `new HomePage()` every time you need to interact with that page is redundant. Even though the Home page content may defer depending on your app state but when that's the case you are better off creating separate Page classes, e.g. `AdminHomePage`, `GuestHomePage` and so on.

That being said, instead of instantiate the Page class every time you need it is better to hold this singletons in some global namespace `Pages` or `App`, in Ruby this would be:

```ruby
class AdminHome < SitePrism::Page; end
class GuestHome < SitePrism::Page; end

module Pages
  def self.admin_home
    @admin_home ||= AdminHome.new
  end

  def self.guest_home
    @guest_home ||= GuestHome.new
  end
end

# Usage
expect(Pages.admin_home).to be_displayed
```

Site prism proposes [almost the same approach][site_prism_epilogue].

In Javascript through propob it would be:

```javascript
var propob = require('propob');

var AdminHome = new propob.Page({});
var GuestHome = new propob.Page({});

global.Pages = {
  adminHome: new AdminHome(),
  guestHome: new GuestHome(),
};

// Usage - verbose
it('should load the admin home page url', function() {
  expect(Pages.adminHome).toBeDisplayed();
});

// Usage - dry
describe('tests need to be grouped in at least 1 describe block', function() {
  Pages.adminHome.itShouldBeDisplayed();
});
```

### Test Suite Architecture

Either if you use Cucumber or Jasmine you should write your plan your test suite from the highest domain level from the user of the product perspective because in the end you'll be creating an [Internal DSL][] on top of your product domain knowledge.

### Workflows

High level tests are the workflows that will use the lower layer of page objects infrastructure so to keep abstractions properly this layer should know nothing about the internal page elements or for instance selenium API details.

Diagram

| Test Workflows > Page Objects > Page Elements > Selenium API

### Models

It is convenient to have class models for the entities your web product manages, this is particularly useful for passing around test data via params.

For instance you may want to have a `User` class to represent the test users on your systems that may look something link this

```typescript
// Typescript 1.4
class User {
  firstName: string;
  lastName:  string;
  
  constructor(firstName: string, lastName: string) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

var user = new User('Leo', 'Gallucci');
```

Of course you could manage to have your test data in [plain old JS objects][pojo] but eventually you will need to compute values or add some logic that naturally belongs to the model like `user.fullName()` in the above example so is much better if you plan it upfront.

### Test Data

Use test data generators as much as possible including build system values like build number, browser short name & version and some kind of auto-increment because date based strings tend to be long. If the tests are not running in CI use `process.pid` instead of the absent build number.

Store the generated test data along with the testing artifacts like JSON or HTML reporting output.

For i18n test suites, store or generate your tests data in a localized independent primitive and convert the values like currency & dates with some localization library then, again, store the generated test data in some JSON file so can be tracked down later.

#### Seed you database with partitioned test data
TODO: pending

#### Mocking your back-end for UI tests
TODO: pending

#### Backup your local database to troubleshoot broken tests
TODO: pending

### Directory organization

    flows/     - High level reusable multi-page workflows.
    models/    - Test data models go here
    pages/     - Your page objects go here.
    testdata/  - JSON or CSV tests data to be loaded by the models.
    specs/     - Your high level tests will go here.
    common/    - Reusable support utility functions can go here.
    support/   - Reusable support utility functions can also go here.

## Requirements
- Protractor >= 2.0.0
- Jasmine 1.3.1 (current default Protractor testing framework). Jasmine 2 support is pending.
- AngularJS >= 1.2.7
- NodeJS >= 0.10.33 & < 0.11
- npm >= 2.6.1

## Alternatives to this framework

There's been [astrolabe][] for some time now whose definition is:

> Astrolabe is an extension for protractor that adds page objects to your functional/e2e tests.

But even with [astrolabe][] out there it didn't quite fit our team testing purposes.

## License

The project is made possible by volunteer contributors who put their time and made the source code freely available under the [Apache 2.0 license][license].


[Protractor]: https://github.com/angular/protractor
[site_prism]: https://github.com/natritmeyer/site_prism
[Capybara]: https://github.com/jnicklas/capybara
[astrolabe]: https://github.com/stuplum/astrolabe
[DRY]: http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[license]: LICENSE.md
[site_prism_page]: https://github.com/natritmeyer/site_prism#creating-a-page-model
[site_prism_set_url]: https://github.com/natritmeyer/site_prism#adding-a-url
[site_prism_parameterized_url]: https://github.com/natritmeyer/site_prism#parameterized-urls
[site_prism_navigating_to_page]: https://github.com/natritmeyer/site_prism#navigating-to-the-page
[site_prism_all_there]: https://github.com/natritmeyer/site_prism#checking-that-all-mapped-elements-are-present-on-the-page
[site_prism_become_invisible]: https://github.com/natritmeyer/site_prism#waiting-for-an-element-to-become-invisible
[site_prism_become_visible]: https://github.com/natritmeyer/site_prism#waiting-for-an-element-to-become-visible
[site_prism_params_assert]: https://github.com/natritmeyer/site_prism#specifying-parameter-values-for-templated-urls
[site_prism_page_to_be_displayed]: https://github.com/natritmeyer/site_prism#verifying-that-a-particular-page-is-displayed
[site_prism_epilogue]: https://github.com/natritmeyer/site_prism#epilogue
[site_prism_interact_iframe]: https://github.com/natritmeyer/site_prism#interacting-with-an-iframes-contents
[site_prism_iframes]: https://github.com/natritmeyer/site_prism#iframes
[site_prism_sections_collections]: https://github.com/natritmeyer/site_prism#section-collections
[site_prism_setions_within_sections]: https://github.com/natritmeyer/site_prism#sections-within-sections
[site_prism_section_becomes_visible_or_opp]: https://github.com/natritmeyer/site_prism#waiting-for-a-section-to-become-visible-or-invisible
[DOM]: http://en.wikipedia.org/wiki/Document_Object_Model
[site_prism_section_present]: https://github.com/natritmeyer/site_prism#waiting-for-a-section-to-exist
[site_prism_sections]: https://github.com/natritmeyer/site_prism#sections
[site_prism_all_there_exclude]: https://github.com/natritmeyer/site_prism/pull/98
[PageObject]: https://code.google.com/p/selenium/wiki/PageObjects
[LoadableComponent]: https://code.google.com/p/selenium/wiki/LoadableComponent
[tree structure]: http://en.wikipedia.org/wiki/Tree_structure
[pojo]: http://en.wikipedia.org/wiki/Plain_Old_Java_Object#cite_note-2
[DSL]: http://en.wikipedia.org/wiki/Domain-specific_language
[Internal DSL]: http://martinfowler.com/books/dsl.html

[icons-source-page]: http://www.softicons.com/
[done-icon]: images/done-icon-16x16.png "Feature done in propob"
[pending-icon]: images/pending-icon-16x16.png "Feature pending"
[wont-do-icon]: images/wont-do-icon-16x16.png "Won't do or not planned"
