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

NPM will be used to manage as many of the assets that are currently being vendored or bundled as possible.
If we run into issues with packages that are only available for Bower in the future, we will consider adding Bower as well. The main advantage of using only NPM is that we have a central location for all js dependencies and only one tool added rather then two, which reduces the complexity. NPM also allows namespacing JS modules using CommonJS to prevent possible collisions between various modules.
The node modules will be integrated into the asset pipeline using one of the methods detailed under the Alternatives heading below.
As we will currently continue using the Rails assets pipeline, any plugin will be able to use whatever method they choose to handle any assets as long as the result is added to the pipeline (as is done currently).

# Drawbacks
[drawbacks]: #drawbacks

Some reasons to stay with the current solution:
* No extra work needed.
* Single package management system - bundler. All dependencies are in one central location, build process is simple.
* Rails asset pipeline doesn’t require any modifications to handle the current solution.

# Alternatives
[alternatives]: #alternatives

__Option 1: Use Grunt task runner__

_Pros:_
* Existing knowledge base and tasks from Katello/Bastion.

_Cons:_
* Each task needs manual set up and configuration.
* Creates multiple intermediate files while running.

__Option 2: Use Gulp task runner__

_Pros:_
* Runs faster then Grunt and with less configuration.
* Tasks can be built as streams of smaller units.

_Cons:_
* Each task needs manual set up and configuration.
* Little relevant knowledge within the current core team.

__Option 3: Webpack__

[POC PR](https://github.com/theforeman/foreman/pull/3433)

_Pros:_
* Live reload development server.
* Ability to package the code in separate chunks that are only loaded when needed.
* Multiple loaders allow multiple tasks be run without requiring an additional task runner and with minimal configuration.
* Can possibly replace rails asset pipeline in the future if desired.

_Cons:_
* More complex configuration compared to webpack.
* Less examples in other projects.
* Possible that some tasks may still require an additional runner.

__Option 4: Browserify__

[POC PR](https://github.com/theforeman/foreman/pull/3326)

_Pros:_
* Simple installation and configuration.
* Easy to tie into rails asset pipeline.
* Possible to handle certain tasks with transforms.

_Cons:_
* Only handles JS assets in its core (some transforms may be able to help).
* Possible that some tasks may still require an additional runner.
* Integration with Bower seems more complex.

__Option 5: Use Rake__

_Pros:_
* Write tasks in ruby rather then js - accessible to all team members.
* Integration with plugins will possibly be easier if needed in the future.

_Cons:_
* Each task needs manual set up and configuration.
* Using a non-native solution for js that may lead to some of the issues we currently face with bundler.
* We would need to create this solution from scratch with no examples rather then use an existing one.

# Unresolved questions
[unresolved]: #unresolved-questions

Issues that are common to any alternative chosen:
* How to package rpms w/o internet connection, as Koji does not allow internet connection?
** This could possibly be resolved by committing the compiled assets into source control prior to package builds, as packages only provide compiled assets and therefor don't require the entire development environment.

