- Feature Name: update_asset_packaging
- Start Date: 2016-05-10
- RFC PR: https://github.com/theforeman/rfcs/pull/3
- Issue: 

# Summary
[summary]: #summary

The current way in which we manage asset dependencies is cumbersome and leads to various issues. This RFC proposes that we replace the current method with NPM and Webpack.

# Motivation
[motivation]: #motivation

Currently, most assets (js/css) are used as gems and managed by bundler.
A few are included directly in the vendor/ directory due to lack of gems.
Some of the reasons that we want to change the way we currently handle this:

* Some assets are not packaged as gems, so we need to either vendor or package them ourselves.
* Some asset gems are not updated at the same rate as the upstream source.
* Each gem dependency must be packaged separately as RPM/DEB.
* Difficult to integrate newer JS technology such as es2015 or js modules.
* Assets included in the vendor/ directory need manual updating when new versions are released.

# Detailed design
[design]: #detailed-design

NPM will be used to manage as many of the assets that are currently being vendored or bundled as possible.
If we run into issues with packages that are only available for Bower in the future, we will consider adding Bower as well. The main advantage of using only NPM is that we have a central location for all js dependencies and only one tool added rather then two, which reduces the complexity. NPM also allows namespacing JS modules using CommonJS to prevent possible collisions between various modules.

The node modules will be integrated into foreman using Webpack. Webpack has a powerful system of loaders that allow handling various tasks. Webpack also comes with a small webserver that allows live-reload of all assets managed by it, hopefully leading to increased developer happiness. In addition, webpack has various other strengths that make it seem like the current leading option - ability to handle css assets and all js module flavors, smart division of code into bundles and more.

As we will currently continue using the Rails assets pipeline, any plugin will be able to use whatever method they choose to handle any assets as long as the result is added to the pipeline (as is done currently).

Packaging, especially for RPMs, poses a challenge, as Koji builders do not allow internet connection. Building the js assets for production requires running `npm install` to download the needed libraries, followed by `rake webpack:compile` which compiles only the needed portions of the js libraries along with our code into bundles that are minified. Several approaches were considered to handle this:

* Packaging each node module as a separate package. This is unfeasible, since there are hundreds of modules required for the generation of the minified files. In addition, we would not want to package every single one ourselves, given that one of the goals of this change is to reduce packaging workload. However, some concern has been raised that this may be the only approach that fully conforms with the Fedora Packaging Guidelines.
* Committing either the `node_modules` directory or the compiled assets could be committed into source control. This has a downside of committing a large amount of generated code into git, and may lead to issues with forgetting to recompile before commit.
* Compiling and minifying the js assets before the build and add the resulting files into the tarball passed into the builder.
* Adding the `node_modules` directory to the source tarball before being sent to the builder, with the compiling and minification being run on the builder.

In the end, the approach taken was to package every top-level npm dependancy as a bundle with all its requirements. This reduces the number of npm packages that we need to repackage as rpms from hundredrs of individual modules to only a few dozen, which are all listed in package.json. The process of generating the rpm specs for the node modules is mostly automated, using https://github.com/dLobatog/npm2rpm. 
Debian packaging is not run in a disconnected environment, so packaging for Debian will simply run `npm install` during the build process to get the node modules needed for the build.

After a certain period of attempting this solution, we will review the choice and decide if we wish to continue in this direction or change to a different one.

# Drawbacks
[drawbacks]: #drawbacks

Some reasons to stay with the current solution:
* No extra work needed.
* Single package management system - bundler. All dependencies are in one central location, build process is simple.
* Rails asset pipeline doesn’t require any modifications to handle the current solution.

# Alternatives
[alternatives]: #alternatives

Several other alternatives were considered before choosing this option. The alternatives considered:

For package management - NPM, Bower, remain with current solution.

For handling the assets (getting them working with foreman) - Grunt, Gulp, Webpack, Browserify, Rake.

History of the discussion and pros and cons of each option can be read in the previous commits of this file.

# Unresolved questions
[unresolved]: #unresolved-questions
