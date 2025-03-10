.. spelling::

  ing
  ver

.. _conanfile_attributes:

Attributes
==========

name
----
This is a string, with a minimum of 2 and a maximum of 50 characters (though shorter names are recommended), that defines the package name. It will be the ``<pkgName>/version@user/channel`` of the package reference.
It should match the following regex ``^[a-zA-Z0-9_][a-zA-Z0-9_\+\.-]{1,50}$``, so start with alphanumeric or underscore, then alphanumeric, underscore, +, ., - characters.

The name is only necessary for ``export``-ing the recipe into the local cache (``export`` and ``create`` commands), if they are not defined in the command line.
It might take its value from an environment variable, or even any python code that defines it (e.g. a function that reads an environment variable, or a file from disk).
However, the most common and suggested approach would be to define it in plain text as a constant, or provide it as command line arguments.


version
-------
The version attribute will define the version part of the package reference: ``pkgName/<version>@user/channel``
It is a string, and can take any value, matching the same constraints as the ``name`` attribute.
In case the version follows semantic versioning in the form ``X.Y.Z-pre1+build2``, that value might be used for requiring this package through version ranges instead of exact versions.

The version is only strictly necessary for ``export``-ing the recipe into the local cache (``export`` and ``create`` commands), if they are not defined in the command line.
It might take its value from an environment variable, or even any python code that defines it (e.g. a function that reads an environment variable, or a file from disk).
Please note that this value might be used in the recipe in other places (as in ``source()`` method to retrieve code from elsewhere), making this value not constant means that it may evaluate differently in different contexts (e.g., on different machines or for different users) leading to unrepeatable or unpredictable results.
The most common and suggested approach would be to define it in plain text as a constant, or provide it as command line arguments.


description
-----------
This is an optional, but strongly recommended text field, containing the description of the package,
and any information that might be useful for the consumers. The first line might be used as a
short description of the package.

.. code-block:: python

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        description = """This is a Hello World library.
                        A fully featured, portable, C++ library to say Hello World in the stdout,
                        with incredible iostreams performance"""

homepage
--------

Use this attribute to indicate the home web page of the library being packaged. This is useful to link
the recipe to further explanations of the library itself like an overview of its features, documentation, FAQ
as well as other related information.

.. code-block:: python

    class EigenConan(ConanFile):
        name = "eigen"
        version = "3.3.4"
        homepage = "http://eigen.tuxfamily.org"

.. _package_url:

url
---

It is possible, even typical, if you are packaging a third party lib, that you just develop
the packaging code. Such code is also subject to change, often via collaboration, so it should be stored
in a VCS like git, and probably put on GitHub or a similar service. If you do indeed maintain such a
repository, please indicate it in the ``url`` attribute, so that it can be easily found.

.. code-block:: python

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        url = "https://github.com/conan-io/hello.git"

The ``url`` is the url **of the package** repository, i.e. not necessarily the original source code.
It is optional, but highly recommended, that it points to GitHub, Bitbucket or your preferred
code collaboration platform. Of course, if you have the conanfile inside your library source,
you can point to it, and afterwards use the ``url`` in your ``source()`` method.

This is a recommended, but not mandatory attribute.

license
-------

This field is intended for the license of the **target** source code and binaries, i.e. the code
that is being packaged, not the ``conanfile.py`` itself. This info is used to be displayed by
the :command:`conan info` command and possibly other search and report tools.

.. code-block:: python

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        license = "MIT"

This attribute can contain several, comma separated licenses. It is a text string, so it can
contain any text, including hyperlinks to license files elsewhere.

However, we strongly recommend packagers of Open-Source projects to use
`SPDX <https://spdx.dev>`_ identifiers from the `SPDX license
list <https://spdx.dev/licenses/>`_ instead of free-formed text. This will help
people wanting to automate license compatibility checks, like consumers of your
package, or you if your package has Open-Source dependencies.

This is a recommended, but not mandatory attribute.

author
------

Intended to add information about the author, in case it is different from the Conan user. It is
possible that the Conan user is the name of an organization, project, company or group, and many
users have permissions over that account. In this case, the author information can explicitly
define who is the creator/maintainer of the package

.. code-block:: python

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        author = "John J. Smith (john.smith@company.com)"

This is an optional attribute.

topics
------

Topics provide a useful way to group related tags together and to quickly tell developers what a
package is about. Topics also make it easier for customers to find your recipe. It could be useful
to filter packages by topics.

The ``topics`` attribute should be a tuple with the needed topics inside.

.. code-block:: python

    class ProtocInstallerConan(ConanFile):
        name = "protoc_installer"
        version = "0.1"
        topics = ("protocol-buffers", "protocol-compiler", "serialization", "rpc")

This is an optional attribute.

.. _user_channel:

user, channel
-------------

**These fields are optional in a Conan reference**, they could be useful to identify a forked recipe
from the community with changes specific for your company. Using these fields you may keep the
same ``name`` and ``version`` and use the ``user/channel`` to disambiguate your recipe.

The value of these fields can be accessed from within a ``conanfile.py``:

.. code-block:: python

    from conans import ConanFile

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"

        def requirements(self):
            self.requires("common-lib/version")
            if self.user and self.channel:
                # If the recipe is using them, I want to consume my fork.
                self.requires("say/0.1@%s/%s" % (self.user, self.channel))
            else:
                # otherwise, I'll consume the community one
                self.requires("say/0.1")


Only packages that have already been exported (packages in the local cache or in a remote server)
can have a user/channel assigned. For package recipes working in the user space, there is no
current user/channel by default, although they can be defined at :command:`conan install` time with:

.. code-block:: bash

    $ conan install <path to conanfile.py> user/channel

.. seealso::

    FAQ: :ref:`faq_recommendation_user_channel`

.. warning::

    Environment variables ``CONAN_USERNAME`` and ``CONAN_CHANNEL`` that were used to assign a value
    to these fields are now deprecated and will be removed in Conan 2.0. Don't use them to
    populate the value of ``self.user`` and ``self.channel``.


default_user, default_channel
-----------------------------

For package recipes working in the user space, with local methods like :command:`conan install .` and :command:`conan build .`,
there is no current user/channel. If you are accessing to ``self.user`` or ``self.channel`` in your recipe,
you need to declare the environment variables ``CONAN_USERNAME`` and ``CONAN_CHANNEL`` or you can set the attributes
``default_user`` and ``default_channel``. You can also use python ``@property``:

.. code-block:: python

    from conans import ConanFile

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        default_user = "myuser"

        @property
        def default_channel(self):
            return "mydefaultchannel"

        def requirements(self):
            self.requires("pkg/0.1@%s/%s" % (self.user, self.channel))


.. _settings_property:

settings
--------

There are several things that can potentially affect a package being created, i.e. the final
package will be different (a different binary, for example), if some input is different.

Development project-wide variables, like the compiler, its version, or the OS
itself. These variables have to be defined, and they cannot have a default value listed in the
conanfile, as it would not make sense.

It is obvious that changing the OS produces a different binary in most cases. Changing the compiler
or compiler version changes the binary too, which might have a compatible ABI or not, but the
package will be different in any case.

For these reasons, the most common convention among Conan recipes is to distinguish binaries by the following four settings, which is reflected in the `conanfile.py` template used in the `conan new` command:

.. code-block:: python

    settings = "os", "compiler", "build_type", "arch"

When Conan generates a compiled binary for a package with a given combination of the settings above, it generates a unique ID for that binary by hashing the current values of these settings.

But what happens for example to **header only libraries**? The final package for such libraries is not
binary and, in most cases it will be identical, unless it is automatically generating code.
We can indicate that in the conanfile:

.. code-block:: python

   from conans import ConanFile

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        # We can just omit the settings attribute too
        settings = None

        def build(self):
            #empty too, nothing to build in header only

You can restrict existing settings and accepted values as well, by redeclaring the settings
attribute:

.. code-block:: python

    class HelloConan(ConanFile):
        settings = {"os": ["Windows"],
            "compiler": {"Visual Studio": {"version": [11, 12]}},
            "arch": None}

In this example we have just defined that this package only works in Windows, with VS 10 and 11.
Any attempt to build it in other platforms with other settings will throw an error saying so.
We have also defined that the runtime (the MD and MT flags of VS) is irrelevant for us
(maybe we using a universal one?). Using None as a value means, *maintain the original values* in order
to avoid re-typing them. Then, "arch": None is totally equivalent to "arch": ["x86", "x86_64", "arm"]
Check the reference or your ~/.conan/settings.yml file.

As re-defining the whole settings attribute can be tedious, it is sometimes much simpler to
remove or tune specific fields in the ``configure()`` method. For example, if our package is runtime
independent in VS, we can just remove that setting field:

.. code-block:: python

    settings = "os", "compiler", "build_type", "arch"

    def configure(self):
        self.settings.compiler["Visual Studio"].remove("runtime")

It is possible to check the settings to implement conditional logic, with attribute syntax:

.. code-block:: python

    def build(self):
        if self.settings.os == "Windows" and self.settings.compiler.version == "15":
            # do some special build commands
        elif self.settings.arch == "x86_64":
            # Other different commands

Those comparisons do content checking, for example if you do a typo like ``self.settings.os == "Windows"``,
Conan will fail and tell you that is not a valid ``settings.os`` value, and the possible range of values.

Likewise, if you try to access some setting that doesn't exist, like ``self.settings.compiler.libcxx``
for the ``Visual Studio`` setting, Conan will fail telling that ``libcxx`` does not exist for that compiler.

If you want to do a safe check of settings values, you could use the ``get_safe()`` method:

.. code-block:: python

    def build(self):
        # Will be None if doesn't exist
        arch = self.settings.get_safe("arch")
        # Will be None if doesn't exist
        compiler_version = self.settings.get_safe("compiler.version")
        # Will be the default version if the return is None
        build_type = self.settings.get_safe("build_type", default="Release")

The ``get_safe()`` method will return ``None`` if that setting or subsetting doesn't exist and there is no default value assigned.

If you want to do a safe deletion of settings, you could use the ``rm_safe()``
method. For example, in the ``configure()`` method a typical pattern for a C library would
be:

.. code-block:: python

    def configure(self):
        self.settings.rm_safe("compiler.libcxx")
        self.settings.rm_safe("compiler.cppstd")


.. _conanfile_options:

options
-------

Conan provides this attribute to declare traits which will affect only one reference, unlike the settings that are typically the
same for all the recipes in a Conan graph. Options are declared per recipe, this attribute consist on a dictionary where the key is the
option name and the value is the list of different values that the option can take.

.. important::

    All the options with their values are encoded into the package ID, as everything that affects the generated binary.
    See :ref:`method_configure_config_options` and :ref:`method_package_id` methods for information about removing certain
    options for some configurations.


A very common one is the option ``shared`` with allowed values of ``[True, False]`` that many recipes declare and use to configure the
build system to produce a static library or a shared library.

Values for each option can be typed or plain strings (``"value"``, ``True``, ``None``, ``42``,...) and there is a special value, ``"ANY"``, for
options that can take any value. When an option uses ``"ANY"``, but its default value is ``None``, then it should be added to the possible option values too.

.. code-block:: python

    class MyPkg(ConanFile):
        ...
        options = {
            "shared": [True, False],
            "option1": ["value1", "value2"],
            "option2": ["ANY"],
            "option3": [None, "value1", "value2"],
            "option4": [True, False, "value"],
            "option5": [None, "ANY"],
        }

Every option in a recipe needs to be assigned a value from the ones declared in the ``options`` attribute. The
consumer can define options using different methods: command line, profile or consumer
recipes; **an uninitialized option will get the value** ``None`` **and it will be a valid value if it is contained
in the list of valid values**. Invalid values will produce an error. See attribute :ref:`conanfile_default_options`
for a way to declare a default value for options in a recipe.

.. tip::

    - You can inspect available package options reading the package recipe, which can be
      done with the command :command:`conan inspect mypkg/0.1@user/channel`.

.. tip::

    - Options ``"shared": [True, False]`` and ``"fPIC": [True, False]`` are automatically managed in :ref:`cmake_reference` and
      :ref:`autotools_reference` build helpers.


**Define the value of an option**

As we mentioned before, values for options in a recipe can be defined using different ways, let's
go over all of them for the example recipe ``mypkg`` defined above:

- In the recipe that declares the option:

  + Using the attribute ``default_options`` in the recipe itself.

  + In the ``config_options()`` method of the recipe.

  + In the ``configure()`` method of the recipe itself (**this one has the highest precedence**, this
    value can't be overridden)

    .. code-block:: python

        class MyPkg(ConanFile):

            options = {
                "shared": [True, False],
                "option1": ["value1", "value2"],
                "option2": ["ANY"],
            }

            def configure(self):
                if some_condition:
                    self.options.shared = False

- From a recipe that requires this one:

  + using the ``default_options`` attribute of the consumer:

    .. code-block:: python

        class OtherPkg(ConanFile):
            requires = "mypkg/0.1@user/channel"
            default_options = {"mypkg:shared": False}

  + in the ``configure()`` method of the consumer (**highest precedence after** ``configure()`` **in the recipe itself**):

    .. code-block:: python

        class OtherPkg(ConanFile):
            requires = "mypkg/0.1@user/channel"

            def configure(self):
                self.options['mypkg'].shared = False

    This method allows to assign values based on other conditions, it can have some drawbacks
    as it is explainded in the :ref:`mastering section<conditional_settings_options_requirements>`.

- In the *conanfile.txt* file:

  .. code-block:: text

      [requires]
      mypkg/0.1@user/channel

      [options]
      mypkg:shared=False

- It is also possible to define default values for the options of a recipe using
  :ref:`profiles<profiles>`. They will apply whenever that recipe is used:

  .. code-block:: text

      [settings]
      setting=value

      [options]
      mypkg:shared=False

- Last way of defining values for options is to pass these values using the command argument :command:`-o,--option` in the command line:

  .. code-block:: bash

    $ conan install . -o mypkg:shared=True

Regarding the precedence of all these ways of assigning a value to an option, it works like any other configuration element in Conan:
the closer to the consumer and the command line the higher the precedence. The list above is ordered from the less priority to the
highest one (with the exceptional assignment in ``configure()`` method which cannot be overridden).


**Get the value of an option**

Values from options can be retrieved after they are assigned. For options that belong to the same recipe, the value can
be retrieved in any method to run logic conditional to their values. **Options from required packages can be
retrieved only after the full graph has been resolved**, this means that the value will be available in the methods
``validate()``, ``build()``, ``package()``, ``package_info()``. Accessing those values in other methods can lead to unexpected results.


.. code-block:: python

    class OtherPkg(ConanFile):
        requires = "mypkg/0.1@user/channel"

        def validate(self):
            if self.options['mypkg'].shared:
                raise ConanInvalidConfiguration("Cannot use shared library of requirement 'mypkg'")

If you want to retrieve the value of an option and fallback to a known value if the option doesn't exist
you can use the ``get_safe()`` method:

.. code-block:: python

    def build(self):
        # Will return None if doesn't exist
        fpic = self.options.get_safe("fPIC")
        # Will return the default value if the return is None
        shared = self.options.get_safe("shared", default=False)

The ``get_safe()`` method will return ``None`` if that option doesn't exist and there is no default value assigned.

If you want to do a safe deletion of options, you could use the ``rm_safe()`` method. For
example, in the ``config_options()`` method a typical pattern for Windows library would be:

.. code-block:: python

    def config_options(self):
        if self.settings.os == "Windows":
            self.options.rm_safe("fPIC")

**Evaluate options**

It is very important to know how the options are evaluated in conditional expressions and how the
comparison operator works with them:

- Evaluation for the typed value and the string one is the same, so all these inputs would
  behave the same:

    - ``default_options = {"shared": True, "option": None}``
    - ``default_options = {"shared": "True", "option": "None"}``
    - ``mypkg:shared=True``, ``mypkg:option=None`` on profiles, command line or *conanfile.txt*

- **Implicit conversion to boolean is case insensitive**, so the
  expression ``bool(self.options.option)``:

    - equals ``True`` for the values ``True``, ``"True"`` and ``"true"``, and any other value that
      would be evaluated the same way in Python code.
    - equals ``False`` for the values ``False``, ``"False"`` and ``"false"``, also for the empty
      string and for ``0`` and ``"0"`` as expected.

- Comparison using ``is`` is always equals to ``False`` because the types would be different as
  the option value is encapsulated inside a Python class.

- Explicit comparisons with the ``==`` symbol **are case sensitive**, so:

    - ``self.options.option = "False"`` satisfies ``assert self.options.option == False``,
      ``assert self.options.option == "False"``, but ``assert self.options.option != "false"``.

- A different behavior has ``self.options.option = None``, because
  ``assert self.options.option != None``.



.. _conanfile_default_options:

default_options
---------------

The attribute ``default_options`` has the purpose of defining the default values for the options
if the consumer (consuming recipe, project, profile or the user through the command line) does
not define them. This attribute should be defined as a python dictionary:

.. code-block:: python

    class MyPkg(ConanFile):
        ...
        options = {"build_tests": [True, False],
                   "option1": ["value1", "value2"],
                   "option2": ["ANY"],
                   "option3": [None, "ANY"],
                   }
        default_options = {"build_tests": True,
                           "option1": "value1",
                           "option2": 42,
                           "option3": None,
                           }

        def build(self):
            cmake = CMake(self)
            cmake.definitions['BUILD_TESTS'] = self.options.build_tests
            cmake.configure()
            ...

Remember that you can also assign default values for options of your requirements as we've seen in
the attribute :ref:`conanfile_options`.

You can also set the options conditionally to a final value with ``configure()`` instead of using ``default_options``:

.. code-block:: python

    class OtherPkg(ConanFile):
        settings = "os", "arch", "compiler", "build_type"
        options = {"some_option": [True, False]}
        # Do NOT declare 'default_options', use 'config_options()'

        def configure(self):
            if self.options.some_option == None:
                if self.settings.os == 'Android':
                    self.options.some_option = True
                else:
                    self.options.some_option = False

Take into account that if a value is assigned in the ``configure()`` method it cannot be overridden.

.. important::

    Default options can be specified as a dictionary only for Conan version >= 1.8.

.. seealso::

    Read more about the :ref:`config_options()<method_configure_config_options>` method.

requires
--------

Specify package dependencies as a list or tuple of other packages:

.. code-block:: python

    class MyLibConan(ConanFile):
        requires = "hello/1.0@user/stable", "OtherLib/2.1@otheruser/testing"

You can specify further information about the package requirements:

.. code-block:: python

    class MyLibConan(ConanFile):
        requires = [("hello/0.1@user/testing"),
                    ("say/0.2@dummy/stable", "override"),
                    ("bye/2.1@coder/beta", "private")]

.. code-block:: python

    class MyLibConan(ConanFile):
        requires = (("hello/1.0@user/stable", "private"), )


Requirements can be complemented by 2 different parameters:

**private**: a dependency can be declared as private if it is going to be fully embedded and hidden
from consumers of the package. It might be necessary in some extreme cases, like having to use two
different versions of the same library (provided that they are totally hidden in a shared library, for
example), but it is mostly discouraged otherwise.

**override**: packages can define overrides of their dependencies, if they require the definition of
specific versions of the upstream required libraries, but not necessarily direct dependencies. For example,
a package can depend on A(v1.0), which in turn could conditionally depend on Zlib(v2), depending on whether
the compression is enabled or not. Now, if you want to force the usage of Zlib(v3) you can:

..  code-block:: python

    class HelloConan(ConanFile):
        requires = ("ab/1.0@user/stable", ("zlib/3.0@other/beta", "override"))

This **will not introduce a new dependency**, it will just change ``zlib/2.0`` to ``zlib/3.0`` if ``ab`` actually
requires it. Otherwise zlib will not be a dependency of your package.

.. note::

    To prevent accidental override of transitive dependencies, check the config variable
    :ref:`general.error_on_override<conan_conf>` or the environment variable
    :ref:`CONAN_ERROR_ON_OVERRIDE<env_vars_conan_error_on_override>`.

.. _version_ranges_reference:

version ranges
++++++++++++++

The syntax is using brackets:

..  code-block:: python

    class HelloConan(ConanFile):
        requires = "pkg/[>1.0 <1.8]@user/stable"

Expressions are those defined and implemented by [python node-semver](https://pypi.org/project/node-semver/). Accepted expressions would be:

..  code-block:: python

    >1.1 <2.1    # In such range
    2.8          # equivalent to =2.8
    ~=3.0        # compatible, according to semver
    >1.1 || 0.8  # conditions can be OR'ed

.. container:: out_reference_box

    Go to :ref:`Mastering/Version Ranges<version_ranges>` if you want to learn more about version ranges.

tool_requires
--------------

Tool requirements are requirements that are only installed and used when the package is built from sources. If there is an existing pre-compiled binary, then the tool requirements for this package will not be retrieved.

They can be specified as a comma separated tuple in the package recipe:

.. code-block:: python

    class MyPkg(ConanFile):
        tool_requires = "tool_a/0.2@user/testing", "tool_b/0.2@user/testing"

Read more: :ref:`Tool requirements <build_requires>`

.. _exports_attribute:

exports
-------

This **optional attribute** declares the set of files that should be exported and stored side by
side with the *conanfile.py* file to make the recipe work: other python files that the recipe will
import, some text file with data to read,...

The ``exports`` field can declare one single file or pattern, or a list of any of the previous
elements. Patterns use `fnmatch <https://docs.python.org/3/library/fnmatch.html>`_
formatting to declare files to include or exclude.

For example, if we have some python code that we want the recipe to use in a ``helpers.py`` file,
and have some text file *info.txt* we want to read and display during the recipe evaluation
we would do something like:

.. code-block:: python

    exports = "helpers.py", "info.txt"

Exclude patterns are also possible, with the ``!`` prefix:

.. code-block:: python

    exports = "*.py", "!*tmp.py"


.. _exports_sources_attribute:

exports_sources
---------------

This **optional attribute** declares the set of files that should be exported together with the
recipe and will be available to generate the package. Unlike ``exports`` attribute, these files
shouldn't be used by the *conanfile.py* Python code, but to compile the library or generate
the final package. And, due to its purpose, these files will only be retrieved if requested
binaries are not available or the user forces Conan to compile from sources.

The ``exports_sources`` attribute can declare one single file or pattern, or a list of any of the
previous elements. Patterns use `fnmatch <https://docs.python.org/3/library/fnmatch.html>`_
formatting to declare files to include or exclude.

Together with the ``source()`` and ``imports()`` methods, and the :ref:`SCM feature<scm_feature>`,
this is another way to retrieve the sources to create a package. Unlike the other methods, files
declared in ``exports_sources`` will be exported together with the *conanfile.py* recipe, so,
if nothing else is required, it can create a self-contained package with all the sources
(like a snapshot) that will be used to generate the final artifacts.

Some examples for this attribute are:

.. code-block:: python

    exports_sources = "include*", "src*"

Exclude patterns are also possible, with the ``!`` prefix:

.. code-block:: python

    exports_sources = "include*", "src*", "!src/build/*"


Note, if the recipe defines the ``layout()`` method and specifies a ``self.folders.source = "src"`` it won't affect
where the files (from the ``exports_sources``) are copied. They will be copied to the base source folder. So, if you
want to replace some file that got into the ``source()`` method, you need to explicitly copy it from the parent folder
or even better, from ``self.export_sources_folder``.


.. code-block:: python

   import os, shutil
   from conan import ConanFile
   from conan.tools.files import save, load

   class Pkg(ConanFile):
      ...
      exports_sources = "CMakeLists.txt"

      def layout(self):
          self.folders.source = "src"
          self.folders.build = "build"

      def source(self):
          # emulate a download from web site
          save(self, "CMakeLists.txt", "MISTAKE: Very old CMakeLists to be replaced")
          # Now I fix it with one of the exported files
          shutil.copy("../CMakeLists.txt", ".")
          shutil.copy(os.path.join(self.export_sources_folder, "CMakeLists.txt", "."))



generators
----------

Generators specify which is the output of the :command:`install` command in your project folder. By default, a *conanbuildinfo.txt* file is
generated, but you can specify different generators and even use more than one.

.. code-block:: python

    class MyLibConan(ConanFile):
        generators = "cmake", "gcc"

You can also set the generators conditionally in the :ref:`configure() method<method_configure_config_options>`
like in the example below.

.. code-block:: python

    class MyLibConan(ConanFile):
        settings = "os", "compiler", "arch", "build_type"
        def configure(self):
            if self.settings.os == "Windows":
                self.generators = ["msbuild"]


Check the full :ref:`generators list<generators>`.

.. _attribute_build_stages:

should_configure, should_build, should_install, should_test
-----------------------------------------------------------

Read only variables defaulted to ``True``.

These variables allow you to control the build stages of a recipe during a :command:`conan build` command with the optional arguments
:command:`--configure`/:command:`--build`/:command:`--install`/:command:`--test`. For example, consider this ``build()`` method:

.. code-block:: python

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()
        cmake.install()
        cmake.test()

If nothing is specified, all four methods will be called. But using command line arguments, this can be changed:

.. code-block:: bash

    $ conan build . --configure  # only run cmake.configure(). Other methods will do nothing
    $ conan build . --build      # only run cmake.build(). Other methods will do nothing
    $ conan build . --install    # only run cmake.install(). Other methods will do nothing
    $ conan build . --test       # only run cmake.test(). Other methods will do nothing
    # They can be combined
    $ conan build . -c -b # run cmake.configure() + cmake.build(), but not cmake.install() nor cmake.test()

Autotools and Meson helpers already implement the same functionality. For other build systems, you can use these variables in the
``build()`` method:

.. code-block:: python

    def build(self):
        if self.should_configure:
            # Run my configure stage
        if self.should_build:
            # Run my build stage
        if self.should_install: # If my build has install, otherwise use package()
            # Run my install stage
        if self.should_test:
            # Run my test stage

Note that the ``should_configure``, ``should_build``, ``should_install``, ``should_test`` variables will always be ``True`` while building in
the cache and can be only modified for the local flow with :command:`conan build`.

build_policy
------------

With the ``build_policy`` attribute the package creator can change conan's build behavior.
The allowed ``build_policy`` values are:

- ``missing``: If this package is not found as a binary package, Conan will build it from source.
- ``always``: This package will always be built from source, also **retrieving the source code each time** by executing the "source" method.

.. code-block:: python
   :emphasize-lines: 2

    class PocoTimerConan(ConanFile):
        build_policy = "always" # "missing"

.. _short_paths_reference:

short_paths
-----------

This attribute is specific to Windows, and ignored on other operating systems.
It tells Conan to workaround the limitation of 260 chars in Windows paths.

.. important::

    Since Windows 10 (ver. 10.0.14393), it is possible to `enable long paths at the system level
    <https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file#maximum-path-length-limitation>`_.
    Latest python 2.x and 3.x installers enable this by default. With the path limit removed both on the OS
    and on Python, the ``short_paths`` functionality becomes unnecessary, and can be disabled explicitly
    through the ``CONAN_USER_HOME_SHORT`` environment variable.

Enabling short paths management will "link" the ``source`` and ``build`` directories of the package to a different
location, in Windows it will be ``C:\.conan\tmpdir``. All the folder layout in the local cache is maintained.

Set ``short_paths=True`` in your *conanfile.py*:

..  code-block:: python

    from conans import ConanFile

    class ConanFileTest(ConanFile):
        ...
        short_paths = True

.. seealso::

    There is an :ref:`environment variable <env_vars>` ``CONAN_USE_ALWAYS_SHORT_PATHS`` to force
    activate this behavior for all packages.


This behavior will also work in Cygwin, the short folder directory will be ``/home/<user>/.conan_short``
by default, but it can be modified as we've explained before.


.. _no_copy_source:

no_copy_source
--------------

The attribute ``no_copy_source`` tells the recipe that the source code will not be copied from the ``source`` folder to the ``build`` folder.
This is mostly an optimization for packages with large source codebases or header-only, to avoid extra copies. It is **mandatory** that the source code must not be modified at all by the configure or build scripts, as the source code will be shared among all builds.

To be able to use it, the package recipe can access the ``self.source_folder`` attribute, which will point to the ``build`` folder when ``no_copy_source=False`` or not defined, and will point to the ``source`` folder when ``no_copy_source=True``.

When this attribute is set to True, the ``self.copy()`` lines will be called twice, one copying from the ``source`` folder and the other copying from the ``build`` folder.

Read :ref:`header-only<header_only>` section for an example using ``no_copy_source`` attribute.


.. _folders_attributes_reference:

.. _attribute_source_folder:

source_folder
-------------

The folder in which the source code lives.

When a package is built in the Conan local cache its value is the same as the ``build`` folder by default. This is due to the fact that the
source code is copied from the ``source`` folder to the ``build`` folder to ensure isolation and avoiding modifications of shared common
source code among builds for different configurations. Only when ``no_copy_source=True`` this folder will actually point to the package
``source`` folder in the local cache.

When executing Conan commands in the :ref:`package_dev_flow` like :command:`conan source`, this attribute will be pointing to the folder
specified in the command line.

.. _attribute_install_folder:

install_folder
--------------

The folder in which the installation of packages outputs the generator files with the information of dependencies.
By default in the the local cache its value is the same as ``self.build_folder`` one.

When executing Conan commands in the :ref:`package_dev_flow` like :command:`conan install` or :command:`conan build`, this attribute will
be pointing to the folder specified in the command line.

.. _attribute_build_folder:

build_folder
------------

The folder used to build the source code. In the local cache a build folder is created with the name of the package ID that will be built.

When executing Conan commands in the :ref:`package_dev_flow` like :command:`conan build`, this attribute will be pointing to the folder
specified in the command line.

.. _attribute_package_folder:

package_folder
--------------

The folder to copy the final artifacts for the binary package. In the local cache a package folder is created for every different package
ID.

When executing Conan commands in the :ref:`package_dev_flow` like :command:`conan package`, this attribute will be pointing to the folder
specified in the command line.

.. _attribute_recipe_folder:

recipe_folder
-------------

Available since: `1.28.0 <https://github.com/conan-io/conan/releases/tag/1.28.0>`_

The folder where the recipe *conanfile.py* is stored, either in the local folder or in the cache. This is useful in order to access files
that are exported along with the recipe.

.. _cpp_info_attributes_reference:

cpp_info
--------

.. important::

    This attribute is only defined inside ``package_info()`` method being `None` elsewhere.

The ``self.cpp_info`` attribute is responsible for storing all the information needed by consumers of a package: include directories,
library names, library paths... There are some default values that will be applied automatically if not indicated otherwise.

This object should be filled in ``package_info()`` method.

+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| NAME                                 | DESCRIPTION                                                                                                          |
+======================================+======================================================================================================================+
| self.cpp_info.includedirs            | Ordered list with include paths. Defaulted to ``["include"]``                                                        |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.libdirs                | Ordered list with lib paths. Defaulted to ``["lib"]``                                                                |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.resdirs                | Ordered list of resource (data) paths. Defaulted to ``["res"]``. If layout() is declared, defaulted to ``[]`` \*     |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.bindirs                | Ordered list with paths to binaries (executables, dynamic libraries,...). Defaulted to ``["bin"]``                   |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.builddirs              | | Ordered list with build scripts directory paths. Defaulted to ``[""]`` (Package folder directory)                  |
|                                      | | CMake generators will search in these dirs for files like *findXXX.cmake*                                          |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.libs                   | Ordered list with the library names, Defaulted to ``[]`` (empty)                                                     |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.defines                | Preprocessor definitions. Defaulted to ``[]`` (empty)                                                                |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.cflags                 | Ordered list with pure C flags. Defaulted to ``[]`` (empty)                                                          |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.cppflags               | [DEPRECATED: Use cxxflags instead]                                                                                   |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.cxxflags               | Ordered list with C++ flags. Defaulted to ``[]`` (empty)                                                             |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.sharedlinkflags        | Ordered list with linker flags (shared libs). Defaulted to ``[]`` (empty)                                            |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.exelinkflags           | Ordered list with linker flags (executables). Defaulted to ``[]`` (empty)                                            |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.frameworks             | Ordered list with the framework names (OSX), Defaulted to ``[]`` (empty)                                             |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.frameworkdirs          | Ordered list with frameworks search paths (OSX). Defaulted to ``["Frameworks"]`` or ``[]`` if layout() is declared \*|
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.rootpath               | Filled with the root directory of the package, see ``deps_cpp_info``                                                 |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.name                   | **[DEPRECATED]** Use ``self.cpp_info.names`` instead.                                                                |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.names["generator"]     | | Alternative name for the package used by an specific generator to create files or variables.                       |
|                                      | | If set for a generator it will override the information provided by self.cpp_info.name.                            |
|                                      | | Like the cpp_info.name, this is only supported by `cmake`, `cmake_multi`, `cmake_find_package`,                    |
|                                      | | `cmake_find_package_multi`, `cmake_paths` and `pkg_config` generators.                                             |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.filenames["generator"] | | Alternative name for the filename produced by a specific generator. If set for a generator it will                 |
|                                      | | override the "names" value. This is only supported by the `cmake_find_package` and                                 |
|                                      | | `cmake_find_package_multi` generators.                                                                             |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.system_libs            | Ordered list with the system library names. Defaulted to ``[]`` (empty)                                              |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.build_modules          | | Dictionary of lists per generator containing relative paths to build system related utility module                 |
|                                      | | files created by the package. Used by CMake generators to export *.cmake* files with functions for                 |
|                                      | | consumers. Defaulted to ``{}`` (empty)                                                                             |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.components             | | **[Experimental]** Dictionary with different components a package may have: libraries, executables...              |
|                                      | | **Warning**: Using components with other ``cpp_info`` non-default values or configs is not supported               |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.requires               | | **[Experimental]** List of components to consume from requirements (it applies only to                             |
|                                      | | generators that implements components feature).                                                                    |
|                                      | | **Warning**: If declared, only the components listed here will used by the linker and consumers.                   |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+
| self.cpp_info.objects                | | **[Experimental]** List of object libraries (*.obj* or *.o*). Defaulted to ``[]`` (empty)                          |
|                                      | | Only supported by new :ref:`CMakeDeps<conan_tools_cmake>` generator                                                |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+

.. note::

   (*) If the ``layout()`` is declared, for Conan >= 1.50, the default values for ``self.cpp_info.frameworkdirs`` and ``self.cpp_info.resdirs``
   change to ``[]``, being a way to migrate to Conan 2.0, where the default values for these field changed. Read more in the
   :ref:`migration guide<conan2_migration_guide_recipes_package_info>`.


The paths of the directories in the directory variables indicated above are relative to the
:ref:`self.package_folder<folders_attributes_reference>` directory.

.. warning::

    Components is a **experimental** feature subject to breaking changes in future releases.

:ref:`Using components <package_information_components>` you can achieve a more fine-grained control over individual libraries available in
a single Conan package. Components allow you define a ``cpp_info`` like object per each of those libraries and also requirements between
them and to components of other packages (the following case is not a real example):

.. code-block:: python

    def package_info(self):
        self.cpp_info.names["cmake_find_package"] = "OpenSSL"
        self.cpp_info.names["cmake_find_package_multi"] = "OpenSSL"
        self.cpp_info.components["crypto"].names["cmake_find_package"] = "Crypto"
        self.cpp_info.components["crypto"].libs = ["libcrypto"]
        self.cpp_info.components["crypto"].defines = ["DEFINE_CRYPTO=1"]
        self.cpp_info.components["crypto"].requires = ["zlib::zlib"]  # Depends on all components in zlib package

        self.cpp_info.components["ssl"].names["cmake"] = "SSL"
        self.cpp_info.components["ssl"].includedirs = ["include/headers_ssl"]
        self.cpp_info.components["ssl"].libs = ["libssl"]
        self.cpp_info.components["ssl"].requires = ["crypto",
                                                    "boost::headers"]  # Depends on headers component in boost package
        self.cpp_info.components["ssl"].names["cmake"] = "SSL"

        obj_ext = "obj" if platform.system() == "Windows" else "o"
        self.cpp_info.components["ssl-objs"].objects = [os.path.join("lib", "ssl-object.{}".format(obj_ext))]

The interface of the ``Component`` object is the same as the one used by the ``cpp_info`` object and
has **the same default directories**.

.. warning::

    Using components and global ``cpp_info`` non-default values or release/debug configurations at the same time is not allowed (except for
    ``self.cpp_info.names``).

Dependencies among components and to components of other requirements can be defined using the ``requires`` attribute and the name
of the component. The dependency graph for components will be calculated and values will be aggregated in the correct order for each field.

**New properties model for the cpp_info new tools**

.. warning::

    Using ``set_property`` and ``get_property`` methods for ``cpp_info`` is an **experimental**
    feature subject to breaking changes in future releases.

Using ``.names``, ``.filenames`` and ``.build_modules`` will not work any more for new
generators, like :ref:`CMakeDeps<CMakeDeps>` and :ref:`PkgConfigDeps<PkgConfigDeps>`.
They have a new way of setting this information using ``set_property`` and
``get_property`` methods of the ``cpp_info`` object (available since Conan 1.36).

.. code-block:: python

    def set_property(self, property_name, value)
    def get_property(self, property_name):

To read more about the new ``set_property`` and ``get_property`` methods for ``cpp_info``
please check the dedicated section in the :ref:`Conan 2.0 migration guide <conanv2_properties_model>`.


.. _deps_cpp_info_attributes_reference:

deps_cpp_info
-------------

Contains the ``cpp_info`` object of the requirements of the recipe. In addition of the above fields, there are also properties to obtain the
absolute paths, and ``name`` and ``version`` attributes:

+---------------------------------------------------+---------------------------------------------------------------------+
| NAME                                              | DESCRIPTION                                                         |
+===================================================+=====================================================================+
| self.deps_cpp_info["dep"].include_paths           | "dep" package ``includedirs`` but transformed to absolute paths     |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].lib_paths               | "dep" package ``libdirs`` but transformed to absolute paths         |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].bin_paths               | "dep" package ``bindirs`` but transformed to absolute paths         |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].build_paths             | "dep" package ``builddirs`` but transformed to absolute paths       |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].res_paths               | "dep" package ``resdirs`` but transformed to absolute paths         |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].framework_paths         | "dep" package  ``frameworkdirs`` but transformed to absolute paths  |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].build_modules_paths     | "dep" package ``build_modules`` but transformed to absolute paths   |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].get_name("<generator>") | Get the name declared for the given generator                       |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].version                 | Get the version of the "dep" package                                |
+---------------------------------------------------+---------------------------------------------------------------------+
| self.deps_cpp_info["dep"].components              | | **[Experimental]** Dictionary with different components a package |
|                                                   | | may have: libraries, executables...                               |
+---------------------------------------------------+---------------------------------------------------------------------+

To get a list of all the dependency names from ``deps_cpp_info``, you can call the `deps` member:

.. code-block:: python

    class PocoTimerConan(ConanFile):
        ...
        def build(self):
            # deps is a list of package names: ["poco", "zlib", "openssl"]
            deps = self.deps_cpp_info.deps

It can be used to get information about the dependencies, like used compilation flags or the
root folder of the package:

.. code-block:: python
   :emphasize-lines: 8, 11, 14

    class PocoTimerConan(ConanFile):
        ...
        requires = "zlib/1.2.11", "openssl/1.0.2u"
        ...

        def build(self):
            # Get the directory where zlib package is installed
            self.deps_cpp_info["zlib"].rootpath

            # Get the absolute paths to zlib include directories (list)
            self.deps_cpp_info["zlib"].include_paths

            # Get the sharedlinkflags property from OpenSSL package
            self.deps_cpp_info["openssl"].sharedlinkflags


.. note::

    If using the experimental feature :ref:`with different context for host and build <build_requires_context>`, this
    attribute will contain only information from packages in the *host* context.


.. _env_info_attributes_reference:

env_info
--------

This attribute is only defined inside ``package_info()`` method, being None elsewhere, so please use it only inside this method.

The ``self.env_info`` object can be filled with the environment variables to be declared in the packages reusing the recipe.

.. seealso::

    Read :ref:`package_info() method docs <method_package_info>` for more info.

.. _deps_env_info_attributes_reference:

deps_env_info
-------------

You can access to the declared environment variables of the requirements of the recipe.

**Note:** The environment variables declared in the requirements of a recipe are automatically applied
and it can be accessed with the python ``os.environ`` dictionary. Nevertheless if
you want to access to the variable declared by some specific requirement you can use the ``self.deps_env_info`` object.

.. code-block:: python
   :emphasize-lines: 10

    import os

    class RecipeConan(ConanFile):
        ...
        requires = "package1/1.0@conan/stable", "package2/1.2@conan/stable"
        ...

        def build(self):
            # Get the SOMEVAR environment variable declared in the "package1"
            self.deps_env_info["package1"].SOMEVAR

            # Access to the environment variables globally
            os.environ["SOMEVAR"]


.. note::

    If using the experimental feature :ref:`with different context for host and build <build_requires_context>`, this
    attribute will contain only information from packages in the *build* context.


user_info
---------

This attribute is only defined inside ``package_info()`` method, being None elsewhere, so please use it only inside this method.

The ``self.user_info`` object can be filled with any custom variable to be accessed in the packages reusing the recipe.

.. seealso::

    Read :ref:`package_info() method docs <method_package_info>` for more info.

.. _deps_user_info_attributes_reference:

deps_user_info
--------------

You can access the declared ``user_info.XXX`` variables of the requirements through the ``self.deps_user_info`` object like this:


.. code-block:: python
   :emphasize-lines: 9

    import os

    class RecipeConan(ConanFile):
        ...
        requires = "package1/1.0@conan/stable"
        ...

        def build(self):
            self.deps_user_info["package1"].SOMEVAR


.. note::

    If using the experimental feature :ref:`with different context for host and build <build_requires_context>`, this
    attribute will contain only information from packages in the *host* context. Use :ref:`user_info_build_attributes_reference`
    to access information from packages that belong to *build* context.


.. _user_info_build_attributes_reference:

user_info_build
---------------

.. warning::

    This section refers to the **experimental feature** that is activated when using ``--profile:build`` and ``--profile:host``
    in the command-line. It is currently under development, features can be added or removed in the following versions.


This attribute offers the information declared in the ``user_info.XXXX`` variables of the requirements that belong to the *build*
context, it is available only if Conan is invoked with two profiles (see :ref:`this section <build_requires_context>` to
know more about this feature.

.. code-block:: python
   :emphasize-lines: 9

    import os

    class RecipeConan(ConanFile):
        ...
        tool_requires = "tool/1.0"
        ...

        def build(self):
            self.user_info_build["tool"].SOMEVAR


info
----

Object used to control the unique ID for a package. Check the :ref:`package_id() <method_package_id>` to see the details of the ``self.info``
object.


.. _apply_env:

apply_env
---------

When ``True`` (Default), the values from ``self.deps_env_info`` (corresponding to the declared ``env_info`` in the ``requires`` and ``tool_requires``)
will be automatically applied to the ``os.environ``.

Disable it setting ``apply_env`` to False if you want to control by yourself the environment variables
applied to your recipes.

You can apply manually the environment variables from the requires and tool_requires:

.. code-block:: python
   :emphasize-lines: 2

    import os
    from conans import tools

    class RecipeConan(ConanFile):
        apply_env = False

        def build(self):
            with tools.environment_append(self.env):
                # The same if we specified apply_env = True
                pass

.. _in_local_cache:

in_local_cache
--------------

A boolean attribute useful for conditional logic to apply in user folders local commands.
It will return `True` if the conanfile resides in the local cache ( we are installing the package)
and `False` if we are running the conanfile in a user folder (local Conan commands).

.. code-block:: python

    import os

    class RecipeConan(ConanFile):
        ...

        def build(self):
            if self.in_local_cache:
                # we are installing the package
            else:
                # we are building the package in a local directory


.. _develop_attribute:

develop
-------

A boolean attribute useful for conditional logic. It will be ``True`` if the package is created with :command:`conan create`, or if the
*conanfile.py* is in user space:

.. code-block:: python

    class RecipeConan(ConanFile):

        def build(self):
            if self.develop:
                self.output.info("Develop mode")

It can be used for conditional logic in other methods too, like ``requirements()``, ``package()``, etc.

This recipe will output "Develop mode" if:

.. code-block:: bash

    $ conan create . user/testing
    # or
    $ mkdir build && cd build && conan install ..
    $ conan build ..

But it will not output that when it is a transitive requirement or installed with :command:`conan install`.

.. _keep_imports:

keep_imports
------------

Just before the ``build()`` method is executed, if the conanfile has an ``imports()`` method, it is
executed into the build folder, to copy binaries from dependencies that might be necessary for
the ``build()`` method to work. After the method finishes, those copied (imported) files are removed,
so they are not later unnecessarily repackaged.

This behavior can be avoided declaring the ``keep_imports=True`` attribute. This can be useful, for example
to :ref:`repackage artifacts <repackage>`


.. _scm_attribute:

scm
---

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases. Although this
    is an experimental feature, the use of the feature using ``scm_to_conandata`` is considered
    stable.

Used to clone/checkout a repository. It is a dictionary with the following possible values:

.. code-block:: python

    from conans import ConanFile, CMake, tools

    class HelloConan(ConanFile):
         scm = {
            "type": "git",
            "subfolder": "hello",
            "url": "https://github.com/conan-io/hello.git",
            "revision": "master"
         }
        ...


- **type** (Required): Currently only ``git`` and ``svn`` are supported. Others can be added eventually.
- **url** (Required): URL of the remote or ``auto`` to capture the remote from the local working
  copy (credentials will be removed from it). When type is ``svn`` it can contain
  the `peg_revision <http://svnbook.red-bean.com/en/1.7/svn.advanced.pegrevs.html>`_.
- **revision** (Required): id of the revision or ``auto`` to capture the current working copy one.
  When type is ``git``, it can also be the branch name or a tag.
- **subfolder** (Optional, Defaulted to ``.``): A subfolder where the repository will be cloned.
- **username** (Optional, Defaulted to ``None``): When present, it will be used as the login to authenticate with the remote.
- **password** (Optional, Defaulted to ``None``): When present, it will be used as the password to authenticate with the remote.
- **verify_ssl** (Optional, Defaulted to ``True``): Verify SSL certificate of the specified **url**.
- **shallow** (Optional, Defaulted to ``True``): Use shallow clone for Git repositories.
- **submodule** (Optional, Defaulted to ``None``):
   - ``shallow``: Will sync the git submodules using ``submodule sync``
   - ``recursive``: Will sync the git submodules using ``submodule sync --recursive``

Attributes ``type``, ``url`` and ``revision`` are required to upload the recipe to a remote server.

SCM attributes are evaluated in the working directory where the *conanfile.py* is located before
exporting it to the Conan cache, so these values can be returned from arbitrary functions that
depend on the local directory. Nevertheless, all the other code in the recipe must be able to
run in the export folder inside the cache, where it has access only to the files exported (see
attribute :ref:`exports <exports_attribute>` and :ref:`conandata_yml`) and to any other functionality
from a :ref:`python_requires <python_requires>` package.

.. warning::

    By default, in Conan v1.x the information after evaluating the attribute ``scm`` will be stored in the
    *conanfile.py* file (the recipe will be modified when exported to the Conan cache) and any value will be
    written in plain text (watch out about passwords).
    However, you can activate the :ref:`scm_to_conandata<conan_conf>` config option, the *conanfile.py*
    won't be modified (data is stored in a different file) and the fields ``username`` and ``password`` won't be
    stored, so these one will be computed each time the recipe is loaded.

.. note::

    In case of git, by default conan will try to perform shallow clone of the repository, and will fallback to the full
    clone in case shallow fails (e.g. not supported by the server).

To know more about the usage of ``scm`` check:

- :ref:`Creating packages/Recipe and sources in a different repo <external_repo>`
- :ref:`Creating packages/Recipe and sources in the same repo <package_repo>`


.. _revision_mode_attribute:

revision_mode
-------------

.. warning::

    This attribute is part of the :ref:`package revisions<package_revisions>` feature, so
    it is also an **experimental** feature subject to breaking changes in future releases.

This attribute allow each recipe to declare how the revision for the recipe itself should
be computed. It can take two different values:

 - ``"hash"`` (by default): Conan will use the checksum hash of the recipe manifest to
   compute the revision for the recipe.
 - ``"scm"``: the commit ID will be used as the recipe revision if it belongs to a known
   repository system (Git or SVN). If there is no repository it will raise an error.


.. _python_requires_attribute:

python_requires (legacy)
------------------------

.. warning::

    This attribute has been superseded by the new :ref:`python_requires`. Even if this is an **experimental**
    feature subject to breaking changes in future releases, this legacy ``python_requires`` syntax has not
    been removed yet, but it will be removed in Conan 2.0.

Python requires are associated with the ``ConanFile`` declared in the recipe file, data
from those imported recipes is accessible using the ``python_requires`` attribute in
the recipe itself. This attribute is a dictionary where the key is the name of the
*python requires* reference and the value is a dictionary with the following information:

 - ``ref``: full reference of the python requires.
 - ``exports_folder``: directory in the cache where the exported files are located.
 - ``exports_sources_folder``: directory in the cache where the files exported using the
   ``exports_sources`` attribute of the python requires recipe are located.

You can use this information to copy files accompanying a python requires to the consumer
workspace.:

.. code-block:: python

    from conans import ConanFile

    class PyReq(ConanFile):
        name = "pyreq"
        exports_sources = "CMakeLists.txt"

        def source(self):
            pyreq = self.python_requires['pyreq']
            path = os.path.join(pyreq.exports_sources_folder, "CMakeLists.txt")
            shutil.copy(src=path, dst=self.source_folder)

python_requires
---------------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

This class attribute allows to define a dependency to another Conan recipe and reuse its code.
Its basic syntax is:

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        python_requires = "pyreq/0.1@user/channel"  # recipe to reuse code from

        def build(self):
            self.python_requires["pyreq"].module # access to the whole conanfile.py module
            self.python_requires["pyreq"].module.myvar  # access to a variable
            self.python_requires["pyreq"].module.myfunct()  # access to a global function
            self.python_requires["pyreq"].path # access to the folder where the reused file is


Read more about this attribute in :ref:`python_requires`


python_requires_extend
----------------------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.


This class attribute defines one or more classes that will be injected in runtime as base classes of
the recipe class. Syntax for each of these classes should be a string like ``pyreq.MyConanfileBase``
where the ``pyreq`` is the name of a ``python_requires`` and ``MyConanfileBase`` is the name of the class
to use.

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        python_requires = "pyreq/0.1@user/channel", "utils/0.1@user/channel"
        python_requires_extend = "pyreq.MyConanfileBase", "utils.UtilsBase"  # class/es to inject


Read more about this attribute in :ref:`python_requires`


.. _conandata_attribute:

conan_data
----------

This attribute is a dictionary with the keys and values provided in a :ref:`conandata_yml` file format placed next to the *conanfile.py*.
This YAML file is automatically exported with the recipe and automatically loaded with it too.

You can declare information in the *conandata.yml* file and then access it inside any of the methods of the recipe.
For example, a *conandata.yml* with information about sources that looks like this:

.. code-block:: YAML

    sources:
      "1.1.0":
        url: "https://www.url.org/source/mylib-1.0.0.tar.gz"
        sha256: "8c48baf3babe0d505d16cfc0cf272589c66d3624264098213db0fb00034728e9"
      "1.1.1":
        url: "https://www.url.org/source/mylib-1.0.1.tar.gz"
        sha256: "15b6393c20030aab02c8e2fe0243cb1d1d18062f6c095d67bca91871dc7f324a"

.. code-block:: python

    def source(self):
        tools.get(**self.conan_data["sources"][self.version])


deprecated
----------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.28.0 <https://github.com/conan-io/conan/releases/tag/1.28.0>`_

This attribute declares that the recipe is deprecated, causing a user-friendly warning message to be emitted whenever it is used.
For example, the following code:

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        name = "cpp-taskflow"
        version = "1.0"
        deprecated = True

may emit a warning like:

.. code-block:: bash

    cpp-taskflow/1.0: WARN: Recipe 'cpp-taskflow/1.0' is deprecated. Please, consider changing your requirements.

Optionally, the attribute may specify the name of the suggested replacement:

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        name = "cpp-taskflow"
        version = "1.0"
        deprecated = "taskflow"

This will emit a warning like:

.. code-block:: bash

    cpp-taskflow/1.0: WARN: Recipe 'cpp-taskflow/1.0' is deprecated in favor of 'taskflow'. Please, consider changing your requirements.

If the value of the attribute evaluates to ``False``, no warning is printed.


provides
--------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.28.0 <https://github.com/conan-io/conan/releases/tag/1.28.0>`_

This attribute declares that the recipe provides the same functionality as other recipe(s). The attribute is usually needed if two or more
libraries implement the same API to prevent link-time and run-time conflicts (ODR violations). One typical situation is forked libraries.

Some examples are:

 - `LibreSSL <https://www.libressl.org/>`__, `BoringSSL <https://boringssl.googlesource.com/boringssl/>`__ and `OpenSSL <https://www.openssl.org/>`__
 - `libav <https://en.wikipedia.org/wiki/Libav>`__ and `ffmpeg <https://ffmpeg.org/>`__
 - `MariaDB client <https://downloads.mariadb.org/client-native>`__ and `MySQL client <https://dev.mysql.com/downloads/c-api/>`__


If Conan encounters two or more libraries providing the same functionality within a single graph, it raises an error:

.. code-block:: bash

    At least two recipes provides the same functionality:
     - 'libjpeg' provided by 'libjpeg/9d', 'libjpeg-turbo/2.0.5'

The attribute value should be a string with a recipe name or a tuple of such recipe names.

For example, to declare that ``libjpeg-turbo`` recipe offers the same functionality as ``libjpeg`` recipe, the following code could be used:

.. code-block:: python

    from conans import ConanFile

    class LibJpegTurbo(ConanFile):
        name = "libjpeg-turbo"
        version = "1.0"
        provides = "libjpeg"


To declare that a recipe provides the functionality of several different recipes at the same time, the following code could be used:

.. code-block:: python

    from conans import ConanFile

    class OpenBLAS(ConanFile):
        name = "openblas"
        version = "1.0"
        provides = "cblas", "lapack"

If the attribute is omitted, the value of the attribute is assumed to be equal to the current package name. Thus, it's redundant for
``libjpeg`` recipe to declare that it provides ``libjpeg``, it's already implicitly assumed by Conan.

conf
----

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.47.0 <https://github.com/conan-io/conan/releases/tag/1.47.0>`_

In the ``self.conf`` attribute we can find all the conf entries declared in the :ref:`[conf] section of the profiles<profiles_tools_conf>`
in addition of the declared :ref:`self.conf_info<conf_in_recipes>` entries from the first level tool requirements. The profile entries have priority.

This is an example of a recipe for a tool called ``my_android_ndk`` with the following recipe:

.. code-block:: python

    from conan import ConanFile

    class MyAndroidNDKRecipe(ConanFile):

      name="my_android_ndk"
      version="1.0"

      def package_info(self):
          self.conf_info.define("tools.android:ndk_path", "bar")

If we require that tool:

.. code-block:: python

    from conan import ConanFile

    class MyConsumer(ConanFile):

      tool_requires = "my_android_ndk/1.0"

      def generate(self):
          self.output.info("NDK host: %s" % self.conf.get("tools.android:ndk_path"))
          self.output.info("Custom var1: %s" % self.conf.get("user.custom.var1"))


And we install it applying this profile:

.. code-block:: text
   :caption: myprofile

   include(default)
   [conf]
   tools.android:ndk_path=foo
   user.custom.var1=hello

.. code-block:: shell

   $ conan install . --profile myprofile
   ...
   conanfile.py: Applying build-requirement: my_android_ndk/1.0
   conanfile.py: Generator txt created conanbuildinfo.txt
   conanfile.py: Calling generate()
   conanfile.py: NDK host: foo
   conanfile.py: Custom var1: hello
   ...

Without the profile:

.. code-block:: shell

   $ conan install .
   ...
   conanfile.py: Applying build-requirement: my_android_ndk/1.0
   conanfile.py: Generator txt created conanbuildinfo.txt
   conanfile.py: Calling generate()
   conanfile.py: NDK host: bar
   conanfile.py: Custom var1: None


win_bash
--------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.39.0 <https://github.com/conan-io/conan/releases/tag/1.39.0>`_

When ``True`` it enables the new run in a subsystem bash in Windows mechanism. :ref:`Read more here<conanfile_win_bash>`.

.. code-block:: python

    from conans import ConanFile

    class FooRecipe(ConanFile):
        ...
        win_bash = True

.. _test_type:

test_type
---------

.. warning::

    Test type is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.44.0 <https://github.com/conan-io/conan/releases/tag/1.44.0>`_

This attribute allows testing requirements and build requiments explicitly on test package.
In Conan 2.0 the ``test_type`` attribute will be ignored, the behavior will be always explicit, so declaring ``test_type="explicit"`` will make the test recipe compatible with Conan 2.0.
The possible values are:

 - ``requires`` (default): The package being tested will be consumed as a regular requirement automatically.
 - ``build_requires``: The package being tested will be consumed as a build requirement automatically. It can be combined with ``requires`` to be required in both ways.
 - ``explicit``: The test package will not solve its dependencies automatically, you need to declare it explicitly using the reference at ``self.tested_reference_str``. This will be the only behavior for Conan 2.0. The additional values ``requires`` and ``build_requires`` (if specified) will be ignored.

To solve build requirements and requirements automatically as regularly on Conan 1.0

 .. code-block:: python

    from conans import ConanFile, CMake
    import os

    class TestPackage(ConanFile):
        test_type = "build_requires", "requires"

        def build(self):
            cmake = CMake(self)
            cmake.configure()
            cmake.build()

        def test(self):
            bin_path = os.path.join("bin", "test_package")
            self.run(bin_path, run_environment=True)

To explicitly declare the required dependencies as required on Conan 2.0:

 .. code-block:: python

    from conans import ConanFile, CMake
    import os

    class TestPackage(ConanFile):
        test_type = "explicit"

        def requirements(self):
            self.requires(self.tested_reference_str)
            # and, or
            self.build_requires(self.tested_reference_str)

        def build(self):
            cmake = CMake(self)
            cmake.configure()
            cmake.build()

        def test(self):
            bin_path = os.path.join("bin", "test_package")
            self.run(bin_path, run_environment=True)


For more information see :ref:`explicit test package requirement <explicit_test_package_requirement>` and :ref:`testing tool requirements <testing_build_requires>`.
