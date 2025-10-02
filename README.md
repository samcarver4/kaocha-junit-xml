# lambdaisland/kaocha-junit-xml

<!-- badges -->
[![GitHub Actions](https://github.com/lambdaisland/kaocha-junit-xml/actions/workflows/main.yml/badge.svg)](https://github.com/lambdaisland/kaocha-junit-xml/actions/workflows/main.yml) [![cljdoc badge](https://cljdoc.org/badge/lambdaisland/kaocha-junit-xml)](https://cljdoc.org/d/lambdaisland/kaocha-junit-xml) [![Clojars Project](https://img.shields.io/clojars/v/lambdaisland/kaocha-junit-xml.svg)](https://clojars.org/lambdaisland/kaocha-junit-xml)
<!-- /badges -->

[Kaocha](https://github.com/lambdaisland/kaocha) plugin to generate a JUnit XML version of the test results.

<!-- opencollective -->
## Lambda Island Open Source

Thank you! kaocha-junit-xml is made possible thanks to our generous backers. [Become a
backer on OpenCollective](https://opencollective.com/lambda-island) so that we
can continue to make kaocha-junit-xml better.

<a href="https://opencollective.com/lambda-island">
<img src="https://opencollective.com/lambda-island/organizations.svg?avatarHeight=46&width=800&button=false">
<img src="https://opencollective.com/lambda-island/individuals.svg?avatarHeight=46&width=800&button=false">
</a>
<img align="left" src="https://github.com/lambdaisland/open-source/raw/master/artwork/lighthouse_readme.png">

&nbsp;

kaocha-junit-xml is part of a growing collection of quality Clojure libraries created and maintained
by the fine folks at [Gaiwan](https://gaiwan.co).

Pay it forward by [becoming a backer on our OpenCollective](http://opencollective.com/lambda-island),
so that we continue to enjoy a thriving Clojure ecosystem.

You can find an overview of all our different projects at [lambdaisland/open-source](https://github.com/lambdaisland/open-source).

&nbsp;

&nbsp;
<!-- /opencollective -->

## Usage

- Add kaocha-junit-xml as a dependency

``` clojure
;; deps.edn
{:aliases
 {:test
  {:extra-deps {lambdaisland/kaocha {...}
                lambdaisland/kaocha-junit-xml {:mvn/version "1.17.101"}}}}}
```

or

``` clojure
;; project.clj
(defproject ,,,
  :dependencies [,,,
                 [lambdaisland/kaocha-junit-xml "1.17.101"]])
```

- Enable the plugin and set an output file

``` clojure
;; tests.edn
#kaocha/v1
{:plugins [:kaocha.plugin/junit-xml]
 :kaocha.plugin.junit-xml/target-file "junit.xml"}
```

Or from the CLI

``` shell
bin/kaocha --plugin kaocha.plugin/junit-xml --junit-xml-file junit.xml
```

Optionally you can omit captured output from junit.xml

``` clojure
;; tests.edn
#kaocha/v1
{:plugins [:kaocha.plugin/junit-xml]
 :kaocha.plugin.junit-xml/target-file      "junit.xml"
 :kaocha.plugin.junit-xml/omit-system-out? true}
```

Or from the CLI

``` shell
bin/kaocha --plugin kaocha.plugin/junit-xml --junit-xml-file junit.xml --junit-xml-omit-system-out
```

## Requirements

Requires at least Kaocha 0.0-306 and Clojure 1.9.

## CI Integration

Some CI tooling supports the `junit` `xml` output in various flavours.

### CircleCI

One of the services that can use this output is CircleCI. Your
`.circleci/config.yml` could look like this:

``` yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/clojure:tools-deps-1.9.0.394
    steps:
      - checkout
      - run: mkdir -p test-results/kaocha
      - run: bin/kaocha --plugin kaocha.plugin/junit-xml --junit-xml-file test-results/kaocha/results.xml --junit-xml-add-location-metadata --junit-xml-use-relative-path-in-location
      - store_test_results:
          path: test-results
```

### GitHub Actions

The following configuration will create annotations for test failures on files of 
the relevant commit/PR. First enable the plugin with the `add-location-metadata?` 
flag in your `tests.edn`:

```edn
#kaocha/v1
{:plugins [:kaocha.plugin/junit-xml]
 :kaocha.plugin.junit-xml/target-file "junit.xml"
 :kaocha.plugin.junit-xml/add-location-metadata? true}
```

Then, an example `.github/workflows/build.yml` may look like: 

```yml
name: Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: clojure:openjdk-8-tools-deps-1.11.1.1113
    steps:
      - uses: actions/checkout@v2
      - name: test
        run: |
          bin/kaocha
      - name: Annotate failure
        if: failure()
        uses: mikepenz/action-junit-report@41a3188dde10229782fd78cd72fc574884dd7686
        with:
          report_paths: junit.xml
```

### Gitlab

Configuring Gitlab to parse JUnit XML is easy; just add a `report` artifact that
points to the XML file:

```yaml
test:
  only:
    -tags
  script:
    - make test
  artifacts:
    reports:
      junit: junit.xml
```

See the [Gitlab documentation on reports using
JUnit](https://docs.gitlab.com/ce/ci/junit_test_reports.html) for more information.

## Caveats

For timing information (timestamp and running time) this plugin relies on the
`kaocha.plugin/profiling` plugin. If the plugin is not present then a running
time of 0 will be reported.

For output capturing the `kaocha.plugin/capture-output` must be present. If it
is not present `<system-out>` will always be empty.

## Resources

It was hard to find a definitive source of the Ant Junit XML format. I mostly
went with [this page](http://llg.cubic.org/docs/junit/) for documentation.

For information on how to configure CircleCI to use this information, see
[store_test_results](https://circleci.com/docs/2.0/configuration-reference/#store_test_results).

After reports that the output was not compatible with Azure Devops Pipeline the
output was changed to adhere to [this schema](https://github.com/windyroad/JUnit-Schema/blob/49e95a79cc0bfba7961aaf779710a43a4d3f96bd/JUnit.xsd).

The `--junit-xml-add-location-metadata` flag was added to enhance `testcase` 
output with test location metadata à la 
[pytest](https://docs.pytest.org/en/latest/how-to/output.html?highlight=junit#creating-junitxml-format-files). 
This allows for integration with various tools on GitHub Actions for producing
annotations on files in commits/PRs with test failure data. For example, the 
[JUnit Report Action](https://github.com/marketplace/actions/junit-report-action).

The `--junit-xml-use-relative-path-in-location` flasg was added to enhance the `--junit-xml-add-location-metadata`
output for test ecosystems that rely on relative file-pathing.
Notably, CircleCI's [parallelization](https://circleci.com/docs/use-the-circleci-cli-to-split-tests/#junit-xml-reports) assumes file paths in JUnit files are relative to the runner's working directory.

<!-- contributing -->
## Contributing

We warmly welcome patches to kaocha-junit-xml. Please keep in mind the following:

- adhere to the [LambdaIsland Clojure Style Guide](https://nextjournal.com/lambdaisland/clojure-style-guide)
- write patches that solve a problem 
- start by stating the problem, then supply a minimal solution `*`
- by contributing you agree to license your contributions as EPL 1.0
- don't break the contract with downstream consumers `**`
- don't break the tests

We would very much appreciate it if you also

- update the CHANGELOG and README
- add tests for new functionality

We recommend opening an issue first, before opening a pull request. That way we
can make sure we agree what the problem is, and discuss how best to solve it.
This is especially true if you add new dependencies, or significantly increase
the API surface. In cases like these we need to decide if these changes are in
line with the project's goals.

`*` This goes for features too, a feature needs to solve a problem. State the problem it solves first, only then move on to solving it.

`**` Projects that have a version that starts with `0.` may still see breaking changes, although we also consider the level of community adoption. The more widespread a project is, the less likely we're willing to introduce breakage. See [LambdaIsland-flavored Versioning](https://github.com/lambdaisland/open-source#lambdaisland-flavored-versioning) for more info.
<!-- /contributing -->

<!-- license -->
## License

Copyright &copy; 2018-2025 Arne Brasseur and contributors

Available under the terms of the Eclipse Public License 1.0, see LICENSE.txt
<!-- /license -->