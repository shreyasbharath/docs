.. _methods:

Methods
=======

.. _method_source:

source()
--------

Method used to retrieve the source code from any other external origin like github using ``$ git clone`` or just a regular download.

For example, "exporting" the source code files, together with the *conanfile.py* file, can be handy if the source code is not under version
control. But if the source code is available in a repository, you can directly get it from there:

.. code-block:: python

    from conans import ConanFile

    class HelloConan(ConanFile):
        name = "hello"
        version = "0.1"
        settings = "os", "compiler", "build_type", "arch"

        def source(self):
            self.run("git clone https://github.com/conan-io/hello.git")
            # You can also change branch, commit or whatever
            # self.run("cd hello && git checkout 2fe5...")
            #
            # Or using the Git class:
            # git = tools.Git(folder="hello")
            # git.clone("https://github.com/conan-io/hello.git")


The current working directory where the ``source()`` method runs is the ``self.source_folder``. Note, however, that this folder
can be different if the recipe defines the ``layout()`` method and specifies a ``self.folders.source = "src"``. In that case, the
``self.source_folder`` and the current working directory will be the composition of the base folder (typically where the recipe is)
and the user specified ``"src"`` subfolder.

This will work, as long as git is in your current path (so in Win you probably want to run things in msysgit, cmder, etc). You can also use
another VCS or direct download/unzip. For that purpose, we have provided some helpers, but you can use your own code or origin as well. This
is a snippet of the conanfile of the Poco library:

..  code-block:: python

    from conans import ConanFile
    from conans.tools import download, unzip, check_md5, check_sha1, check_sha256
    import os
    import shutil

    class PocoConan(ConanFile):
        name = "poco"
        version = "1.6.0"

        def source(self):
            zip_name = "poco-1.6.0-release.zip"
            download("https://github.com/pocoproject/poco/archive/poco-1.6.0-release.zip", zip_name)
            # check_md5(zip_name, "51e11f2c02a36689d6ed655b6fff9ec9")
            # check_sha1(zip_name, "8d87812ce591ced8ce3a022beec1df1c8b2fac87")
            # check_sha256(zip_name, "653f983c30974d292de58444626884bee84a2731989ff5a336b93a0fef168d79")
            unzip(zip_name)
            shutil.move("poco-poco-1.6.0-release", "poco")
            os.unlink(zip_name)

The download, unzip utilities can be imported from conan, but you can also use your own code here
to retrieve source code from any origin. You can even create packages for pre-compiled libraries
you already have, even if you don't have the source code. You can download the binaries, skip
the ``build()`` method and define your ``package()`` and ``package_info()`` accordingly.

You can also use ``check_md5()``, ``check_sha1()`` and ``check_sha256()`` from the :ref:`tools <tools_check_with_algorithm_sum>` module to
verify that a package is downloaded correctly.

.. note::

    It is very important to recall that the ``source()`` method will be executed just once, and the source code will be shared for all the
    package builds. So it is not a good idea to conditionally use settings or options to make changes or patches on the source code. Maybe
    the only setting that makes sense is the OS ``self.settings.os``, if not doing cross-building, for example to retrieve different
    sources:

    .. code-block:: python

            def source(self):
                if platform.system() == "Windows":
                    # download some Win source zip
                else:
                    # download sources from Nix systems in a tgz

    If you need to patch the source code or build scripts differently for different variants of your packages, you can do it in the
    ``build()`` method, which uses a different folder and source code copy for each variant.

    .. code-block:: python

            def build(self):
                tools.patch(patch_file="0001-fix.patch")

build()
-------

This method is used to build the source code of the recipe using the desired commands. You can use your command line tools to invoke your
build system or any of the build helpers provided with Conan.

.. code-block:: python

    def build(self):
        cmake = CMake(self)
        self.run("cmake . %s" % (cmake.command_line))
        self.run("cmake --build . %s" % cmake.build_config)

Build helpers
+++++++++++++

You can use these classes to prepare your build system's command invocation:

- **CMake**: Prepares the invocation of cmake command with your settings.
- **AutoToolsBuildEnvironment**: If you are using configure/Makefile to build your project you can use this helper. Read more:
  :ref:`Building with Autotools <autotools_reference>`.
- **MSBuild**: If you are using Visual Studio compiler directly to build your project you can use this helper :ref:`MSBuild() <msbuild>`.
  For lower level control, the **VisualStudioBuildEnvironment** can also be used: :ref:`VisualStudioBuildEnvironment <visual_studio_build>`.

(Unit) Testing your library
+++++++++++++++++++++++++++

We have seen how to run package tests with conan, but what if we want to run full unit tests on
our library before packaging, so that they are run for every build configuration?
Nothing special is required here. We can just launch the tests from the last command in our
``build()`` method:

.. code-block:: python

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()
        # here you can run CTest, launch your binaries, etc
        cmake.test()

.. _method_package:

package()
---------

The actual creation of the package, once that it is built, is done in the ``package()`` method. Using the ``self.copy()`` method, artifacts
are copied from the build folder to the package folder.

The syntax of ``self.copy`` inside ``package()`` is as follows:

.. code-block:: python

    self.copy(pattern, dst="", src="", keep_path=True, symlinks=None, excludes=None, ignore_case=True)

Returns: A list with absolute paths of the files copied in the destination folder.

Parameters:
    - **pattern** (Required): A pattern following fnmatch syntax of the files you want to copy, from the build to the package folders.
      Typically something like ``*.lib`` or ``*.h``.
    - **src** (Optional, Defaulted to ``""``): The folder where you want to search the files in the build folder. If you know that your
      libraries when you build your package will be in *build/lib*, you will typically use ``build/lib`` in this parameter. Leaving it empty
      means the root build folder in local cache.
    - **dst** (Optional, Defaulted to ``""``): Destination folder in the package. They will typically be ``include`` for headers, ``lib``
      for libraries and so on, though you can use any convention you like. Leaving it empty means the root package folder in local cache.
    - **keep_path** (Optional, Defaulted to ``True``): Means if you want to keep the relative path when you copy the files from the **src**
      folder to the **dst** one. Typically headers are packaged with relative path.
    - **symlinks** (Optional, Defaulted to ``None``): Set it to True to activate symlink copying, like typical lib.so->lib.so.9.
    - **excludes** (Optional, Defaulted to ``None``): Single pattern or a tuple of patterns to be excluded from the copy. If a file matches
      both the include and the exclude pattern, it will be excluded.
    - **ignore_case** (Optional, Defaulted to ``True``): If enabled, it will do a case-insensitive pattern matching.

For example:

.. code-block:: python

    self.copy("*.h", "include", "build/include") #keep_path default is True

The final path in the package will be: ``include/mylib/path/header.h``, and as the *include* is usually added to the path, the includes
will be in the form: ``#include "mylib/path/header.h"`` which is something desired.

``keep_path=False`` is something typically desired for libraries, both static and dynamic. Some compilers as MSVC, put them in paths as
*Debug/x64/MyLib/Mylib.lib*. Using this option, we could write:

.. code-block:: python

    self.copy("*.lib", "lib", "", keep_path=False)

And it will copy the lib to the package folder *lib/Mylib.lib*, which can be linked easily.

.. note::

    If you are using CMake and you have an install target defined in your CMakeLists.txt, you might be able to reuse it for this
    ``package()`` method. Please check :ref:`reuse_cmake_install`.

This method copies files from build/source folder to the package folder depending on two situations:

- **Build folder and source folder are the same**: Normally during :command:`conan create` source folder content is copied to the build
  folder. In this situation ``src`` parameter of ``self.copy()`` will be relative to the build folder in the local cache.

- **Build folder is different from source folder**: When :ref:`developing a package recipe<package_dev_flow>` and source and build folder
  are different (:command:`conan package . --source-folder=source --build-folder=build`) or when :ref:`no_copy_source` is defined,
  every ``self.copy()`` is internally called twice: One will copy from the source folder (``src`` parameter of ``self.copy()`` will point to the
  source folder), and the other will copy from the build folder (``src`` parameter of ``self.copy()`` will point to the build folder).

.. _method_package_info:

package_info()
--------------

cpp_info
++++++++

Each package has to specify certain build information for its consumers. This can be done in the ``cpp_info`` attribute within the
``package_info()`` method.

The :ref:`cpp_info_attributes_reference` attribute has the following properties you can assign/append to:

.. code-block:: python

    self.cpp_info.names["generator_name"] = "<PKG_NAME>"
    self.cpp_info.includedirs = ['include']  # Ordered list of include paths
    self.cpp_info.libs = []  # The libs to link against
    self.cpp_info.system_libs = []  # System libs to link against
    self.cpp_info.libdirs = ['lib']  # Directories where libraries can be found
    self.cpp_info.resdirs = ['res']  # Directories where resources, data, etc. can be found
    self.cpp_info.bindirs = ['bin']  # Directories where executables and shared libs can be found
    self.cpp_info.srcdirs = []  # Directories where sources can be found (debugging, reusing sources)
    self.cpp_info.build_modules = {}  # Build system utility module files
    self.cpp_info.defines = []  # preprocessor definitions
    self.cpp_info.cflags = []  # pure C flags
    self.cpp_info.cxxflags = []  # C++ compilation flags
    self.cpp_info.sharedlinkflags = []  # linker flags
    self.cpp_info.exelinkflags = []  # linker flags
    self.cpp_info.components  # Dictionary with the different components a package may have
    self.cpp_info.requires = None  # List of components from requirements

- **names**: Alternative name(s) for the package to be used by generators.
- **includedirs**: List of relative paths (starting from the package root) of directories where headers can be found. By default it is
  initialized to ``['include']``, and it is rarely changed.
- **libs**: Ordered list of libs the client should link against. Empty by default, it is common that different configurations produce
  different library names. For example:

  .. code-block:: python

      def package_info(self):
          if not self.settings.os == "Windows":
              self.cpp_info.libs = ["libzmq-static.a"] if self.options.static else ["libzmq.so"]
          else:
              ...

- **libdirs**: List of relative paths (starting from the package root) of directories in which to find library object binaries (\*.lib,
  \*.a, \*.so, \*.dylib). By default it is initialized to ``['lib']``, and it is rarely changed.
- **resdirs**: List of relative paths (starting from the package root) of directories in which to find resource files (images, xml, etc). By
  default it is initialized to ``['res']``, and it is rarely changed.
- **bindirs**: List of relative paths (starting from the package root) of directories in which to find library runtime binaries (like
  Windows .dlls). By default it is initialized to ``['bin']``, and it is rarely changed.
- **srcdirs**: List of relative paths (starting from the package root) of directories in which to find sources (like
  .c, .cpp). By default it is empty. It might be used to store sources (for later debugging of packages, or to reuse those sources building
  them in other packages too).
- **build_modules**: Dictionary of lists per generator containing relative paths to build system related utility module files created by the package. Used by CMake generators to
  include *.cmake* files with functions for consumers. e.g: ``self.cpp_info.build_modules["cmake_find_package"].append("cmake/myfunctions.cmake")``. Those files
  will be included automatically in `cmake`/`cmake_multi` generators when using `conan_basic_setup()` and will be automatically added in
  `cmake_find_package`/`cmake_find_package_multi` generators when `find_package()` is used.
- **defines**: Ordered list of preprocessor directives. It is common that the consumers have to specify some sort of defines in some cases,
  so that including the library headers matches the binaries.
- **system_libs**: Ordered list of system libs the consumer should link against. Empty by default.
- **cflags**, **cxxflags**, **sharedlinkflags**, **exelinkflags**: List of flags that the consumer should activate for proper behavior.
  Usage of C++11 could be configured here, for example, although it is true that the consumer may want to do some flag processing to check
  if different dependencies are setting incompatible flags (c++11 after c++14).

  .. code-block:: python

      if self.options.static:
          if self.settings.compiler == "Visual Studio":
              self.cpp_info.libs.append("ws2_32")
          self.cpp_info.defines = ["ZMQ_STATIC"]

          if not self.settings.os == "Windows":
              self.cpp_info.cxxflags = ["-pthread"]

  Note that due to the way that some build systems, like CMake, manage forward and back slashes, it might
  be more robust passing flags for Visual Studio compiler with dash instead. Using ``"/NODEFAULTLIB:MSVCRT"``,
  for example, might fail when using CMake targets mode, so the following is preferred and works both
  in the global and targets mode of CMake:

  .. code-block:: python

      def package_info(self):
          self.cpp_info.exelinkflags = ["-NODEFAULTLIB:MSVCRT",
                                        "-DEFAULTLIB:LIBCMT"]

- **components**: **[Experimental]** Dictionary with names as keys and a component object as value to model the different components a
  package may have: libraries, executables... Read more about this feature at :ref:`package_information_components`.
- **requires**: **[Experimental]** List of components from the requirements this package (and its consumers) should link with. It will
  be used by generators that add support for components features (:ref:`package_information_components`).


If your recipe has requirements, you can access to the information stored in the ``cpp_info`` of your requirements
using the ``deps_cpp_info`` object:

.. code-block:: python

    class OtherConan(ConanFile):
        name = "OtherLib"
        version = "1.0"
        requires = "mylib/1.6.0@conan/stable"

        def build(self):
            self.output.warn(self.deps_cpp_info["mylib"].libdirs)

.. note::

    Please take into account that defining ``self.cpp_info.bindirs`` directories, does not have any effect on system paths, PATH environment
    variable, nor will be directly accessible by consumers. ``self.cpp_info`` information is translated to build-systems information via
    generators, for example for CMake, it will be a variable in ``conanbuildinfo.cmake``. If you want a package to make accessible its
    executables to its consumers, you have to specify it with ``self.env_info`` as described in :ref:`method_package_info_env_info`.

.. _method_package_info_env_info:

env_info
++++++++

Each package can also define some environment variables that the package needs to be reused. It's specially useful for
:ref:`installer packages<create_installer_packages>`, to set the path with the "bin" folder of the packaged application. This can be done in
the ``env_info`` attribute within the ``package_info()`` method.

.. code-block:: python

    self.env_info.path.append("ANOTHER VALUE") # Append "ANOTHER VALUE" to the path variable
    self.env_info.othervar = "OTHER VALUE" # Assign "OTHER VALUE" to the othervar variable
    self.env_info.thirdvar.append("some value") # Every variable can be set or appended a new value

One of the most typical usages for the PATH environment variable, would be to add the current binary package directories to the path, so
consumers can use those executables easily:

.. code-block:: python

    # assuming the binaries are in the "bin" subfolder
    self.env_info.PATH.append(os.path.join(self.package_folder, "bin"))

The :ref:`virtualenv<virtual_environment_generator>` generator will use the ``self.env_info`` variables to prepare a script to
activate/deactivate a virtual environment. However, this could be directly done using the :ref:`virtualrunenv_generator` generator.

They will be automatically applied before calling the consumer *conanfile.py* methods ``source()``, ``build()``, ``package()`` and
``imports()``.

If your recipe has requirements, you can access to your requirements ``env_info`` as well using the ``deps_env_info`` object.

.. code-block:: python

    class OtherConan(ConanFile):
        name = "OtherLib"
        version = "1.0"
        requires = "mylib/1.6.0@conan/stable"

        def build(self):
            self.output.warn(self.deps_env_info["mylib"].othervar)

.. _method_package_info_user_info:

user_info
+++++++++

If you need to declare custom variables not related with C/C++ (``cpp_info``) and the variables are not environment variables
(``env_info``), you can use the ``self.user_info`` object.

Currently only the ``cmake``, ``cmake_multi`` and ``txt`` generators supports ``user_info`` variables.

.. code-block:: python

    class MyLibConan(ConanFile):
        name = "mylib"
        version = "1.6.0"

        # ...

        def package_info(self):
            self.user_info.var1 = 2

For the example above, in the ``cmake`` and ``cmake_multi`` generators, a variable ``CONAN_USER_MYLIB_var1`` will be declared. If your
recipe has requirements, you can access to your requirements ``user_info`` using the ``deps_user_info`` object.

.. code-block:: python

    class OtherConan(ConanFile):
        name = "otherlib"
        version = "1.0"
        requires = "mylib/1.6.0@conan/stable"

        def build(self):
            self.out.warn(self.deps_user_info["mylib"].var1)

.. important::

    Both ``env_info`` and ``user_info`` objects store information in a "key <-> value" form and the values are always considered strings.
    This is done for serialization purposes to *conanbuildinfo.txt* files and to avoid the deserialization of complex structures. It is up to the consumer to convert the string to the expected type:

    .. code-block:: python

        # In a dependency
        self.user_info.jars="jar1.jar, jar2.jar, jar3.jar"  # Use a string, not a list
        ...

        # In the dependent conanfile
        jars = self.deps_user_info["pkg"].jars
        jar_list = jars.replace(" ", "").split(",")


set_name(), set_version()
--------------------------
Dynamically define ``name`` and ``version`` attributes in the recipe with these methods. The following example
defines the package name reading it from a *name.txt* file and the version from the branch and commit of the
recipe's repository.

These functions are executed after assigning the values of the ``name`` and ``version`` if they are provided
from the command line.

..  code-block:: python

    from conans import ConanFile, tools

    class HelloConan(ConanFile):
        def set_name(self):
            # Read the value from 'name.txt' if it is not provided in the command line
            self.name = self.name or tools.load("name.txt")

        def set_version(self):
            git = tools.Git()
            self.version = "%s_%s" % (git.get_branch(), git.get_revision())

The ``set_name()`` and ``set_version()`` methods should respectively set the ``self.name`` and ``self.version`` attributes.
These methods are only executed when the recipe is in a user folder (:command:`export`, :command:`create` and
:command:`install <path>` commands).

The above example uses the current working directory as the one to resolve the relative "name.txt" path and the git repository.
That means that the "name.txt" should exist in the directory where conan was launched.
To define a relative path to the *conanfile.py*, irrespective of the current working directory it is necessary to do:

..  code-block:: python

    import os
    from conans import ConanFile, tools

    class HelloConan(ConanFile):
        def set_name(self):
            f = os.path.join(self.recipe_folder, "name.txt")
            self.name = tools.load(f)

        def set_version(self):
            git = tools.Git(folder=self.recipe_folder)
            self.version = "%s_%s" % (git.get_branch(), git.get_revision())


.. warning::

    The ``set_name()`` and ``set_version()`` methods are alternatives to the ``name`` and ``version`` attributes. It is
    not advised or supported to define both a ``name`` attribute and a ``set_name()`` method.  Likewise, it is
    not advised or supported to define both a ``version`` attribute and a ``set_version()`` method. If you define both,
    you may experience unexpected behavior.

.. seealso::

    See more examples :ref:`in this howto <capture_version>`.


.. _method_configure_config_options:


configure(), config_options()
-----------------------------

If the package options and settings are related, and you want to configure either, you can do so in the ``configure()`` and
``config_options()`` methods.

..  code-block:: python

    class MyLibConan(ConanFile):
        name = "MyLib"
        version = "2.5"
        settings = "os", "compiler", "build_type", "arch"
        options = {"static": [True, False],
                    "header_only": [True False]}

        def configure(self):
            # If header only, the compiler, etc, does not affect the package!
            if self.options.header_only:
                self.settings.clear()
                del self.options.static

The package has 2 options set, to be compiled as a static (as opposed to shared) library, and also not to involve any builds, because
header-only libraries will be used. In this case, the settings that would affect a normal build, and even the other option (static vs
shared) do not make sense, so we just clear them. That means, if someone consumes MyLib with the ``header_only=True`` option, the package
downloaded and used will be the same, irrespective of the OS, compiler or architecture the consumer is building with.

You can also restrict the settings used deleting any specific one. For example, it is quite common
for C libraries to delete the ``compiler.libcxx`` and ``compiler.cppstd`` as your library does not
depend on any C++ standard library:

.. code-block:: python

    def configure(self):
        del self.settings.compiler.libcxx
        del self.settings.compiler.cppstd

The most typical usage would be the one with ``configure()`` while ``config_options()`` should be used more sparingly. ``config_options()``
is used to configure or constraint the available options in a package, **before** they are given a value. So when a value is tried to be
assigned it will raise an error. For example, let's suppose that a certain package library cannot be built as shared library in Windows, it
can be done:

.. code-block:: python

    def config_options(self):
        if self.settings.os == "Windows":
            del self.options.shared

This will be executed before the actual assignment of ``options`` (then, such ``options`` values cannot be used inside this function), so
the command :command:`conan install -o pkg:shared=True` will raise an exception in Windows saying that ``shared`` is not an option for such
package.

These methods can also be used to assign values to options as seen in :ref:`conanfile_options`. Values assigned
in the ``configure()`` method cannot be overridden, while values assigned in ``config_options()`` can.

.. _invalid_configuration:

Invalid configuration
+++++++++++++++++++++

Conan allows the recipe creator to declare invalid configurations, those that are known not to work
with the library being packaged. There is an especial kind of exception that can be raised from
the ``validate()`` method to state this situation: ``conans.errors.ConanInvalidConfiguration``. Here
it is an example of a recipe for a library that doesn't support Windows operating system:

.. code-block:: python

    def validate(self):
        if self.settings.os != "Windows":
            raise ConanInvalidConfiguration("Library MyLib is only supported for Windows")

This exception will be propagated and Conan application will finish with a :ref:`special return code <invalid_configuration_return_code>`.

.. note::

    For managing invalid configurations, please check the new experimental ``validate()`` method (:ref:`method_validate`).


.. _method_validate:

validate()
----------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.32.0 <https://github.com/conan-io/conan/releases/tag/1.32.0>`_

The ``validate()`` method can be used to mark a binary as "impossible" or invalid for a given configuration. For example,
if a given library does not build or work at all in Windows it can be defined as:

.. code-block:: python

    from conans import ConanFile
    from conans.errors import ConanInvalidConfiguration

    class Pkg(ConanFile):
        settings = "os"

        def validate(self):
            if self.settings.os == "Windows":
                raise ConanInvalidConfiguration("Windows not supported")

If you try to use, consume or build such a package, it will raise an error, returning exit code :ref:`exit code <invalid_configuration_return_code>`:

.. code-block:: bash

    $ conan create . pkg/0.1@ -s os=Windows
    ...
    Packages
        pkg/0.1:INVALID - Invalid
    ...
    > ERROR: There are invalid packages (packages that cannot exist for this configuration):
    > pkg/0.1: Invalid ID: Windows not supported

A major difference with ``configure()`` is that this information can be queried with the ``conan info`` command, for example this
is possible without getting an error:

.. code-block:: bash

    $ conan export . test/0.1@user/testing
    ...
    > test/0.1@user/testing: Exported revision: ...

    $ conan info test/0.1@user/testing
    >test/0.1@user/testing
        ID: INVALID
        BuildID: None
        Remote: None
        ...

Another important difference with the ``configure()`` method, is that ``validate()`` is evaluated after the graph has been computed and
the information has been propagated downstream. So the values used in ``validate()`` are guaranteed to be final real values,
while values at ``configure()`` time are not. This might be important, for example when checking values of options of dependencies:

.. code-block:: python

    from conans import ConanFile
    from conans.errors import ConanInvalidConfiguration

    class Pkg(ConanFile):
        requires = "dep/0.1"

        def validate(self):
            if self.options["dep"].myoption == 2:
                raise ConanInvalidConfiguration("Option 2 of 'dep' not supported")


If a package uses ``compatible_packages`` feature, it should not add to those compatible packages configurations that will not be valid,
for example:

.. code-block:: python

    from conans import ConanFile
    from conans.errors import ConanInvalidConfiguration

    class Pkg(ConanFile):
        settings = "os", "build_type"

        def validate(self):
            if self.settings.os == "Windows":
                raise ConanInvalidConfiguration("Windows not supported")

        def package_id(self):
            if self.settings.build_type == "Debug" and self.settings.os != "Windows":
                compatible_pkg = self.info.clone()
                compatible_pkg.settings.build_type = "Release"
                self.compatible_packages.append(compatible_pkg)

Note the ``self.settings.os != "Windows"`` in the ``package_id()``. If this is not provided, the ``validate()`` might still work and
raise an error, but in the best case it will be wasted resources (compatible packages do more API calls to check them), so it is
strongly recommended to properly define the ``package_id()`` method to no include incompatible configurations.


validate_build()
----------------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.51.0 <https://github.com/conan-io/conan/releases/tag/1.51.0>`_

The ``validate_build()`` method is used to verify if a configuration is valid for building a package. It is different
from the ``validate()`` method that checks if the binary package is "impossible" or invalid for a given configuration.

In Conan 2.0, the ``validate()`` method should do the checks of the settings and options using the ``self.info.settings``
and ``self.info.options``.

The ``validate_build()`` method has to use always the ``self.settings`` and ``self.options``:

.. code-block:: python

    from conan import ConanFile
    from conan.errors import ConanInvalidConfiguration

    class myConan(ConanFile):
        name = "foo"
        version = "1.0"
        settings = "os", "arch", "compiler"

        def package_id(self):
            # For this package, it doesn't matter the compiler used for the binary package
            del self.info.settings.compiler

        def validate_build(self):
            # But we know this cannot be build with "gcc"
            if self.settings.compiler == "gcc":
                raise ConanInvalidConfiguration("This doesn't build in GCC")

        def validate(self):
            # We shouldn't check here the self.info.settings.compiler because it has been removed in the package_id()
            # so it doesn't make sense to check if the binary is compatible with gcc because the compiler doesn't matter
            pass





.. _method_requirements:

requirements()
--------------

Besides the ``requires`` field, more advanced requirement logic can be defined in the ``requirements()`` optional method, using for example
values from the package ``settings`` or ``options``:

.. code-block:: python

    def requirements(self):
        if self.options.myoption:
            self.requires("zlib/1.2@drl/testing")
        else:
            self.requires("opencv/2.2@drl/stable")

This is a powerful mechanism for handling **conditional dependencies**.

When you are inside the method, each call to ``self.requires()`` will add the corresponding requirement to the current list of requirements.
It also has optional parameters that allow defining the special cases, as is shown below:

..  code-block:: python

    def requirements(self):
        self.requires("zlib/1.2@drl/testing", private=True, override=False)

``self.requires()`` parameters:

    - **override** (Optional, Defaulted to ``False``): True means that this is not an actual requirement, but something to be passed
      upstream and override possible existing values.
    - **private** (Optional, Defaulted to ``False``): True means that this requirement will be somewhat embedded, and totally hidden. It might be necessary in some extreme cases, like having to use two
      different versions of the same library (provided that they are totally hidden in a shared library, for
      example), but it is mostly discouraged otherwise.

.. note::

    To prevent accidental override of transitive dependencies, check the config variable
    :ref:`general.error_on_override<conan_conf>` or the environment variable
    :ref:`CONAN_ERROR_ON_OVERRIDE<env_vars_conan_error_on_override>`.


build_requirements()
--------------------

The requires specified in this method are only installed and used when the package is built from sources.
If there is an existing pre-compiled binary, then the tool requirements for this package will not be retrieved.

This method is useful for defining conditional tool requirements, for example:

.. code-block:: python

    class MyPkg(ConanFile):

        def build_requirements(self):
            if self.settings.os == "Windows":
                self.tool_requires("tool_win/0.1@user/stable")

.. seealso::

    :ref:`Tool requirements <build_requires>`

.. _method_system_requirements:

system_requirements()
---------------------

It is possible to install system-wide packages from Conan. Just add a ``system_requirements()`` method to your conanfile and specify what
you need there.

For a special use case you can use also ``conans.tools.os_info`` object to detect the operating system, version and distribution (Linux):

- ``os_info.is_linux``: True if Linux.
- ``os_info.is_windows``: True if Windows.
- ``os_info.is_macos``: True if macOS.
- ``os_info.is_freebsd``: True if FreeBSD.
- ``os_info.is_solaris``: True if SunOS.
- ``os_info.os_version``: OS version.
- ``os_info.os_version_name``: Common name of the OS (Windows 7, Mountain Lion, Wheezy...).
- ``os_info.linux_distro``: Linux distribution name (None if not Linux).
- ``os_info.bash_path``: Returns the absolute path to a bash in the system.
- ``os_info.uname(options=None)``: Runs the "uname" command and returns the output. You can pass arguments with the `options` parameter.
- ``os_info.detect_windows_subsystem()``: Returns "MSYS", "MSYS2", "CYGWIN" or "WSL" if any of these Windows subsystems are detected.

.. warning::

    The values returned from some of these variables (``linux_distro``, ``os_version`` and ``os_version_name``) use the external
    dependency `distro <https://pypi.org/project/distro/>`_, values returned might be different from one version to another,
    please check their changelog for bugfixes and new features.


You can also use ``SystemPackageTool`` class, that will automatically invoke the right system package
tool: **apt**, **yum**, **dnf**, **pkg**, **pkgutil**, **brew** and **pacman** depending on the
system we are running.

..  code-block:: python

    from conans.tools import os_info, SystemPackageTool

    def system_requirements(self):
        pack_name = None
        if os_info.linux_distro == "ubuntu":
            if os_info.os_version > "12":
                pack_name = "package_name_in_ubuntu_10"
            else:
                pack_name = "package_name_in_ubuntu_12"
        elif os_info.linux_distro == "fedora" or os_info.linux_distro == "centos":
            pack_name = "package_name_in_fedora_and_centos"
        elif os_info.is_macos:
            pack_name = "package_name_in_macos"
        elif os_info.is_freebsd:
            pack_name = "package_name_in_freebsd"
        elif os_info.is_solaris:
            pack_name = "package_name_in_solaris"

        if pack_name:
            installer = SystemPackageTool()
            installer.install(pack_name) # Install the package, will update the package database if pack_name isn't already installed

On Windows, there is no standard package manager, however **choco** can be invoked as an optional:

..  code-block:: python

    from conans.tools import os_info, SystemPackageTool, ChocolateyTool

    def system_requirements(self):
        if os_info.is_windows:
            pack_name = "package_name_in_windows"
            installer = SystemPackageTool(tool=ChocolateyTool()) # Invoke choco package manager to install the package
            installer.install(pack_name)

.. _systempackagetool:

SystemPackageTool
+++++++++++++++++

.. warning::

    SystemPackageTool will disappear in Conan 2.0, there's already a new implementation of
    these wrappers in :ref:`conan_tools_system_package_manager` that will be the default
    in Conan 2.0.

.. code-block:: python

    def SystemPackageTool(runner=None, os_info=None, tool=None, recommends=False, output=None, conanfile=None, default_mode="enabled")

Available tool classes: **AptTool**, **YumTool**, **DnfTool**, **BrewTool**, **PkgTool**,
**PkgUtilTool**, **ChocolateyTool**, **PacManTool**.

Methods:
    - **add_repository(repository, repo_key=None)**: Add ``repository`` address in your current repo list.
    - **update()**: Updates the system package manager database. It's called automatically from the ``install()`` method by default.
    - **install(packages, update=True, force=False)**: Installs the ``packages`` (could be a list or a string). If ``update`` is True it
      will execute ``update()`` first if it's needed. The packages won't be installed if they are already installed at least of ``force``
      parameter is set to True. If ``packages`` is a list the first available package will be picked (short-circuit like logical **or**).
      **Note**: This list of packages is intended for providing **alternative** names for the same package, to account for small variations
      of the name for the same package in different distros. To install different packages, one call to ``install()`` per package is necessary.
    - **install_packages(packages, update=True, force=False, arch_names=None)**: Installs all ``packages`` (could be a list or a string).
      If ``update`` is True it will execute ``update()`` first if it's needed. The packages won't be installed if they are already installed
      at least of ``force`` parameter is set to True. If ``packages`` has a nested list or tuple, the first available package will be picked
      (short-circuit like logical **or**).
    - **installed(package_name)**: Verify if ``package_name`` is actually installed. It returns ``True`` if it is installed, otherwise ``False``.

The use of ``sudo`` in the internals of the ``install()`` and ``update()`` methods is controlled by the ``CONAN_SYSREQUIRES_SUDO``
environment variable, so if the users don't need sudo permissions, it is easy to opt-in/out.

When the environment variable ``CONAN_SYSREQUIRES_SUDO`` is not defined, Conan will try to use :command:`sudo` if the following conditions are met:

    - :command:`sudo` is available in the ``PATH``.
    - The platform name is ``posix`` and the UID (user id) is not ``0``

Also, when the environment variable :ref:`CONAN_SYSREQUIRES_MODE <env_vars_conan_sysrequires_mode>`
is not defined, Conan will work as if its value was ``enabled`` unless you pass the ``default_mode``
argument to the constructor of ``SystemPackageTool``. In that case, it will work as if
``CONAN_SYSREQUIRES_MODE`` had been defined to that value. If ``CONAN_SYSREQUIRES_MODE`` is defined,
it will take preference and the ``default_mode`` parameter will not affect. This can be useful when a
recipe has system requirements but we don't want to automatically install them if the user has not
defined ``CONAN_SYSREQUIRES_MODE`` but to warn him about the missing requirements and allowing him to
install them.

Conan will keep track of the execution of this method, so that it is not invoked again and again at every Conan command. The execution is
done per package, since some packages of the same library might have different system dependencies. If you are sure that all your binary
packages have the same system requirements, just add the following line to your method:

..  code-block:: python

    def system_requirements(self):
        self.global_system_requirements=True
        if ...

To install multi-arch packages it is possible passing the desired architecture manually according
your package manager:

..  code-block:: python

            name = "foobar"
            platforms = {"x86_64": "amd64", "x86": "i386"}
            installer = SystemPackageTool(tool=AptTool())
            installer.install("%s:%s" % (name, platforms[self.settings.arch]))

However, it requires a boilerplate which could be automatically solved by your settings in ConanFile:

..  code-block:: python

            installer = SystemPackageTool(conanfile=self)
            installer.install(name)

The ``SystemPackageTool`` is adapted to support possible prefixes and suffixes, according to the
instance of the package manager. It validates whether your current settings are configured for
cross-building, and if so, it will update the package name to be installed according to
``self.settings.arch``.

To install more than one package at once:

..  code-block:: python

        def system_requirements(self):
            packages = [("vim", "nano", "emacs"), "firefox", "chromium"]
            installer = SystemPackageTool()
            installer.install_packages(packages)
            # e.g. apt-get install -y --no-recommends vim firefox chromium

The ``install_packages`` will install the first text editor available (only one) following the tuple order, while it will install both web browsers.


.. _method_imports:

imports()
---------

Importing files copies files from the local store to your project. This feature is handy for copying shared libraries (*dylib* in Mac, *dll*
in Win) to the directory of your executable, so that you don't have to mess with your PATH to run them. But there are other use cases:

- Copy an executable to your project, so that it can be easily run. A good example is the **Google's protobuf** code generator.
- Copy package data to your project, like configuration, images, sounds... A good example is the **OpenCV** demo, in which face detection
  XML pattern files are required.

Importing files is also very convenient in order to redistribute your application, as many times you will just have to bundle your project's
bin folder.

A typical ``imports()`` method for shared libs could be:

.. code-block:: python

   def imports(self):
      self.copy("*.dll", "", "bin")
      self.copy("*.dylib", "", "lib")

The ``self.copy()`` method inside ``imports()`` supports the following arguments:

.. code-block:: python

    def copy(pattern, dst="", src="", root_package=None, folder=False, ignore_case=True, excludes=None, keep_path=True)

Parameters:
    - **pattern** (Required): An fnmatch file pattern of the files that should be copied.
    - **dst** (Optional, Defaulted to ``""``): Destination local folder, with reference to current directory, to which the files will be
      copied.
    - **src** (Optional, Defaulted to ``""``): Source folder in which those files will be searched. This folder will be stripped from the
      dst parameter. E.g., `lib/Debug/x86`. It accepts symbolic folder names like ``@bindirs`` and ``@libdirs`` which will map to the
      ``self.cpp_info.bindirs`` and ``self.cpp_info.libdirs`` of the source package, instead of a hardcoded name.
    - **root_package** (Optional, Defaulted to *all packages in deps*): An fnmatch pattern of the package name ("OpenCV", "Boost") from
      which files will be copied.
    - **folder** (Optional, Defaulted to ``False``): If enabled, it will copy the files from the local cache to a subfolder named as the
      package containing the files. Useful to avoid conflicting imports of files with the same name (e.g. License).
    - **ignore_case** (Optional, Defaulted to ``True``): If enabled, it will do a case-insensitive pattern matching.
    - **excludes** (Optional, Defaulted to ``None``): Allows defining a list of patterns (even a single pattern) to be excluded from the
      copy, even if they match the main ``pattern``.
    - **keep_path** (Optional, Defaulted to ``True``): Means if you want to keep the relative path when you copy the files from the **src**
      folder to the **dst** one. Useful to ignore (``keep_path=False``) path of *library.dll* files in the package it is imported from.

Example to collect license files from dependencies:

.. code-block:: python

    def imports(self):
        self.copy("license*", dst="licenses", folder=True, ignore_case=True)

If you want to be able to customize the output user directory to work with both the ``cmake`` and ``cmake_multi`` generators, then you can
do:

.. code-block:: python

    def imports(self):
        dest = os.getenv("CONAN_IMPORT_PATH", "bin")
        self.copy("*.dll", dst=dest, src="bin")
        self.copy("*.dylib*", dst=dest, src="lib")

And then use, for example: :command:`conan install . -e CONAN_IMPORT_PATH=Release -g cmake_multi`


To import files from packages that have different layouts, for example a package uses folder ``libraries`` instead of ``lib``,
or to import files from packages that could be in editable mode, a symbolic ``src`` argument can be provided:

.. code-block:: python

    def imports(self):
        self.copy("*", src="@bindirs", dst="bin")
        self.copy("*", src="@libdirs", dst="lib")

This will import all files from all the dependencies ``self.cpp_info.bindirs`` folders to the local "bin" folder, and all files
from the dependencies ``self.cpp_info.libdirs`` folders to the local "lib" folder. This include packages that are in *editable*
mode and declares ``[libdirs]`` and ``[bindirs]`` in their editable layouts.


When a conanfile recipe has an ``imports()`` method and it builds from sources, it will do the following:

- Before running ``build()`` it will execute ``imports()`` in the build folder, copying dependencies artifacts
- Run the ``build()`` method, which could use such imported binaries.
- Remove the copied (imported) artifacts after ``build()`` is finished.

You can use the :ref:`keep_imports <keep_imports>` attribute to keep the imported artifacts, and maybe :ref:`repackage <repackage>` them.

.. _method_package_id:

package_id()
------------

Creates a unique ID for the package. Default package ID is calculated using ``settings``, ``options`` and ``requires`` properties. When a
package creator specifies the values for any of those properties, it is telling that any value change will require a different binary
package.

However, sometimes a package creator would need to alter the default behavior, for example, to have only one binary package for several
different compiler versions. In that case you can set a custom ``self.info`` object implementing this method and the package ID will be
computed with the given information:

.. code-block:: python

    def package_id(self):
        v = Version(str(self.settings.compiler.version))
        if self.settings.compiler == "gcc" and (v >= "4.5" and v < "5.0"):
            self.info.settings.compiler.version = "GCC 4 between 4.5 and 5.0"

Please, check the section :ref:`define_abi_compatibility` to get more details.

self.info
+++++++++

This ``self.info`` object stores the information that will be used to compute the package ID.

This object can be manipulated to reflect the information you want in the computation of the package ID. For example, you can delete
any setting or option:

.. code-block:: python

    def package_id(self):
        del self.info.settings.compiler
        del self.info.options.shared

self.info.clear()
^^^^^^^^^^^^^^^^^

Available since: `1.50.0 <https://github.com/conan-io/conan/releases/tag/1.50.0>`_

The package will always be the same, irrespective of the settings (OS, compiler or architecture), options and dependencies.

.. code-block:: python

    def package_id(self):
        self.info.clear()


self.info.shared_library_package_id()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Available since: `1.19.2 <https://github.com/conan-io/conan/releases/tag/1.19.2>`_

When a shared library links with a static library, the binary code of the later one is "embedded" or copied into the shared library.
That means that any change in the static library basically requires a new binary re-build of the shared one to integrate those changes.
Note that this doesn't happen in the static-static and shared-shared library dependencies.

Use this ``shared_library_package_id()`` helper in the ``package_id()`` method:

.. code-block:: python

    def package_id(self):
        self.info.shared_library_package_id()

This helper automatically detects if the current package has the ``shared`` option and it is ``True`` and if it is depending on static libraries in other packages (having a ``shared`` option equal ``False`` or not having it, which means a header-only library). Only then, any change in the dependencies will affect the ``package_id`` of this package, (internally, ``package_revision_mode`` is applied to the dependencies).
It is recommended its usage in packages that have the ``shared`` option.

If you want to have in your dependency graph all static libraries or all shared libraries, (but not shared with embedded static ones) it can be defined with a ``*:shared=True``
option in command line or profiles, but can also be defined in recipes like:

.. code-block:: python

    def configure(self):
        if self.options.shared:
            self.options["*"].shared = True

self.info.vs_toolset_compatible() / self.info.vs_toolset_incompatible()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default (``vs_toolset_compatible()`` mode) Conan will generate the same binary package when the compiler is Visual Studio and the
``compiler.toolset`` matches the specified ``compiler.version``. For example, if we install some packages specifying the following settings:

.. code-block:: python

    def package_id(self):
        self.info.vs_toolset_compatible()
        # self.info.vs_toolset_incompatible()

.. code-block:: text

    compiler="Visual Studio"
    compiler.version=14

And then we install again specifying these settings:

.. code-block:: text

    compiler="Visual Studio"
    compiler.version=15
    compiler.toolset=v140

The compiler version is different, but Conan will not install a different package, because the used ``toolchain`` in both cases are
considered the same. You can deactivate this default behavior using calling ``self.info.vs_toolset_incompatible()``.

This is the relation of Visual Studio versions and the compatible toolchain:

+-----------------------+--------------------+
| Visual Studio Version | Compatible toolset |
+=======================+====================+
| 17                    | v143               |
+-----------------------+--------------------+
| 16                    | v142               |
+-----------------------+--------------------+
| 15                    | v141               |
+-----------------------+--------------------+
| 14                    | v140               |
+-----------------------+--------------------+
| 12                    | v120               |
+-----------------------+--------------------+
| 11                    | v110               |
+-----------------------+--------------------+
| 10                    | v100               |
+-----------------------+--------------------+
| 9                     | v90                |
+-----------------------+--------------------+
| 8                     | v80                |
+-----------------------+--------------------+

.. _info_discard_include_build_settings:

self.info.discard_build_settings() / self.info.include_build_settings()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default (``discard_build_settings()``) Conan will generate the same binary when you change the ``os_build`` or ``arch_build`` when the
``os`` and ``arch`` are declared respectively. This is because ``os_build`` represent the machine running Conan, so, for the consumer, the
only setting that matters is where the built software will run, not where is running the compilation. The same applies to ``arch_build``.

With ``self.info.include_build_settings()``, Conan will generate different packages when you change the ``os_build`` or ``arch_build``.

.. code-block:: python

    def package_id(self):
        self.info.discard_build_settings()
        # self.info.include_build_settings()



self.info.default_std_matching() / self.info.default_std_non_matching()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default (``default_std_matching()``) Conan will detect the default C++ standard of your compiler to
not generate different binary packages.

For example, you already built some ``gcc 6.1`` packages, where the default std is ``gnu14``.
If you specify a value for the setting ``compiler.cppstd`` equal to the default one, ``gnu14``, Conan won't generate
new packages, because it was already the default of your compiler.

With ``self.info.default_std_non_matching()``, Conan will generate different packages when you specify the ``compiler.cppstd``
even if it matches with the default of the compiler being used:

.. code-block:: python

    def package_id(self):
        self.info.default_std_non_matching()
        # self.info.default_std_matching()


Same behavior applies if you use the deprecated setting ``cppstd``.


Compatible packages
^^^^^^^^^^^^^^^^^^^
The ``package_id()`` method serves to define the "canonical" binary package ID, the identifier of the binary that correspond to the
input configuration of settings and options. This canonical binary package ID will be always computed, and Conan will check for its
existence to be downloaded and installed.

If the binary of that package ID is not found, Conan lets the recipe writer define an ordered list of compatible package IDs, of other configurations
that should be binary compatible and can be used as a fallback. The syntax to do this is:

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        settings = "os", "compiler", "arch", "build_type"

        def package_id(self):
            if self.settings.compiler == "gcc" and self.settings.compiler.version == "4.9":
                compatible_pkg = self.info.clone()
                compatible_pkg.settings.compiler.version = "4.8"
                self.compatible_packages.append(compatible_pkg)

This will define that, if we try to install this package with ``gcc 4.9`` and there isn't a binary available for that configuration, Conan will check
if there is one available built with ``gcc 4.8`` and use it. But not the other way round.

.. seealso::

    For more information about :ref:`compatible packages read this <compatible_packages>`


.. _method_build_id:

build_id()
----------

In the general case, there is one build folder for each binary package, with the exact same hash/ID of the package. However this behavior
can be changed, there are a couple of scenarios that this might be interesting:

- You have a build script that generates several different configurations at once, like both debug and release artifacts, but you actually
  want to package and consume them separately. Same for different architectures or any other setting.
- You build just one configuration (like release), but you want to create different binary packages for different consuming cases. For
  example, if you have created tests for the library in the build step, you might want to create two packages: one just containing the
  library for general usage, and another one also containing the tests. First package could be used as a reference and the other one as a
  tool to debug errors.

In both cases, if using different settings, the system will build twice (or more times) the same binaries, just to produce a different final
binary package. With the ``build_id()`` method this logic can be changed. ``build_id()`` will create a new package ID/hash for the build
folder, and you can define the logic you want in it. For example:

..  code-block:: python

    settings = "os", "compiler", "arch", "build_type"

    def build_id(self):
        self.info_build.settings.build_type = "Any"

So this recipe will generate a final different package for each debug/release configuration. But as the ``build_id()`` will generate the
same ID for any ``build_type``, then just one folder and one build will be done. Such build should build both debug and release artifacts,
and then the ``package()`` method should package them accordingly to the ``self.settings.build_type`` value. Different builds will still be
executed if using different compilers or architectures. This method is basically an optimization of build time, avoiding multiple re-builds.

Other information like custom package options can also be changed:

..  code-block:: python

    def build_id(self):
        self.info_build.options.myoption = 'MyValue' # any value possible
        self.info_build.options.fullsource = 'Always'

If the ``build_id()`` method does not modify the ``build_id``, and produce a different one than
the ``package_id``, then the standard behavior will be applied. Consider the following:

..  code-block:: python

    settings = "os", "compiler", "arch", "build_type"

    def build_id(self):
        if self.settings.os == "Windows":
            self.info_build.settings.build_type = "Any"

This will only produce a build ID different if the package is for Windows. So the behavior
in any other OS will be the standard one, as if the ``build_id()`` method was not defined:
the build folder will be wiped at each :command:`conan create` command and a clean build will
be done.

.. _method_compatibility:

compatibility()
---------------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since Conan `1.47.0 <https://github.com/conan-io/conan/releases/tag/1.47.0>`_

This method can be used in a *conanfile.py* to define packages that are compatible between
each other. If there are not binaries available for the requested settings and options
this mechanism will retrieve the compatible packages' binaries if they exist.  The method
should return a list of compatible configurations. For example, if we want that binaries
built with gcc versions 4.8, 4.7 and 4.6 are considered compatible with the ones compiled
with 4.9 we could declare the ``compatibility()`` like this:

..  code-block:: python

    def compatibility(self):
        if self.settings.compiler == "gcc" and self.settings.compiler.version == "4.9":
            return [{"settings": [("compiler.version", v)]}
                    for v in ("4.8", "4.7", "4.6")]

The format of the list returned is as shown below:

..  code-block:: python

        [
            {
                "settings": [(<setting>, <value>), (<setting>, <value>), ...], 
                "options": [(<option>, <value>), (<option>, <value>), ...]
            },
            {
                "settings": [(<setting>, <value>), (<setting>, <value>), ...], 
                "options": [(<option>, <value>), (<option>, <value>), ...]
            },
            ...
        ]


.. _method_deploy:

deploy()
--------

This method can be used in a *conanfile.py* to install in the system or user folder artifacts from packages.

..  code-block:: python

    def deploy(self):
        self.copy("*.exe")  # copy from current package
        self.copy_deps("*.dll") # copy from dependencies

Where:

- ``self.copy()`` is the ``self.copy()`` method executed inside :ref:`package() method <method_package>`.
- ``self.copy_deps()`` is the same as ``self.copy()`` method inside :ref:`imports() method <method_imports>`.

Both methods allow the definition of absolute paths (to install in the system), in the ``dst`` argument. By default, the ``dst``
destination folder will be the current one.

The ``deploy()`` method is designed to work on a package that is installed directly from its reference, as:

.. code-block:: bash

    $ conan install pkg/0.1@user/channel
    > ...
    > pkg/0.1@user/testing deploy(): Copied 1 '.dll' files: mylib.dll
    > pkg/0.1@user/testing deploy(): Copied 1 '.exe' files: myexe.exe

All other packages and dependencies, even transitive dependencies of ``pkg/0.1@user/testing`` will not be deployed, it is the responsibility
of the installed package to deploy what it needs from its dependencies.


.. seealso::

    For a different approach to deploy package files in the user space folders, check the :ref:`deploy generator<deploy_generator>`.


.. _method_init:

init()
------

This is an optional method for initializing conanfile values, designed for inheritance from :ref:`python requires <python_requires>`.
Assuming we have a ``base/1.1@user/testing`` recipe:

.. code-block:: python

    class MyConanfileBase(object):
        license = "MyLicense"
        settings = "os", # tuple!


    class PyReq(ConanFile):
        name = "base"
        version = "1.1"


We could reuse and inherit from it with:

.. code-block:: python

    class PkgTest(ConanFile):
        license = "MIT"
        settings = "arch", # tuple!
        python_requires = "base/1.1@user/testing"
        python_requires_extend = "base.MyConanfileBase"

        def init(self):
            base = self.python_requires["base"].module.MyConanfileBase
            self.settings = base.settings + self.settings  # Note, adding 2 tuples = tuple
            self.license = base.license  # License is overwritten

The final ``PkgTest`` conanfile will have both ``os`` and ``arch`` as settings, and ``MyLicense`` as license.

This method can also be useful if you need to unconditionally initialize class attributes like
``license`` or ``description`` or any other :ref:`attributes<conanfile_attributes>` from datafiles other than
`conandata.yml`. For example, you have a `json` file containing the information about the
``license``, ``description`` and ``author`` for the library:


.. code-block:: json
    :caption: *data.json*

    {"license": "MIT", "description": "This is my awesome library.", "author": "Me"}

Then, you can load that information from the ``init()``  method:

.. code-block:: python

    import os
    import json
    from conans import ConanFile, load


    class Lib(ConanFile):
        exports = "data.json"
        def init(self):
            data = load(os.path.join(self.recipe_folder, "data.json"))
            d = json.loads(data)
            self.license = d["license"]
            self.description = d["description"]
            self.author = d["author"]

export()
--------

Equivalent to the ``exports`` attribute, but in method form. It supports the ``self.copy()`` to do pattern
based copy of files from the local user folder (the folder containing the *conanfile.py*) to the
cache ``export_folder``

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):

        def export(self):
            self.copy("LICENSE.md")

The current folder (``os.getcwd()``) and the ``self.export_folder`` can be used in the method:

.. code-block:: python

    import os
    from conans import ConanFile
    from conans.tools import save, load

    class Pkg(ConanFile):

        def export(self):
            # we can load files in the user folder
            content = load(os.path.join(os.getcwd(), "data.txt"))
            # We have access to the cache export_folder
            save(os.path.join(self.export_folder, "myfile.txt"), "some content")


The ``self.copy`` support ``src`` and ``dst`` subfolder arguments. The ``src`` is relative to the
current folder (the one containing the *conanfile.py*). The ``dst`` is relative to the cache
``export_folder``.

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):

        def export(self):
            self.output.info("Executing export() method")
            # will copy all .txt files from the local "subfolder" folder to the cache "mydata" one
            self.copy("*.txt", src="mysubfolder", dst="mydata")


export_sources()
----------------

Equivalent to the ``exports_sources`` attribute, but in method form. It supports the ``self.copy()`` to do pattern
based copy of files from the local user folder (the folder containing the *conanfile.py*) to the
cache ``export_sources_folder``

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):

        def export_sources(self):
            self.copy("LICENSE.md")

The current folder (``os.getcwd()``) and the ``self.export_sources_folder`` can be used in the method:

.. code-block:: python

    import os
    from conans import ConanFile
    from conans.tools import save, load

    class Pkg(ConanFile):

        def export_sources(self):
            content = load(os.path.join(os.getcwd(), "data.txt"))
            save(os.path.join(self.export_sources_folder, "myfile.txt"), content)

Note, if the recipe defines the ``layout()`` method and specifies a ``self.folders.source = "src"`` it won't change the
current folder in the ``export_sources`` method. The current dir will be the base source folder (``self.export_sources_folder``).

The ``self.copy`` support ``src`` and ``dst`` subfolder arguments. The ``src`` is relative to the
current folder (the one containing the *conanfile.py*). The ``dst`` is relative to the cache
``export_sources_folder``.

.. code-block:: python

    from conans import ConanFile

    class Pkg(ConanFile):

        def export_sources(self):
            self.output.info("Executing export_sources() method")
            # will copy all .txt files from the local "subfolder" folder to the cache "mydata" one
            self.copy("*.txt", src="mysubfolder", dst="mydata")


generate()
----------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.

Available since: `1.32.0 <https://github.com/conan-io/conan/releases/tag/1.32.0>`_

This method will run after the computation and installation of the dependency graph. This means that it will
run after a :command:`conan install` command, or when a package is being built in the cache, it will be run before
calling the ``build()`` method.

The purpose of ``generate()`` is to prepare the build, generating the necessary files. These files would typically be:

- Files containing information to locate the dependencies, as ``xxxx-config.cmake`` CMake config scripts, or ``xxxx.props``
  Visual Studio property files.
- Environment activation scripts, like ``conanbuildenv.bat`` or ``conanbuildenv.sh``, that define all the necessary environment
  variables necessary for the build.
- Toolchain files, like ``conan_toolchain.cmake``, that contains a mapping between the current Conan settings and options, and the
  build system specific syntax.
- General purpose build information, as a ``conanbuild.conf`` file that could contain information like the CMake generator or
  CMake toolchain file to be used in the ``build()`` method.
- Specific build system files, like ``conanvcvars.bat``, that contains the necessary Visual Studio vcvars.bat call for certain
  build systems like Ninja when compiling with the Microsoft compiler.


The idea is that the ``generate()`` method implements all the necessary logic, making both the user manual builds after a :command:`conan install`
very straightforward, and also the ``build()`` method logic simpler. The build produced by a user in their local flow should result
exactly the same one as the build done in the cache with a ``conan create`` without effort.

In many cases, the ``generate()`` method might not be necessary, and declaring the ``generators`` attribute could be enough:

.. code:: python

    from conans import ConanFile

    class Pkg(ConanFile):
        generators = "CMakeDeps", "CMakeToolchain"


But the ``generate()`` method can explicitly instantiate those generators, customize them, or provide a complete custom
generation. For custom integrations, putting code in a common ``python_require`` would be a good way to avoid repetition in
multiple recipes.

.. code:: python

    from conans import ConanFile
    from conan.tools.cmake import CMakeToolchain

    class Pkg(ConanFile):

        def generate(self):
            tc = CMakeToolchain(self)
            # customize toolchain "tc"
            tc.generate()
            # Or provide your own custom logic

.. _conanfile_layout:

.. _layout_method_reference:

layout()
--------

.. warning::

    This is an **experimental** feature subject to breaking changes in future releases.
    The ``layout()`` feature will be fully functional only in the new build system integrations
    (:ref:`in the conan.tools space <conan_tools>`). If you are using other integrations, they
    might not fully support this feature.

Available since: `1.37.0 <https://github.com/conan-io/conan/releases/tag/1.37.0>`_

Read about the feature :ref:`here<package_layout>`.

In the layout() method you can adjust ``self.folders`` and ``self.cpp``.


.. _layout_folders_reference:


self.folders
++++++++++++


- **self.folders.source** (Defaulted to ""): Specifies a subfolder where the sources are. The ``self.source_folder`` attribute
  inside the ``source(self)`` and ``build(self)`` methods will be set with this subfolder. But the *current working directory*
  in the ``source(self)`` method will not include this subfolder, because it is intended to describe where the sources are after
  downloading (zip, git...) them, not to force where the sources should be. As well, the `export_sources`, `exports` and `scm` sources
  will be copied to the root source directory, being the **self.folders.source** variable the way to describe if the fetched sources
  are still in a subfolder.
  It is used in the cache when running
  :command:`conan create` (relative to the cache source folder) as well as in a local folder when running :command:`conan build`
  (relative to the local current folder).

- **self.folders.build** (Defaulted to ""): Specifies a subfolder where the files from the build are. The ``self.build_folder`` attribute and
  the *current working directory* inside the ``build(self)`` method will be set with this subfolder. It is used in the cache when running
  :command:`conan create` (relative to the cache source folder) as well as in a local folder when running :command:`conan build`
  (relative to the local current folder).

- **self.folders.generators** (Defaulted to ""): Specifies a subfolder where to write the files from the generators and the toolchains.
  In the cache, when running the :command:`conan create`, this subfolder will be relative to the root build folder and when running
  the :command:`conan install` command it will be relative to the current working directory.

- **self.folders.imports** (Defaulted to ""): Specifies a subfolder where to write the files copied when using the ``imports(self)``
  method in a ``conanfile.py``. In the cache, when running the :command:`conan create`, this subfolder will be relative to the root
  build folder and when running the :command:`conan imports` command it will be relative to the current working directory.

Available since: `1.46.0 <https://github.com/conan-io/conan/releases/tag/1.46.0>`_

- **self.folders.root** (Defaulted to None): Specifies a parent directory where the sources, generators, etc., are located specifically when the ``conanfile.py`` is located in a separated subdirectory.

Available since: `1.51.0 <https://github.com/conan-io/conan/releases/tag/1.51.0>`_

- **self.folders.subproject** (Defaulted to None): Specifies a subfolder where the ``conanfile.py`` is relative to the project root. This is particularly useful for :ref:`layouts with multiple subprojects<package_layout_example_multiple_subprojects>`


self.cpp
++++++++

The ``layout()`` method allows to declare ``cpp_info`` objects not only for the final package (like the classic approach with
the ``self.cpp_info`` in the ``package_info(self)`` method) but for the ``self.source_folder`` and ``self.build_folder``.

The fields of the cpp_info objects at ``self.cpp.build`` and ``self.cpp.source`` are the same described :ref:`here<cpp_info_attributes_reference>`.
Components are also supported.


test()
------

The ``test()`` method is only used for *test_package/conanfile.py* recipes. It will execute immediately after ``build()`` has been called, and its goal is to
run some executable or tests on binaries to prove the package is correctly created. Note that it is intended to be used as a
test of the package: the headers are there, the libraries are there, it is possible to link, etc., but not to run unit, integration or functional tests.

It usually takes the form:

.. code:: python

    def test(self):
        if not tools.cross_building(self):
            self.run(os.path.sep.join([".", "bin", "example"]))


Note the ``tools.cross_building()`` check, as it is not possible to run executables different to the build machine architecture. In this case, it would
make sense to check the existence of the binary, or inspect it with tools like ``dumpbin``, ``lipo``, etc to do basic checks about it.

The ``self.run()`` might need some environment help, in case the execution needs for example shared libraries location.
