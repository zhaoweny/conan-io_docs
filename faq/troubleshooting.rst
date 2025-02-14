Troubleshooting
==================

ERROR: The recipe is constraining settings
------------------------------------------

When you install or create a package you might have error like the following one:

.. code-block:: text

    ERROR: The recipe wtl/10.0.9163 is constraining settings. Invalid setting 'Linux' is not a valid 'settings.os' value.
    Possible values are ['Windows']
    Read "http://docs.conan.io/en/latest/faq/troubleshooting.html#error-the-recipe-is-contraining-settings"

This means that your target operating system is not supported by the recipe.

ERROR: Missing prebuilt package
--------------------------------

When installing packages (with :command:`conan install` or :command:`conan create`) it is possible
that you get an error like the following one:

.. code-block:: text

     WARN: Can't find a 'czmq/4.2.0' package for the specified settings, options and dependencies:
    - Settings: arch=x86_64, build_type=Release, compiler=Visual Studio, compiler.runtime=MD, compiler.version=16, os=Windows
    - Options: shared=False, with_libcurl=True, with_libuuid=True, with_lz4=True, libcurl:shared=False, ...
    - Dependencies: openssl/1.1.1d, zeromq/4.3.2, libcurl/7.67.0, lz4/1.9.2
    - Requirements: libcurl/7.Y.Z, lz4/1.Y.Z, openssl/1.Y.Z, zeromq/4.Y.Z
    - Package ID: 7a4079899e0893ca670df1f682b4606abe79ee5b

    ERROR: Missing prebuilt package for 'czmq/4.2.0'
    Try to build it from sources with '--build czmq'
    Use 'conan search <reference> --table table.html'
    Or read 'http://docs.conan.io/en/latest/faq/troubleshooting.html#error-missing-prebuilt-package'

This means that the package recipe ``czmq/4.2.0@`` exists, but for some reason
there is no precompiled package for your current settings. Maybe the package creator didn't build
and shared pre-built packages at all and only uploaded the package recipe, or they are only
providing packages for some platforms or compilers. E.g. the package creator built packages
from the recipe for Visual Studio 14 and 15, but you are using Visual Studio 16. Also you may want to check your
:ref:`package ID mode <default_package_id_mode>` as it may have an influence on the packages available for it.

By default, Conan doesn't build packages from sources. There are several possibilities to overcome this error:

- You can try to build the package for your settings from sources, indicating some build policy as argument, like :command:`--build czmq` or
  :command:`--build missing`. If the package recipe and the source code work for your settings you will have your binaries built locally and
  ready for use.

- If building from sources fails, you might want to fork the original recipe, improve it until it
  supports your configuration, and then use it. Most likely contributing back to the original
  package creator is the way to go. But you can also upload your modified recipe and pre-built
  binaries under your own username too.


.. _error_invalid_setting:

ERROR: Invalid setting
------------------------

It might happen sometimes, when you specify a setting not present in the defaults
that you receive a message like this:

.. code-block:: bash

    $ conan install . -s compiler.version=4.19 ...

    ERROR: Invalid setting '4.19' is not a valid 'settings.compiler.version' value.
    Possible values are ['4.4', '4.5', '4.6', '4.7', '4.8', '4.9', '5.1', '5.2', '5.3', '5.4', '6.1', '6.2']
    Read "http://docs.conan.io/en/latest/faq/troubleshooting.html#error-invalid-setting"


This doesn't mean that such architecture is not supported by conan, it is just that it is not present in the actual
defaults settings. You can find in your user home folder ``~/.conan/settings.yml`` a settings file that you
can modify, edit, add any setting or any value, with any nesting if necessary. See :ref:`custom_settings`.

As long as your team or users have the same settings (you can share with them the file), everything will work. The *settings.yml* file is just a
mechanism so users agree on a common spelling for typical settings. Also, if you think that some settings would
be useful for many other conan users, please submit it as an issue or a pull request, so it is included in future
releases.

It is possible that some build helper, like ``CMake`` will not understand the new added settings,
don't use them or even fail.
Such helpers as ``CMake`` are simple utilities to translate from conan settings to the respective
build system syntax and command line arguments, so they can be extended or replaced with your own
one that would handle your own private settings.

ERROR: Setting value not defined
---------------------------------

When you install or create a package, it is possible to see an error like this:

.. code-block:: bash

    ERROR: hello/0.1@user/testing: 'settings.arch' value not defined

This means that the recipe defined ``settings = "os", "arch", ...`` but a value for the ``arch`` setting was
not provided either in a profile or in the command line. Make sure to specify a value for it in your profile,
or in the command line:

.. code-block:: bash

    $ conan install . -s arch=x86 ...

If you are building a pure C library with gcc/clang, you might encounter an error like this:

.. code-block:: bash

    ERROR: hello/0.1@user/testing: 'settings.compiler.libcxx' value not defined

Indeed, for building a C library, it is not necessary to define a C++ standard library. And if you provide a value,
you might end with multiple packages for exactly the same binary. What has to be done is to remove such subsetting
in your recipe:


.. code-block:: python

    def configure(self):
        del self.settings.compiler.libcxx


ERROR: Failed to create process
--------------------------------

When conan is installed via pip/PyPI, and python is installed in a path with spaces (like many times in Windows "C:/Program Files..."), conan can fail to launch. This is a known python issue, and can't be fixed from conan.
The current workarounds would be:

- Install python in a path without spaces
- Use virtualenvs. Short guide:

.. code-block:: bash

    $ pip install virtualenvwrapper-win # virtualenvwrapper if not Windows
    $ mkvirtualenv conan
    (conan) $ pip install conan
    (conan) $ conan --help

Then, when you will be using conan, for example in a new shell, you have to activate the virtualenv:

.. code-block:: bash

    $ workon conan
    (conan) $ conan --help

Virtualenvs are very convenient, not only for this workaround, but to keep your system clean and to avoid unwanted interaction between different tools and python projects.


ERROR: Failed to remove folder (Windows)
-----------------------------------------
It is possible that operating conan, some random exceptions (some with complete tracebacks) are produced, related to the impossibility to remove one folder. Two things can happen:

- The user has some file or folder open (in a file editor, in the terminal), so it cannot be removed, and the process fails. Make sure to close files, specially if you are opening or inspecting the local conan cache.
- In Windows, the Search Indexer might be opening and locking the files, producing random, difficult to reproduce and annoying errors. Please **disable the Windows Search Indexer for the conan local storage folder**


ERROR: Error while initializing Options
---------------------------------------

When installing a Conan package and the follow error occurs:

.. code-block:: bash

    ERROR: conanfile.py: Error while initializing options. Please define your default_options as list or multiline string

Probably your Conan version is outdated.
The error is related to `default_options` be used as dictionary and only can be handled by Conan >= 1.8.
To fix this error, update Conan to 1.8 or higher.


ERROR: Error while starting Conan Server with multiple workers
--------------------------------------------------------------

When running ``gunicorn`` to start ``conan_server`` in an empty environment:

.. code-block:: bash

    $ gunicorn -b 0.0.0.0:9300 -w 4 -t 300 conans.server.server_launcher:app

        **********************************************
        *                                            *
        *      ERROR: STORAGE MIGRATION NEEDED!      *
        *                                            *
        **********************************************
        A migration of your storage is needed, please backup first the storage directory and run:

        $ conan_server --migrate

Conan Server will try to create `~/.conan_server/data`, `~/.conan_server/server.conf` and `~/.conan_server/version.txt` at first time.
However, as multiple workers are running at same time, it could result in a conflict.
To fix this error, you should run:

.. code-block:: bash

    $ conan_server --migrate

This command must be executed before to start the workers. It will not migrate anything, but it will populate the conan_server folder.
The original discussion about this error is `here <https://github.com/conan-io/conan/issues/4723>`_.


ERROR: Requested a package but found case incompatible
------------------------------------------------------

When installing a package which is already installed, but using a different case, will result on the follow error:

.. code-block:: bash

    $ conan install poco/1.10.1@

        [...]
        ERROR: Failed requirement 'openssl/1.0.2t' from 'poco/1.10.1@'
        ERROR: Requested 'openssl/1.0.2t', but found case incompatible recipe with name 'OpenSSL' in the cache. Case insensitive filesystem can not manage this
        Remove existing recipe 'OpenSSL' and try again.

You can find and use recipes with upper and lower case names (we encourage lowercase variants), but some OSs like
Windows are case insensitive by default, they cannot store at the same time both variants in the Conan cache. To
solve this problem you need to remove existing upper case variant ``OpenSSL``:

.. code-block:: bash

    $ conan remove "OpenSSL/*"


ERROR: Incompatible requirements obtained in different evaluations of 'requirements'
------------------------------------------------------------------------------------

When two different packages require the same package as a dependency, but with different versions, will result in the following error:

.. code-block:: bash

    $ cat conanfile.txt

    [requires]
    baz/1.0.0
    foobar/1.0.0

    $ conan install conanfile.txt

    [...]
    WARN: foobar/1.0.0: requirement foo/1.3.0 overridden by baz/1.0.0 to foo/1.0.0
    ERROR: baz/1.0.0: Incompatible requirements obtained in different evaluations of 'requirements'
    Previous requirements: [foo/1.0.0]
    New requirements: [foo/1.3.0]

As we can see in the following situation: the ``conanfile.txt`` requires 2 packages (``baz/1.0.0`` and ``foobar/1.0.0``) which
both require the package named ``foo``. However, ``baz`` requires ``foo/1.0.0``, but ``foobar`` requires ``foo/1.3.0``.
As the required versions are different, it's considered a conflict and Conan will not solve it.

To solve this kind of collision, you have to choose a version for ``foo`` and add it to the ``conanfile.txt`` as an explicit
requirement:

.. code-block:: text

    [requires]
    foo/1.3.0
    baz/1.0.0
    foobar/1.0.0

Here we choose ``foo/1.3.0`` because is newer. Now we can proceed:

.. code-block:: bash

    $ conan install conanfile.txt

        [...]
        WARN: baz/1.0.0: requirement foo/1.0.0 overridden by foobar/1.0.0 to foo/1.3.0

Conan still warns us about the conflict, but as we have :ref:`versioning_dependencies_overriding` the ``foo`` version, it's no longer an error.
