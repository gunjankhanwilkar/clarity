# Local setup

Clarity uses NodeJS 10+ and NPM 6+ for development, so ensure you have them installed and up to date. To find the exact
version you could check `.nvmrc` file.

It also uses Docker for running visual diff tests, so if you plan to run those tests, you'll have to have Docker installed and running.

The project structure is as follow:

```bash
src
├── clr-angular   # All Angular Clarity components and styles
├── clr-core      # Web Components
├── clr-icons     # Clarity Icons
├── clr-ui        # Common CSS
├── dev           # Development Application for Angular
├── schematics    # Auto update scripts
└── website       # Website and Documentation
```

## Understanding the build

We have four packages:

- `@clr/icons` - Clarity Icons package, which is a standalone web component library for icons
- `@clr/ui` - Clarity UI package, which is a standalone CSS library for Clarity styles
- `@clr/angular` - Clarity Angular package, which depends upon the other two packages to implement a set of Angular components
- `@clr/core` - Clarity Web Components and common utilities.

Each package has a slightly different build process, and this guide describes them each separately.
Many of these commands have a corresponding `watch` command that enables watch
mode while you develop.

## Full Project Build

To build the entire repo and all projects run the command `npm run build:ci`.
This command is useful to run before submitting a PR to ensure everything will
pass the CI build.

## `@clr/angular`

Build Clarity Angular by running `npm run angular:build`, which calls the following tasks to build the package.

- `angular:build` - Build the Angular package for production
- `angular:start` - Run the test project for Clarity Angular Components
- `angular:test` - Run all tests for Clarity Angular
- `angular:test:watch` - Continually run all tests for Clarity Angular

## `@clr/core`

Build Clarity Core by running `npm run core:build`, which calls the following tasks to build the package.

- `core:build` - Builds the Core package for production
- `core:start` - Run the test project for Clarity Core Web Components
- `core:test` - Run all tests for Clarity Core
- `core:test:watch` - Continually run all tests for Clarity Core

## `@clr/icons`

Build Clarity Icons by running `npm run icons:build`, which calls the following tasks to build the package.

- `icons:build:web` - Webpack compiles and bundles the TypeScript assets
- `icons:build:css` - Sass compiles the styles
- `icons:build:optimize` - CSSO optimizes the CSS
- `icons:build:package` - Copy the `package.json` into the package, and set the version number
- `icons:build:web` - Build the raw svg files and zip directories for designers

## `@clr/ui`

Build Clarity UI is by running `npm run ui:build`, which calls the following tasks to build the package.

- `ui:build:css` - Sass compiles the light and dark theme files
- `ui:build:prefix` - Autoprefixer adds prefixes to CSS properties based on browser compatibility
- `ui:build:src` - Copy in the source files for anyone building directly
- `ui:build:optimize` - CSSO Optimize the CSS
- `ui:build:package` - Copy the `package.json` into the package, and set the version number

## Globally Installed NPM packages

The following packages are installed globally in the development environment. The purpose of each is listed below.
You won't need to install these for general development but may wish to do so if you want to run specific scripts for testing or publishing that require them:

- [@angular/cli](https://cli.angular.io/): The whole project uses this for build, preview, and testing.
- [gemini](https://gemini-testing.github.io/): this is used to run cli commands to run visual diff regression tests.
- [html-reporter](https://www.npmjs.com/package/html-reporter): plugin for Gemini to produce an HTML report of the CSS regression tests.

## Additional NPM Scripts

There are a few other NPM scripts that can be useful during build and development.

##### `npm start`

The start command starts up our demo app using the Angular CLI on port 4200 and watches for file changes for live reload.

##### `npm run build`

This script builds npm package candidates for all four packages we currently publish: `@clr/angular`, `@clr/ui`,
`@clr/icons`, and `@clr/core` under the `/dist` folder.

##### `npm test` and `npm run angular:test:watch`

This entry file now lives in src/clr-angular/test.tsand is now confined to what's under clr-angular. It used to be that the file was a single entry point across multiple packages. You can run the tests in watch mode, so they run continuously `npm run angular:test:watch`.

##### `npm run build:ci`

The `build:ci` script is used by Travis-CI to run all of the checks, such as format, lint, and unit tests.
If the code doesn't pass both the format and lint checkers, then the build fails before running the unit tests.

##### `npm run format` and `npm run format:fix`

We use [clang-format](https://github.com/angular/clang-format) as formatter for our code. `npm run format` will not
actually format the file but only check if there are any files that would be changed. We do this via a shell script (`./scripts/clang-check.sh`),
which runs the clang-format command (with `-output-replacements-xml` flag) and greps for any replacement that would be produced.

The `npm run format:fix` does the actual formatting according to the rules specified in `.clang-format` file.

##### `npm run angular:build`

This script produces the `@clr/angular` package using [ng-packagr](https://github.com/dherges/ng-packagr).

The script copies over the `package.json` template from our `npm` folder (this contains templates for `package.json` and
`README.md` for all of our packages) into `src/clr-angular` and sets the correct version number. This step is necessary
because `ng-packagr` requires the `package.json` to be at the root of the `src` (defined in `ng-package.json`).

##### `npm run icons:build`

This script produces the `clr-icons` package by bundling js files that is included in the consuming app.
The `webpack` script also processes the `package.json` and `README.md` files for all of our packages.
This means that running `npm icons:build` by itself does NOT produce a complete package.

##### `npm run core:build`

This script produces the `@clr/core` package that is used to consume the Clarity Core Web Components.

##### `npm run lint` and `npm run lint:fix`

The `lint` script runs the linter and fails if linting fails. The `lint:fix` script is very similar but
is run with the `--fix` flag to auto-fix some rules if possible. Some lint rules cannot be auto fixed, so you have
to fix those manually.

##### `npm run gemini` and `npm run gemini:fix`

These scripts use Docker to start up a container with selenium and chrome to run the Gemini tests. Currently, there are 4 sets
in our codebase, and these are arbitrary sets to parallelize running them in Travis builds. You must pass in the set(s) for both
of these scripts (e.g. `npm run gemini set1 set3`).

##### `npm run format:file -- path/to/file`

When contributing to Clarity, there is a post-commit hook installed and run with
[husky](https://github.com/typicode/husky) that formats the files staged before they are committed. There are
corner cases and editors that may not behave as expected, and it is possible to create a pull request that fails because
the files are not correctly formatted. This command can be used to format a specific file or a space-separated list of files.

##### `npm run angular:golden` and `npm run angular:golden:fix`

To update and test for changes against `clr-angular.d.ts`.

#### How to run visual tests

- You going to need Docker installed and running
- `npm run gemini:fix` - this runs all visual tests and compare them to there snapshots. It could take a long time,
  and also the changes that you made may not affect every test. A better solution is to run only the affected sets.
  `npm run gemini:fix set2 set4`
- Verify that the screenshots which can be found modified in your git changes reflect the changes you intended.
- Commit and push the updated screenshots. This is enough to trigger our CI builds again.
