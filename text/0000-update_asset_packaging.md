- Feature Name: update_asset_packaging
- Start Date: 2016-05-10
- RFC PR:
- Issue:

# Summary
[summary]: #summary

Several suggestions have been made to improve the way we handle our assets packaging.
This RFC attempts to present the various options so that a decision can be made.

# Motivation
[motivation]: #motivation

Currently, most assets (js/css) are used as gems and managed by bundler.
A few are included directly in the vendor/ directory due to lack of gems.
Some of the reasons that we may want to change the way we currently handle this:
* Some assets are not packaged as gems, so we need to either vendor or package them ourselves.
* Some asset gems are not updated at the same rate as the upstream source.
* Each gem dependency must be packaged separately as RPM/DEB.
* Difficult to integrate newer JS technology such as es2015 or js modules.
* Assets included in the vendor/ directory need manual updating when new versions are released.


# Detailed design
[design]: #detailed-design

This will depend on the alternative that will be chosen and updated later.

# Drawbacks
[drawbacks]: #drawbacks

Some reasons to stay with the current solution:
* No extra work needed.
* Single package management system - bundler. All dependencies are in one central location, build process is simple.
* Rails asset pipeline doesn’t require any modifications to handle the current solution.

# Alternatives
[alternatives]: #alternatives

__Option 1: Stay with current solution__

See above for reasons for and against.

__Option 2: Webpack + npm__

[POC PR](https://github.com/theforeman/foreman/pull/3433)

_Pros:_
* Live reload development server.
* Ability to package the code in separate chunks that are only loaded when needed.
* Multiple loaders allow for server-side tasks such as transpiling, linting, minifing etc. to be run without requiring an additional task runner.
* Can possibly replace rails asset pipeline in the future if desired.
* npm uses CommonJS modules which is allows namespacing JS code, which means you just won't accidentally mess with your dependencies.

_Cons:_
* Complex configuration compared to other alternatives.
* Lack of examples in other projects

__Option 3: Browserify + npm__

[POC PR](https://github.com/theforeman/foreman/pull/3326)

_Pros:_
* Simple installation and configuration.
* Easy to tie into rails asset pipeline.
* Easy to add es2015, minification, etc.
* npm uses CommonJS modules which is allows namespacing JS code, which means you just won't accidentally mess with your dependencies.

_Cons:_
* Only handles JS assets.

__Option 4: Bower__

[POC is part of PR](https://github.com/theforeman/foreman/pull/2882)

_Pros:_
* Bower seems to have a larger ecosystem for front-end assets then npm
* Already in use in Katello, amount of effort required is halved.
* Live reload development server could be done with some more work to grunt/gulp or any other build system
* Bower uses flat dependency tree to reduce dependency duplicities that clients would have to download if plain npm + browserify was used.

_Cons:_
* Adds two more package managers instead of one - requires npm to install bower. This means we will have 3 different places that contain dependencies.
* Some packages now are packaged only in npm
* Does not provide solution for any server-side JS (testing, transpiling, etc.). Any such dependencies will need to be handled either by npm or bundler.

__Option 5: npm alone__

No POC available.

_Pros:_
* Only one new tool to add to the project.
* npm uses CommonJS modules which is allows namespacing JS code, which means you just won't accidentally mess with your dependencies.

_Cons:_
* Requires means of adding the npm modules to the assets pipeline, such as a task runner or rake task.

# Unresolved questions
[unresolved]: #unresolved-questions

Issues that are common to any alternative chosen:
* How to package rpms w/o internet connection, as Koji does not allow internet connection?
