.. _CMakeDeps:

CMakeDeps
---------

.. warning::

    These tools are still **experimental** (so subject to breaking changes) but with very stable syntax.
    We encourage the usage of it to be prepared for Conan 2.0.


Available since: `1.33.0 <https://github.com/conan-io/conan/releases/tag/1.33.0>`_

The ``CMakeDeps`` helper will generate one **xxxx-config.cmake** file per dependency, together with other necessary *.cmake* files
like version, flags and directory data or configuration. It can be used like:


.. code-block:: python

    from conan import ConanFile

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"
        requires = "hello/0.1"
        generators = "CMakeDeps"


The full instantiation, that allows custom configuration can be done in the ``generate()`` method:


.. code-block:: python

    from conan import ConanFile
    from conan.tools.cmake import CMakeDeps

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"
        requires = "hello/0.1"

        def generate(self):
            cmake = CMakeDeps(self)
            cmake.generate()


.. important::

    This class will require very soon to define both the "host" and "build" profiles. It is very recommended to
    start defining both profiles immediately to avoid future breaking. Furthermore, some features, like trying to
    cross-compile might not work at all if the "build" profile is not provided.


The ``CMakeDeps`` is a *multi-configuration* generator, it can correctly create files for Release/Debug configurations
to be simultaneously used by IDEs like Visual Studio. In single configuration environments, it is necessary to have
a configuration defined, which must be provided via the ``cmake ... -DCMAKE_BUILD_TYPE=<build-type>`` argument in command line
(Conan will do it automatically when necessary, in the ``CMake.configure()`` helper).


There are some attributes you can adjust in the created ``CMakeDeps`` object to change the default behavior:

configuration
++++++++++++++

Allows to define custom user CMake configuration besides the standard Release, Debug, etc ones.

.. code-block:: python

    def generate(self):
        deps = CMakeDeps(self)
        # By default, ``deps.configuration`` will be ``self.settings.build_type``
        if self.options["hello"].shared:
            # Assuming the current project ``CMakeLists.txt`` defines the ReleasedShared configuration.
            deps.configuration = "ReleaseShared"
        deps.generate()


build_context_activated
+++++++++++++++++++++++

When you have a **build-require**, by default, the config files (`xxx-config.cmake`) files are not generated.
But you can activate it using the **build_context_activated** attribute:

.. code-block:: python

    tool_requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the tool require
        cmake.build_context_activated = ["my_tool"]
        cmake.generate()

.. warning::

    The ``build_context_activated`` feature will fail if no "build" profile is used. This feature only work when using
    the two host and build profiles.


build_context_suffix
++++++++++++++++++++

When you have the same package as a **build-require** and as a **regular require** it will cause a conflict in the generator
because the file names of the config files will collide as well as the targets names, variables names etc.

For example, this is a typical situation with some requirements (capnproto, protobuf...) that contain
a tool used to generate source code at build time (so it is a **build_require**),
but also providing a library to link to the final application, so you also have a **regular require**.
Solving this conflict is specially important when we are cross-building because the tool
(that will run in the building machine) belongs to a different binary package than the library, that will "run" in the
host machine.

You can use the **build_context_suffix** attribute to specify a suffix for a requirement,
so the files/targets/variables of the requirement in the build context (tool require) will be renamed:

.. code-block:: python

    tool_requires = ["my_tool/0.0.1"]
    requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the tool require
        cmake.build_context_activated = ["my_tool"]
        # disambiguate the files, targets, etc
        cmake.build_context_suffix = {"my_tool": "_BUILD"}
        cmake.generate()


.. warning::

    The ``build_context_suffix`` feature will fail if no "build" profile is used. This feature only work when using
    the two host and build profiles.


build_context_build_modules
+++++++++++++++++++++++++++

Also there is another issue with the **build_modules**. As you may know, the recipes of the requirements can declare a
`cppinfo.build_modules` entry containing one or more **.cmake** files.
When the requirement is found by the cmake ``find_package()``
function, Conan will include automatically these files.

By default, Conan will include only the build modules from the
``host`` context (regular requires) to avoid the collision, but you can change the default behavior.

Use the **build_context_build_modules** attribute to specify require names to include the **build_modules** from
**tool_requires**:

.. code-block:: python

    tool_requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the tool require
        cmake.build_context_activated = ["my_tool"]
        # Choose the build modules from "build" context
        cmake.build_context_build_modules = ["my_tool"]
        cmake.generate()


.. warning::

    The ``build_context_build_modules`` feature will fail if no "build" profile is used. This feature only work when using
    the two host and build profiles.

.. _CMakeDeps Properties:

Properties
++++++++++

The following properties affect the CMakeDeps generator:

- **cmake_file_name**: The config file generated for the current package will follow the ``<VALUE>-config.cmake`` pattern,
  so to find the package you write ``find_package(<VALUE>)``.
- **cmake_target_name**: Name of the target to be consumed.
- **cmake_target_aliases**: List of aliases that Conan will create for an already existing target.
- **cmake_find_mode**: Defaulted to ``config``. Possible values are:

  - ``config``: The CMakeDeps generator will create config scripts for the dependency.
  - ``module``: Will create module config (FindXXX.cmake) scripts for the dependency.
  - ``both``: Will generate both config and modules.
  - ``none``: Won't generate any file. It can be used, for instance, to create a system wrapper package so the consumers find the config files in the CMake installation config path and not in the generated by Conan (because it has been skipped).

- **cmake_module_file_name**: Same as **cmake_file_name** but when generating modules with ``cmake_find_mode=module/both``. If not specified it will default to **cmake_file_name**.
- **cmake_module_target_name**: Same as **cmake_target_name**  but when generating modules with ``cmake_find_mode=module/both``.  If not specified it will default to **cmake_target_name**.
- **cmake_build_modules**: List of ``.cmake`` files (route relative to root package folder) that are automatically
  included when the consumer run the ``find_package()``. This property can't be declared in a component, do it in the global ``self.cpp_info``.
- **cmake_set_interface_link_directories**: boolean value that should be only used by dependencies that don't declare `self.cpp_info.libs` but have ``#pragma comment(lib, "foo")`` (automatic link) declared at the public headers. Those dependencies should
  add this property to their *conanfile.py* files at root ``cpp_info`` level (components not supported for now).

Example:

.. code-block:: python

    def package_info(self):
        ...
        # MyFileName-config.cmake
        self.cpp_info.set_property("cmake_file_name", "MyFileName")
        # Names for targets are absolute, Conan won't add any namespace to the target names automatically
        self.cpp_info.set_property("cmake_target_name", "Foo::Foo")

        # Create a new target "MyFooAlias" that is an alias to the "Foo::Foo" target
        self.cpp_info.set_property("cmake_target_aliases", ["MyFooAlias"])
        # The property "cmake_build_modules" can't be declared in a component, do it in self.cpp_info
        self.cpp_info.set_property("cmake_build_modules", [os.path.join("lib", "mypkg.cmake")])

        self.cpp_info.components["mycomponent"].set_property("cmake_target_name", "Foo::Var")

        # Create a new target "VarComponent" that is an alias to the "Foo::Var" component target
        self.cpp_info.components["mycomponent"].set_property("cmake_target_aliases", ["VarComponent"])

        # Skip this package when generating the files for the whole dependency tree in the consumer
        # note: it will make useless the previous adjustments.
        # self.cpp_info.set_property("cmake_find_mode", "none")

        # Generate both MyFileNameConfig.cmake and FindMyFileName.cmake
        self.cpp_info.set_property("cmake_find_mode", "both")
        

.. _Disable CMakeDeps For Installed CMake imports:

Disable CMakeDeps For Installed CMake configuration files
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Some projects may want to disable the ``CMakeDeps`` generator for downstream consumers. This can be done by settings ``cmake_find_mode`` to ``none``.
If the project wants to provide it's own configuration targets, it should append them to the ``buildirs`` attribute of ``cpp_info``.

This method is intended to work with downstream consumers using the ``CMakeToolchain`` generator, which will be populated with the ``builddirs`` attribute.

Example:

.. code-block:: python

    def package(self):
        ...
        cmake.install()

    def package_info(self):
        self.cpp_info.set_property("cmake_find_mode", "none") # Do NOT generate anyfiles
        self.cpp_info.builddirs.append(os.path.join("lib", "cmake", "foo"))
