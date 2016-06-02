- Feature Name: frontend-development-infrastructure-and-workflow-changes
- Start Date: 2016-07-20
- RFC PR: #10
- Issue:

# Summary
[summary]: ../0010-add-react-js-to-foreman-technology-stack.md#summary

Currently, foreman client side development is implemented in native javascript and the jquery library and managed by the Rails asset pipeline. In addition, various unrelated libraries are used for specific tasks, charting for example. This RFC proposes adding the React javascript library to the foreman technology stack.

# Motivation
[motivation]: ../0010-add-react-js-to-foreman-technology-stack.md#motivation

The current setup is based on software which was introduced many years ago, solved  problems that are no longer relevant and do not address the challenges of frontend development today. Many of the libraries we use are no longer maintained. The infrastructure is sparse and the responsibility for enforcing best practices and coding standards falls on the developer.

Towards the turn of the century and into the first decade of the 21th century numerous hardware, software and technological developments (broadband, XMLHTTP, HTML5, JS5, CSS3 among others) combined in a way that led to the web replacing desktop client/server technology as the primary enterprise application platform. In that context, web applications are expected to give the same features and rich user experience that desktop applications have provided in the past.

JQuery, released in 2006, provided a solution to browser compatibility issues, was an easy-to-use tool for DOM manipulation, and encouraged ecosystem development via plugins.

The applications developed were large, complex and difficult to manage. Better solutions were sought for building robust, maintainable and extensible applications.

Today, those browser compatibility issues which jQuery addressed are no longer relevant, direct DOM manipulation is discouraged and jQuery no longer dominates the ecosystem space.

The frontend ecosystem has expanded dramatically with the addition of many JavaScript libraries and frameworks, as well as developer productivity tools.

Reactjs is a view library that falls under the umbrella MV* libraries and frameworks. The common thread among these frameworks is the separation of the management of state (model) from the process of rendering HTML (view). The benefits are similar to those provided by server side MVC, such as Rails.

Applying an MV* approach would facilitate enforcement of best practices and coding standards and make the code produced more of robust, maintainable and extensible. It must be noted however that, any solution must 'work well' with the current foreman Ruby/Rails code base and plugins, as there is no intention to abandon it.

The tools in the MV* space are often classified as being opinionated or unopinionated with respect to the amount of flexibility they allow the developer. This often maps to the breadth of the issues the library or framework attempts to deal with.

Reactjs is a component-based rendering engine. As such, it is both narrow in scope and unopinionated. This will facilitate integration with Rails with minimum side effects on the existing Ruby/Rails code base compared to the opinionated alternatives.

React components demonstrate a high level of encapsulation and support dependency injection. As a result, the components are very testable. The implementation of the recommendations in this RFC will be accompanied by the addition of linting and testing to the client side development process.

# Detailed design
[design]: ../0010-add-react-js-to-foreman-technology-stack.md#detailed-design

Reactjs uses javascript to compose components and render HTML. Data is passed to the component via props, which are immutable. Component state is mutable and managed by the component. When state changes the component is rerendered.

Note: for those familiar with Angular 1, React components are similar to Angular directives.

The [Fixes #13424 - WIP c3 patternfly react implementation PR](https://github.com/theforeman/foreman/pull/3603) provides a POC for integrating Reactjs with foreman.

Top level components are added to an HTML page (mounted) with the `ReactDOM` package `render` method. This is the point at which the Rails view passes the baton to Reactjs.

The paradigm for using React components in Rails views implemented in the POC is as follows:

>The `mount_react_component` helper in the `ReactjsHelper` module is used in the Rails view to which the component is to be added. The helper has 3 paramaters: a string identifier for the specific component to be mounted, a jquery selector that is used to find the DOM element into which to inject the component and, optionally, initial data from the controller in JSON format. See *'app/views/statistics/index.html.erb'* in the POC for an example.

``` erb
    <div id="statistics"></div>

    <%= mount_react_component('StatisticsChartsList', '#statistics', @data)%>
```

> The helper injects inline javascript code into the Rails view which calls the `mount` method on the `reactMounter` object exposed on the global name space with the the parameters described above.

> The `reactMounter` object manages a collection of outwardly facing components. The `mount` method finds the markup for the component specified by the identifier, adds the initial data (if provided) as props, finds the DOM element specified by the jquery selector and calls the `ReactDOM render` method with the component markup and a reference to the DOM element as parameters.

> ReactDOM adds the component to the HTML page and Reactjs takes over from here.

With this paradigm a contributor working in Ruby/Rails could use a previously developed component without great difficulty.

# Drawbacks
[drawbacks]: ../0010-add-react-js-to-foreman-technology-stack.md#drawbacks

The proposed change requires new skills, the capacity to shift back and forth from the Rails and JavaScript mindsets and the integration of very different technologies.

# Alternatives
[alternatives]: ../0010-add-react-js-to-foreman-technology-stack.md#alternatives

The current frontend development paradigm is difficult to work with. Development is time consuming and results haven't been robust or easily testable. There are issues of global name space pollution and unintended side effects due to the lack of a module system. Substantial refactoring should be considered if remaining with the current code and development paradigm going forward.

Building a proprietary framework was not considered because of the abundance of available frameworks and libraries.

Opinionated frameworks such as Angular 1, Angular 2 and Ember were considered. As the author of this RFC has particular knowledge and experience with that framework, Angular 1 was used as a proxy for the opinionated frameworks. The opinionated frameworks provide more built in features including application state management, but at the price of flexibility. It was also thought that the more opinionated frameworks would be more difficult to integrate into existing html.erb pages.

Since Reactjs is less intrusive and does not require a full commitment, it is considered the best solution for our needs.

# Unresolved questions
[unresolved]: ../0010-add-react-js-to-foreman-technology-stack.md#unresolved-questions
