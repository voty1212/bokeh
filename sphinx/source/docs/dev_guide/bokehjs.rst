.. _devguide_bokehjs:

BokehJS
=======

BokehJS is the in-browser client-side runtime library that users of Bokeh
ultimately interact with. This library is written primarily in TypeScript
and is one of the unique things about the Bokeh plotting system.

.. _devguide_bokehjs_motivations:

BokehJS Motivations
-------------------

When researching the wide field of JavaScript plotting libraries, we found
that they were all architected and designed to integrate with other JavaScript.
If they provided any server-side wrappers, those were always "second class"
and primarily designed to generate a simple configuration for the front-end JS.
Of the few JS plotting libraries that offered any level of interactivity, the
interaction was not really configurable or customizable from outside the JS
itself. Very few JS plotting libraries took large and streaming server-side
data into account, and providing seamless access to those facilities from
another language like Python was not a consideration.

This, in turn, has caused the developers of Python plotting libraries to
only treat the browser as a "backend target" environment, for which they
will generate static images or a bunch of JavaScript.

.. _devguide_bokehjs_goals:

Goals
-----

BokehJS is intended to be a standalone, first-class JavaScript plotting
library and *interaction runtime* for dynamic, highly-customizable
information visualization.

.. _devguide_bokehjs_interface:

Interface
---------

BokehJS is a standalone JavaScript library for dynamic and interactive
visualization in the browser. It is built on top of HTML5 canvas, and designed
for high-performance rendering of larger data sets. Its interface is declarative,
in the style of Protovis_, but its implementation consists of a reactive scene
graph (similar to Chaco_).

More information is available at :ref:`userguide_bokehjs`.

CSS Class Names
---------------

The CSS for controlling Bokeh presentation are located in a ``bokeh.css`` file
that is compiled from several separate ``.less`` files in the BokehJS source
tree. All CSS classes specifically for Bokeh DOM elements are prefixed with
the string ``bk-``. For instance some examples are: ``.bk-plot``, ``.bk-toolbar-button``, etc.

.. _devguide_bokehjs_development:

Development
-----------

BokehJS's source code is located in the :bokeh-tree:`bokehjs` directory in Bokeh's
monorepo repository. All further instructions and shell commands assume that
``bokehjs/`` is the current directory.

Some guidelines to adhere to when working on BokehJS:

* Do not use ``for-in`` loops, especially unguarded by ``hasOwnProperty()`` Use
  ``for-of`` loop in combination with ``keys()``, ``values()`` and/or
  ``entries()`` from the ``core/util/object`` module instead.

Requirements
~~~~~~~~~~~~

* node 14+
* npm 7.4+ (most recent version)
* chrome/chromium browser 90+ or equivalent

You can install nodejs with conda:

.. code-block:: sh

    $ conda install -c conda-forge nodejs

or follow the official installation `instructions <https://nodejs.org/en/download/>`_.

Upgrade your npm after installing or updating nodejs, or whenever asked by npm:

.. code-block:: sh

    $ npm install -g npm@7

Officially supported platforms are as follows:

* Linux Ubuntu 20.04+ or equivalent
* Windows 10 (or Server 2019)
* MacOS 10.15

BokehJS can be developed on different platforms and versions of aforementioned
software, but results may vary, especially when it comes to testing (visual
testing in particular).

Building
~~~~~~~~

BokehJS's build is maintained by using an in-house tool that visually resembles
gulp. All commands start with ``node make`` (don't confuse this with GNU make).

Most common commands:

* ``node make build``
* ``node make test``
* ``node make lint``

Use ``node make help`` to list all available commands.

``node make`` automatically runs ``npm install`` whenever ``package.json`` changes.

You can use ``tsc`` directly for error checking (e.g. in an IDE). However, don't use
it for code emit, because we rely on AST transforms to produce viable library code.

.. _devguide_bokehjs_development_testing:

Test suites
~~~~~~~~~~~

BokehJS comes with its own suites of tests and testing framework. All tests for BokehJS
use ``describe()`` and ``it()`` functions.

To launch BokehJS tests, run ``node make test`` from within the
:bokeh-tree:`bokehjs/test` directory.

Instead of running all available BokehJS tests, you can also run individual test
suites with ``node make test:suite_name``. Available tests suites are:

* ``node make test:codebase``: Codebase tests checking file size limits
* ``node make test:defaults``: Tests checking whether the defaults in Bokeh's
  Python models match those of Bokeh's TypeScript models
* ``node make test:unit``: Unit tests for BokehJS
* ``node make test:integration``:
  :ref:`Visual integration tests <devguide_bokehjs_development_visual_testing>`
  comparing locally generated plots against a set of baseline files

You can combine the last two test suites by running ``node make test:lib``.

Unit and integration tests are run in a web browser (see requirements). The test framework
automatically starts a web browser with the right settings to ensure consistent test
results.

.. _devguide_bokehjs_development_devtoolsserver:

Testing with Devtool Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In addition to tests run from the command line, you can also use the BokehJS's devtools
server. Use this system to run tests and review the visual tests' output.

First, start the devtools server with the following command:

.. code-block:: sh

    $ node test/devtools server
    listening on 127.0.0.1:5777

Next, open the displayed server URL in a web browser and navigate to
``/integration/report``.

You can also use the devtools server to initiate test runs. You have two options:

Run tests from a JavaScript console
  Open one of these three endpoints in your web browser:

  * ``/unit``
  * ``/defaults``
  * ``/integration``

  This loads BokehJS and the tests. To run the tests, issue ``Tests.run_all()`` in your
  browser's JavaScript console. This allows you to set breakpoints before running code.
  You can filter out tests by providing a keyword or a regular expression.

Trigger tests with endpoint
  Initiate test runs by accessing one of the following endpoints with your browser:

  * ``/unit/run``
  * ``/defaults/run``
  * ``/integration/run``

  To only run or view specific tests, append ``?k=some%20text`` to the URL. This will
  filter tests by keyword.

  To only run or view tests for a specific plattform, append either ``platform=linux``
  ``platform=macos``, or ``platform=windows`` to the URL.

.. _devguide_bokehjs_development_visual_testing:

CI and Visual Testing
~~~~~~~~~~~~~~~~~~~~~

BokehJS uses a series of visual baseline comparison tests. These tests help make sure
that Bokeh's visual output is consistent with the output expected by design. Any
BokehJS-related pull requests that result in changes to the visual output generated by
BokehJS should include visual baseline comparison tests.

In the background, BokehJS' testing framework runs a headless browser and takes
screenshots of the browser's output. The testing framework then compares the visual
output to each test's dedicated baseline files.

Each test in ``test:integration`` consists of two types of baseline comparisons:

Textual baseline comparison
  For each test, the testing framework compares the pixel location of certain elements
  in the visual output to pixel locations in the baseline data. This baseline data is
  stored as plain text in each test's respective ``.blf`` file.

Visual baseline comparison
  For each test, the testing framework does a pixel-by-pixel comparison of a screenshot
  and a baseline image. These baseline images are stored as ``.png`` files. In contrast
  to textual baseline comparisons, visual baseline comparisons are platform-dependent.
  Even minor differences in font rendering, for example, will make the pixel-by-pixel
  comparison fail.

The visual baseline comparison tests are located in the
:bokeh-tree:`bokehjs/test/integration/` folder and sub-folders.
:ref:`Bokeh's CI <devguide_testing_ci>` runs these tests on Linux, macOS, and Windows
environments. The baseline files for each environment are located in the
:bokeh-tree:`bokehjs/test/baselines/` folder.

Follow these steps to write new visual tests or update existing tests:

1. Create or update visual testing scripts:
    To write a visual test for BokehJS' testing framework, start by importing the
    ``display()`` and ``fig()`` functions from the testing framework's ``_util`` module
    (located in :bokeh-tree:`bokehjs/test/integration/`):

    .. code-block:: TypeScript

        import {display, fig} from "./_util"

    When writing tests, replace BokehJS' standard ``show()`` function with the
    ``display()`` function in ``_util``. ``display()`` accepts the same arguments as
    ``show()`` but also captures the visual output for comparison.

    Similarly, replace BokehJS' standard ``figure()`` with the ``fig()`` function in
    ``_util``. ``fig()`` expects an array of ``[width, height]`` as the first argument,
    followed by the same arguments as ``figure()``. To keep visual tests as efficient as
    possible, you should only use ``width`` and ``height``.

    Keep the width and height of your testing plot as small as possible while still
    being able to see the details you want to test with the naked eye. Try to keep the
    number of elements on your plot to a minimum.

    Follow this general pattern for visual tests:

    .. code-block:: TypeScript

      describe("Your Object", () => {
        it("should show certain behavior", async () => {
          const p = fig([width, height], {figure_attrs})

          ...

          await display(p)
        })
      })

    To change the sensitivity of a visual test, you have the option to set a
    threshold value. The threshold value represents the amounts of pixels by which
    a test image can differ from the baseline image before a test fails. To set a
    threshold value, use ``it.allowing(threshold)``. For example:

    .. code-block:: TypeScript

      describe("Your Object", () => {
        it.allowing(16)("should show certain behavior", async () => {

    Always run ``node make lint`` before committing TypeScript files.

2. Run tests locally:
    Run ``node make tests`` to test your changes on your system. To only run integration
    tests, use ``node make test:integration``.

    If you want to only run a specific test, use the ``-k`` argument and supply a search
    string. The search string is case-sensitive. The BokehJS testing framework tries to
    match your search string to the strings defined in the code's ``describe()`` and
    ``it()`` functions. For example:

    .. code-block:: sh

      $ node make test:integration -k 'Legend annotation'

    The first time you run a new or updated visual test, the BokehJS testing framework
    will notify you that baseline files are missing our outdated. At this point, it will
    also generate all missing or outdated baseline files for your operating system. The
    baseline files will be in a subfolder of :bokeh-tree:`bokehjs/test/baselines/`.

    Use the BokehJS :ref:`devtools server <devguide_bokehjs_development_devtoolsserver>`
    to review your local test results. Optionally, you can use any PNG viewer to inspect
    the generated PNG files. Adjust your testing code until the test's visual output
    matches your expectations.

3. Generate CI baselines and commit test:
    As a final step before pushing your visual tests to Bokeh's GitHub repository, you
    need to generate and commit the baseline files using
    :ref:`Bokeh's CI <devguide_testing_ci>`.

    The baseline files are platform-dependent. This is why the CI will only work
    reliably if you upload baseline files that were created by the CI, not locally
    created files.

    Follow these steps to generate the necessary baseline files and upload them to
    Bokeh's CI:

    1. Push your changes to GitHub and wait for CI to finish.
    2. The CI will expectedly fail because baseline images are either missing (in case
       you created new tests) or outdated (in case you updated existing tests).
    3. After the CI has finished running, go to BokehJS's GitHub_Actions_ page. Find the
       most recent test run for your PR and download the associated ``bokehjs-report``
       artifact.
    4. Unzip the downloaded artifact file into the root folder of your local Bokeh
       repository.
    5. Use the :ref:`devtools server <devguide_bokehjs_development_devtoolsserver>` to
       review the baseline files the CI has created for each platform: first, go to
       ``/integration/report?platform=linux``, then to
       ``/integration/report?platform=macos``, and finally to
       ``/integration/report?platform=windows``.
    6. If you did not detect any unintentional differences, commit all new or modified
       ``*.blf`` and ``*.png`` files from the folders
       :bokeh-tree:`bokehjs/test/baselines/linux`,
       :bokeh-tree:`bokehjs/test/baselines/macos`, and
       :bokeh-tree:`bokehjs/test/baselines/windows`.
    7. Push your changes to GitHub again and verify that the tests pass this time.

    .. note::
      Make sure to only push baseline files to the CI that were created by the CI
      for your specific pull request. Do not include any locally created baseline
      files in your pull request.

      After downloading and unpacking the baseline files from the CI, check your local
      :bokeh-tree:`bokehjs/test/baselines` directory for any modified files that are not
      part of your changes. Make sure only to commit baseline files that are necessary
      for your pull request. Reset the ``baselines`` directory after every failed test
      run (``git checkout`` and/or ``git clean``).

Debugging in Headless Chrome
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Although testing in headless Chrome and running tests manually in Chrome should agree
with each other most of the time, there are rare cases where headless and GUI Chrome
diverge. In this situation one has to debug BokehJS' code directly in the headless
browser.

Start BokehJS' devtools server in one console and run ``node make test:run:headless``
in another. This starts Chrome in headless mode preconfigured for bokehjs' testing
setup. Then open Chrome (or any other web browser), navigate to http://localhost:9222 and
click ``about:blank`` link. This opens remote devtools console. Use its navigation bar
and navigate to e.g. http://localhost:5777/integration/run (or other URL mentioned in
an earlier paragraph). You are now set up for debugging in headless Chrome.

Minimal Model/View Module
~~~~~~~~~~~~~~~~~~~~~~~~~

Models (and views) come in many forms and sizes. At minimum, a model is implemented.
A view may follow if a "visual" model is being implemented. A minimal model/view
module looks like this:

.. code-block:: typescript

    import {BaseModel, BaseModelView} from "models/..."

    export class SomeModelView extends BaseModelView {
      model: SomeModel

      initialize(): void {
        super.initialize()
        // perform view initialization (remove if not needed)
      }

      async lazy_initialize(): Promise<void> {
        await super.lazy_initialize()
        // perform view lazy initialization (remove if not needed)
      }
    }

    export namespace SomeModel {
      export type Attrs = p.AttrsOf<Props>

      export type Props = BaseModel.Props & {
        some_property: p.Property<number>
        // add more property declarations
      }
    }

    export interface SomeModel extends SomeModel.Attrs {}

    export class SomeModel extends BaseModel {
      properties: SomeModel.Props
      __view_type__: SomeModelView

      // do not remove this constructor, or you won't be
      // able to use `new SomeModel({some_property: 1})`
      constructor(attrs?: Partial<SomeModel.Attrs>) {
        super(attrs)
      }

      static {
        this.prototype.default_view = SomeModelView

        this.define<SomeModel.Props>(({Number}) => ({
          some_property: [ Number, 0 ],
          // add more property definitions
        }))
      }
    }

For trivial modules like this, most of the code is just boilerplate to make
BokehJS's code statically type-check and generate useful type declarations
for further consumption (in tests or by users).

Code Style Guide
~~~~~~~~~~~~~~~~

BokehJS doesn't have an explicit style guide. Make your changes consistent in
formatting. Use ``node make lint``. Follow patterns observed in the surrounding
code and apply common sense.

.. _Chaco: https://github.com/enthought/chaco
.. _JSFiddle: http://jsfiddle.net/
.. _Protovis: http://mbostock.github.io/protovis/
.. _GitHub_Actions: https://github.com/bokeh/bokeh/actions?query=workflow%3ABokehJS-CI
