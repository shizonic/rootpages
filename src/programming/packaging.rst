Packaging
=========

.. contents:: Table of Contents

DEB (Debian)
------------

Official guides for building Debian packages:

-  https://wiki.debian.org/BuildingTutorial
-  https://www.debian.org/doc/manuals/maint-guide/
-  https://www.debian.org/doc/manuals/debmake-doc/ch04.en.html

A Debian package can be created by moving into the source code and
creating these files and/or directories.

-  **debian/** = The top-level directory for the package.

   -  **changelog** = Required. A change log listing the updates for the
      package release itself, not the program.
   -  **control** = Required. This describes the package name, version,
      maintainer, dependencies, and more.
   -  **copyright** = Required. The licenses used for the package and
      source code.
   -  install = Optional. A text file listing extra files (not installed
      by ``make install``) that should be copied into the package.
   -  **rules** = Required. A Makefile that explains how to build and
      install the program before being packaged. In a default
      environment setup by the ``dh_make`` command this essentially runs
      ``./configure; make; make install`` in the directory that contains
      the ``debian/`` directory. The shebang at the top of the file
      should normally be ``#!/usr/bin/make -f``.
   -  patches/ = Optional. Files for patching the source code.
   -  {preinst\|postinst\|prerm\|postrm} = Optional. These are
      executable scripts that run before installation, after
      installation, before removable, or after removable. [1]

Install the required packaging dependencies.

.. code-block:: sh

    $ sudo apt-get update
    $ sudo apt-get install devscripts dh-make dpkg-dev

Create a working build directory, download the source code, and then run
``dh_make``.

.. code-block:: sh

    $ mkdir build
    $ cd build
    $ curl -O http://<URL_TO_SOURCE_CODE>
    $ tar -v -x -z -f <PROGRAM_NAME>-<VERSION>.tar.gz
    $ cd <PROGRAM_NAME>-<VERSION>
    $ dh_make -f ../<PROGRAM_NAME>-<VERSION>.tar.gz

This will create a ``debian/`` directory inside of the source code
directory. With a template of all of the files required to build the
source code. A copy tarball of the source code is also created as
``<PROGRAM_NAME>_<VERSION>.orig.tar.gz`` in the ``build`` directory.

The DEB package can now be built.

.. code-block:: sh

    $ dpkg-buildpackage

After building the package, a new source tarball will be created
containing the ``debian`` directory:
``<PROGRAM_NAME>_<VERSION>-<DEB_PACKAGE_RELEASE>.debian.tar.gz``. The
actual package will be named
``<PACKAGE_NAME>_<VERSION>-<DEB_PACKAGE_RELEASE>_<ARCHITECTURE>.deb``.

``changelog`` File Syntax:

::

    <PACKAGE_NAME> (<PROGRAM_VERSION>-<PACKAGE_REVISION>) ; urgency=<URGENCY_LEVEL>

      * <PACKAGE_REVISION_NOTES>

     -- <AUTHOR_FIRST_NAME> <AUTHOR_LAST_NAME> <<EMAIL>>  <DAY>, <DAY_NUMBER> <MONTH> <YEAR> <HH>:<MM>:<SS> <UTC_HOUR_OFFSET>

``changelog`` File Example:

::

    apache (2.4.0-2) stable; urgency=low

      * Second release

     -- Bob Smith <bob@smith.tld>  Mon, 22 Mar 2017 00:01:00 +0200

    apache (2.4.0-1) stable; urgency=low

      * Initial release

     -- Bob Smith <bob@smith.tld>  Mon, 22 Mar 2017 23:12:12 +0100

``control`` File Example [2]:

::

    Source: hello-debian
    Section: utils
    Priority: extra
    Maintainer: Full Name <yourname@example.com>
    Build-Depends: debhelper (>= 8.0.0)
    Standards-Version: 3.9.3
    Vcs-Git: git@github.com:streadway/hello-debian.git
    Vcs-Browser: http://github.com/streadway/hello-debian

    Package: hello-debian
    Section: utils
    Priority: extra
    Architecture: any
    Depends: ${shlibs:Depends}, ${misc:Depends}
    Description: Example package maintenance (under 60 chars)
     The build output from a repository listing the steps to setup a debian
     package in a long-format under 80 characters per line.

Macros
~~~~~~

Many macros exist for helping to build and install Debian packages.

``rule`` macros:

-  dh\_auto\_clean = ``make distclean``
-  dh\_auto\_configure = ``./configure`` with directory options for the
   specific Debian release.
-  dh\_auto\_build = ``make``
-  dh\_auto\_test = ``make test``
-  dh\_auto\_install =

   .. code-block:: sh

       make install DESTDIR=/<PATH_TO_>/<PACKAGE>-<VERSION>-revision/debian/<PACKAGE>

[3]

RPM (Red Hat)
-------------

An RPM is built from a "spec" file. This modified shell script contains
all of the information about the program and on how to install and
uninstall it. It is used to build the RPM.

Common variables:

-  Name = The name of the program.

   -  ``%{name}``

-  Version = The version of the package. Typically this is in the format
   of X.Y.Z (major.minor.bugfix) or ISO date format (for example,
   "2016-01-01").

   -  ``%{version}``

-  Release = Start with "1%{?dist}" for the first build of the RPM.
   Increase the number if the package is ever rebuilt. Start from
   "1%{?dist}" if a new version of the actual program is being built.
-  Summary = One sentence describing the package. A period is not
   allowed at the end.
-  BuildRoot = The directory that contains all of the RPM packages. The
   directory structure under here should mirror the files location in
   relation to the top-level root "/". For example, "/bin/bash" would be
   placed under "$RPM\_BUILD\_ROOT/bin/bash".
-  BuildArch = The architecture that the program is meant to run on.
   This is generally either "x86\_64" or "i386." If the code is not
   dependent on the CPU (for example: Java programs, shell scripts,
   documentation, etc.) then "noarch" can be used.
-  Requires = List the RPM packages that are dependencies needed for
   your program to work.
-  License = The license of the program.
-  URL = A URL link to the program's or, if that is not available, the
   developer's website.
-  Source = A tarball of the source code. It should follow the naming
   standard of ``<RPM_NAME>-<RPM_PROGRAM_VERSION>.tar.gz``.

Sample SPEC file:

::

    Name: my-first-rpm
    Version: 1.0.0
    Release: 1%{?dist}
    Summary: This is my first RPM
    License: GPLv3
    URL: http://example.tld/

If you want to build the RPM, simply run:

.. code-block:: sh

    $ sudo rpmbuild -bb <SPECFILE>.spec

In case you also want to build a source RPM (SRPM) run:

.. code-block:: sh

    $ sudo rpmbuild -ba <SPECFILE>.spec

Sections:

-  ``%description`` = Provide a description of the program.
-  ``%prep`` = Define how to extract the source code for building.
-  ``%setup`` =
-  ``%build`` = This is where the program is built from the source code.
-  ``%install`` = Copy files to a directory structure under
   ``%{buildroot}`` that mirrors where their installed location. The
   ``%{buildroot}`` is the top-level directory of a typical Linux file
   system hierarchy.
-  ``%file`` = These are the files that should be copied over during
   installation. Permissions can also be set.

   -  ``%attr(<MODE>, <USER>, <GROUP>)`` = Define this in front of a
      file or folder to give it custom permissions.

[4]

Macros
~~~~~~

Macros are variables in the RPM spec file that are expanded upon
compilation of the RPM.

Some useful macros include:

-  ``%{patches}`` = An array of all of the defined patch files.
-  ``%{sources}`` = An array of all of the defined source files.

[5]

Directories
~~~~~~~~~~~

During the creation of an RPM there are a few important directories that
can and will be referenced.

-  %{topdir} = The directory that the RPM related files should be
   located. By default this is set to ``%{getenv:HOME}/rpmbuild``.
-  %{builddir} = The ``%{_topdir}/BUILD`` directory. This is where the
   compilation of the program should take place.
-  %{\_sourcedir} = The ``%{_topdir}/SOURCES`` directory. This is where
   patches, service files, and source code can be stored.
-  %{\_specdir} = The ``%{_topdir}/SPECS`` directory. This is where the
   SPEC file for the RPM should be stored.
-  %{\_srcrpmdir} = The ``%{_topdir}/SRPMS`` directory. This is where
   the optional source RPM will be compiled and stored to.
-  %{buildroot} = The ``%{_topdir}/BUILDROOT`` directory. This is the
   file system hierarchy of where the RPM files will actually be
   installed to. This is also set to the ``$RPM_BUILD_ROOT`` shell
   variable.

[6]

Users and Groups
~~~~~~~~~~~~~~~~

Creating a user or group can be done one of two ways.

-  Dynamically = Let the system decide what user identification number
   (UID) and group ID (GID) to use.
-  Static = Specify a specific UID or GID number to use. This is useful
   for keeping permissions identical on multiple platforms.

The Fedora Project recommends using these standardized blocks of code to
accomplish these methods. [7]

Dynamic:

::

    Requires(pre): shadow-utils
    [...]
    %pre
    getent group <GROUP_NAME> >/dev/null || groupadd -r <GROUP_NAME>
    getent passwd <USER_NAME> >/dev/null || \
        useradd -r -g <GROUP_NAME> -s /sbin/nologin \
        -c "<USER_DESCRIPTION>" <USER_NAME>
    exit 0

Static:

::

    Requires(pre): shadow-utils
    <OMITTED>
    %pre
    getent group <GROUP_NAME> >/dev/null || groupadd -f -g <GID> -r <GROUP_NAME>
    if ! getent passwd <USER_NAME> >/dev/null ; then
        if ! getent passwd <UID> >/dev/null ; then
          useradd -r -u <UID> -g <GROUP_NAME> -s /sbin/nologin -c "Useful comment about the purpose of this account" <USER_NAME>
        else
          useradd -r -g <GROUP_NAME> -s /sbin/nologin -c "<USER_DESCRIPTION>" <USER_NAME>
        fi
    fi
    exit 0

Patches
~~~~~~~

Some applications may require patches to work properly. Patches should
be stored in the ``SOURCES`` directories. At the beginning of the spec
file, where the name and version information is defined, patch file
names can also be defined.

Usage:

::

    Patch<NUMBER>: <PATCH_FILE>

Example:

::

    Patch0: php-fpm_listen_port.patch
    Patch1: php_memory_limit.patch

These patches can then be referenced in the ``%setup`` phase (after
``%prep`` and before ``%build%``).

::

    %setup -q

A patched file can be created using the ``diff`` command.

.. code-block:: sh

    $ diff -u <ORIGINAL_FILE> <PATCHED_FILE> > <PATCH_NAME>.patch

If multiple files in a directory have been patched, a more comprehensive
patch file can be made.

.. code-block:: sh

    $ diff -urN <ORIGINAL_SOURCE_DIRECTORY>/ <PATCHED_SOURCE_DIRECTORY>/ > <PATCH_NAME>.patch

In the spec file, the ``%patch`` macro can be used. The ``-p1`` argument
strips off the top-level directory of the patch's path.

Syntax:

::

    %patch0 -p1
    %patch1 -p1

Example patch file:

::

    --- d20-1.0.0_patched/src/dice.h
    +++ d20-1.0.0/src/dice.h

A patch can also be made without the ``%patch`` macro by specifying the
location of the patch file.

.. code-block:: sh

    patch < %{_sourcedir}/<FILE_NAME>

[8]

Troubleshooting
~~~~~~~~~~~~~~~

Error Messages
^^^^^^^^^^^^^^

-  The ``custom_macro`` macro does not exist. Find and install it to ``/usr/lib/rpm/macros.d/``.
-  Alternatively, if the message complains about a native macro instead, it could be used in the wrong section.

.. code-block:: sh

   $ rpmbuild

::

   + echo foo bar
   + %custom_macro
   /var/tmp/rpm-tmp.0Sev9I: line 324: fg: no job control
   error: Bad exit status from /var/tmp/rpm-tmp.0Sev9I (%prep)

[12]

PKGBUILD (Arch Linux)
---------------------

Arch Linux packages are design to be simple and easy to create. A
PKGBUILD file is compressed with a software's contents into a XZ
tarball. This can contain either the source code or compiled program.

Required Variables:

-  pkgname = Name of the software.
-  pkgver = Version of the software.
-  pkgrel = Version of the package (only increase if the PKGBUILD file
   has been modified and not the software).
-  arch = The architecture the software is built for. Any architecture
   that applies should be defined. Valid options: x86\_64, i686, arm
   (armv5), armv6h, armv7h, aarch64 (armv8 64-bit), or any.

Optional Variables:

-  pkgdesc = A brief description of the software.
-  url = The URL of the software's website.
-  license = The license of the software. Valid options: GPL, BSD, MIT,
   Apache, etc.
-  depends = List other package version dependencies.
-  optdepends = List optional dependencies and a brief description.
-  makedepends = List packages required to build the software from
   source.
-  provides = List tools that are provided by the package but do not
   necessarily have file names.
-  conflicts = List any conflicting packages.
-  replaces = List packages that this software should replace.

[9]

Functions

Required:

-  build()

   -  For building the software, PKGBUILD will need to move into the
      directory that the XZ tarball was extracted to. This is
      automatically generated as the "srcdir" variable. In most
      situations this should be the package name and version separated
      by a dash.

      .. code-block:: sh

          $ cd "${srcdir}"

      OR

      .. code-block:: sh

          $ cd "${pkgname}-${pkgver}"

-  package()

   -  These are the steps to copy and/or modify files from the "srcdir"
      to be placed in the "pkgdir" to represent where they will be
      installed on an end-user's system. This acts as the top-level
      directory of a Linux file system hierarchy.

      .. code-block:: sh

          $ cd "${pkgdir}"

   -  An example of installing compiled source code using a Make file.

      .. code-block:: sh

          $ make DESTDIR="${pkgdir}" install

[10][11]

AUR Submission
~~~~~~~~~~~~~~

The Arch Linux User (AUR) repository allows developers to easily upload their own packages. Here are the steps on how to submit a new package to the AUR.

**SSH Key Pair**

Create a unique SSH key pair to use for interacting with the AUR.

.. code-block:: sh

   $ ssh-keygen -t ed25519 -b 4096 -f ~/.ssh/aur
   $ vim ~/.ssh/config
   Host aur.archlinux.org
     IdentityFile ~/.ssh/aur
     User aur

**Create the AUR Git Repository**

Clone a repository with the desired AUR package name. Once files are committed and pushed, this package will be instantly available on the AUR.

.. code-block:: sh

   $ git clone ssh://aur@aur.archlinux.org/<NEW_AUR_PACKAGE_NAME>.git

**Files**

Every AUR git repository needs to contain at least 2 files:

-  PKGBUILD = The PKGBUILD explains how to download and build the source code.
-  .SRCINFO = Information about what packages the PKGBUILD will provide. Generate this by running ``makepkg --printsrcinfo > .SRCINFO``.

    -  Every time the PKGBUILD metadata has been updated, this file needs to be regenerated and committed to the git repository.

Optional files:

-  .gitignore = Ignore build files and directories such as ``pkg`` and ``src``.
-  LICENSE = The license for the PKGBUILD. This is generally the same as the software that it builds.

There should not be any binary or source code hosted in the AUR git repository.

[13]

History
-------

-  `Latest <https://github.com/LukeShortCloud/rootpages/commits/main/src/programming/packaging.rst>`__
-  `< 2019.07.01 <https://github.com/LukeShortCloud/rootpages/commits/main/src/administration/packages.rst>`__
-  `< 2019.01.01 <https://github.com/LukeShortCloud/rootpages/commits/main/src/packages.rst>`__
-  `< 2018.01.01 <https://github.com/LukeShortCloud/rootpages/commits/main/markdown/packages.md>`__

Bibliography
------------

1. "Chapter 7 - Basics of the Debian package management system." The Debian GNU/Linux FAQ. August 28, 2016. Accessed March 25, 2017. https://www.debian.org/doc/manuals/debian-faq/ch-pkg\_basics.en.html
2. "hello-debian README.md." streadway/hello-debian GitHub. March 24, 2014. Accessed May 8, 2017. https://github.com/streadway/hello-debian
3. "Chapter 4. Required files under the debian directory." Debian New Maintainers' Guide. February 25, 2017. Accessed March 24, 2017. https://www.debian.org/doc/manuals/maint-guide/dreq.en.html
4. "How to create an RPM package." Fedora Project. June 22, 2016. Accessed June 28, 2016. http://fedoraproject.org/wiki/How\_to\_create\_an\_RPM\_package
5. "Creating RPM packages." Fedora Docs Site. May 16, 2020. Accessed May 16, 2020. https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html
6. "Packaging:RPMMacros." Fedora Project Wiki. December 1, 2016. Accessed March 13, 2017. https://fedoraproject.org/wiki/Packaging:RPMMacros?rd=Packaging/RPMMacros
7. "Packaging: Users and Groups" Fedora Project. September 14, 2016. Accessed February 25, 2017. https://fedoraproject.org/wiki/Packaging:UsersAndGroups
8. "How to Create and Use Patch Files for RPM Packages." Bob Cromwell. March 20, 2017. Accessed March 20, 2017. http://cromwell-intl.com/linux/rpm-patch.html
9. "PKGBUILD." Arch Linux Wiki. October 26, 2016. Accessed November 19, 2016. https://wiki.archlinux.org/index.php/PKGBUILD
10. "Creating packages." Arch Linux Wiki. July 30, 2016. Accessed November 19, 2016. https://wiki.archlinux.org/index.php/creating\_packages
11. "PKGBUILD(5) Manual Page." Arch Linux Man Pages. February 26, 2016. Accessed November 19, 2016. https://www.archlinux.org/pacman/PKGBUILD.5.html
12. "RPM spec patch application fails." Stack Overflow. August 22, 2016. Accessed March 27, 2020. https://stackoverflow.com/questions/39052950/rpm-spec-patch-application-fails
13. "AUR submission guidelines." Arch Linux Wiki. February 20, 2022. Accessed April 5, 2022. https://wiki.archlinux.org/title/AUR_submission_guidelines
