[//]: # (# Debian Binary Packaging System)

As someone managing a Debian system, you'll frequently manage `.deb` packages. These packages house cohesive functional components like applications and documentation, streamlining their installation and maintenance. Thus, having an understanding of these packages and their usage is valuable.

Debian packages are created to simplify software installation, removal, and management on Debian-based systems. They can be installed, upgraded, or removed using package management tools like `apt` or `dpkg`, which handle dependencies, ensuring that all required components for the software to run are properly installed.

## Structure of a Binary Package

Debian packages can be extracted on any Unix system equipped with fundamental commands like `ar`, `tar`, and `xz` (or sometimes `gzip` or `bzip2`). This feature is important for both portability and disaster recovery.

Let's have a look at `.deb` file

```bash
$ ar x zoom_amd64.deb 
$ ls
control.tar.xz  data.tar.xz  debian-binary  zoom_amd64.deb
$ tar tJf data.tar.xz  | head -n 10
./
./opt/
./opt/zoom/
./opt/zoom/Beep-intercom.pcm
./opt/zoom/Droplet.pcm
./opt/zoom/Embedded.properties
./opt/zoom/Qt/
./opt/zoom/Qt/bin/
./opt/zoom/Qt/bin/qt.conf
./opt/zoom/Qt/lib/
$ tar tJf control.tar.xz
./
./control
./postinst
./postrm
./md5sums
$ cat debian-binary
2.0
```

**debian-binary** This file contains the version of the `.deb` file
**control.tar.xz** This archive file contains meta-information, like the name and version of the package as well as some scripts to run before, during or after installation
**data.tar.xz, data.tar.bz2 data.tar.gz** contains the files to be extracted from the package. Packages may use different compression formats (bzip2, xz, gzip)

## Package Meta-Information

### The control file

Debian packages contains meta-information  about requisites, dependencies, conflicts and suggestions. It also contains scripts that enable the execution of commands at different stages in the package’s lifecycle (installation, upgrade, removal).
For example, for `zoom` package, the control file looks like:

```bash
$ apt-cache show zoom
Package: zoom
Status: install ok installed
Priority: optional
Section: default
Installed-Size: 665727
Maintainer: Zoom Linux Team <https://support.zoom.us>
Architecture: amd64
Version: 5.15.11.7239
Depends: libglib2.0-0, libxcb-keysyms1, libxcb-xinerama0, libdbus-1-3, libxcb-shape0, libxcb-shm0, libxcb-xfixes0, libxcb-randr0, libxcb-image0, libfontconfig1, libgl1-mesa-glx, libegl1-mesa, libxi6, libsm6, libxrender1, libpulse0, libxcomposite1, libxslt1.1, libsqlite3-0, libxcb-xtest0, libxtst6, ibus, libxkbcommon-x11-0, desktop-file-utils, libgbm1, libdrm2, libxcb-cursor0, libxcb-icccm4, libfreetype6 (>= 2.6)
Description: Zoom Cloud Meetings 
 Zoom brings people together to connect and get more done in a frictionless, secure video environment. Our easy, reliable, and innovative video-first solutions provide video meetings and chat, with additional options for webinars and phone service. 
 .
 Zoom is the leading unified communications platform and helps individuals, schools, healthcare professionals and enterprises stay connected. Visit blog.zoom.us and follow @zoom_us. 
 .
 By installing this app, you agree to our Terms of Service (https://zoom.us/terms) and Privacy Statement (https://zoom.us/privacy).
Description-md5: 16fd71141117b0ac78f38bb0054de755
License: see https://www.zoom.us/
Vendor: Zoom Video Communications, Inc.
Homepage: https://www.zoom.us
```

#### Depends filed

The Depends field in the package header defines necessary conditions for a package to function correctly, used by tools like `apt` to install required tools, drivers, libraries etc.

- The range of versions for each dependency can be restricted, such as requiring the package `libc6` in a version equal to or greater than "`2.15`" (written as "`libc6 (>=2.15)`").
- The comma in a list of conditions must be interpreted as a logical "and", while the vertical bar ("`|`") represents a logical "or" operator with a higher priority.
  - (`A` or `B`) and `C` is written as `A | B, C`
  - `A` or (`B` and `C`) is written as `A | B, A | C`

For more details on the syntax of the Depends Filed see: [https://www.debian.org/doc/debian-policy/ch-relationships.html](https://www.debian.org/doc/debian-policy/ch-relationships.html)

The dependencies system  has another use with **meta-packages**. These are empty packages that only describe dependencies. They facilitate the installation of a consistent group of programs preselected by the meta-package maintainer; as such, `apt install meta-package` will automatically install all of these programs using the meta-package’s dependencies. *gnome*, *kde-full* and *linux- image-amd64* packages are examples of meta-packages.

#### Recommends and Suggests fields

The Recommends and Suggests fields indicate non-compulsory dependencies. The **recommended dependencies** enhance package functionality but aren't essential to its operation.  The **suggested dependencies** indicate that certain packages may complement and increase utility.
>You should install recommended packages unless you know why they're not needed.
>Conversely, it is not necessary to install “suggested” packages unless you know why you need them.

#### Conflicts filed

The Conflicts field indicates when a package cannot be installed simultaneously with another due to common reasons like having the same file, service, or operation. `dpkg` will refuse to install a package if it triggers a conflict with an already installed one, except if the new package specifies to replace the existing one.

#### Breaks field

The Breaks field signals that the installation of a package will “break” another package (or particular versions of it). `dpkg` will refuse to install a package that breaks an already installed package, and `apt` will try to resolve the problem by updating the package that would be broken to a newer version (which is assumed to be fixed and, thus, compatible again).

#### Provides filed

This filed introduces the concept of **virtual-package** which associates a generic *service* with it and indicates that a package completely replaces another.

It is essential to clearly distinguish meta-packages from virtual packages. The former are real packages (including real .deb files), whose only purpose is to express dependencies. Virtual packages, however, do not exist physically; they are only a means of identifying real packages based on common, logical criteria (service provided, compatibility with a standard program or a pre-existing package, etc.).

Let's look at two examples:

##### Example 1

All mail servers, such as *postfix* or *sendmail* provide the `mail-transport-agent` virtual package. Thus, any package that needs this service to be functional (e.g. `smartlist` or `sympa`) simply states in its dependencies that it requires a `mail-transport-agent` instead of specifying a large yet incomplete list of possible solutions (e.g. p`ostfix | sendmail | exim4 | …`). Furthermore,  each of these packages declares a conflict with the `mail-transport-agent` virtual package prohibiting the installation of two mail servers side by side.

##### Example 2

The Provides field is useful when a package's content is included in a larger package. For instance, the `libdigest-md5-perl` Perl module, which was optional in Perl 5.6, has been integrated as standard in Perl 5.8 (and new versions). Since then, the package perl has declared `Provides: libdigest-md5-perl`, so that the dependencies on this package are met if the user has Perl 5.8 (or newer)

#### Replaces filed

The Replaces field certifies that although the package has files that are identical to those in another package, it has the right to replace them. The lack of this specification causes `dpkg` to fail, indicating that it cannot overwrite the files of another package (technically, it is possible to force it to do so with the `--force-overwrite` option, but that is not considered standard operation).

### Debian Confifile

A conffile refers to a configuration file associated with a package. These files typically reside in the `/etc` directory or its subdirectories. Conffiles store customizable settings for a particular software package.

What distinguishes conffiles from regular files is how package managers handle them during package upgrades. When a package containing a conffile is updated, and if the original version of the conffile has been modified by the user, the package manager doesn't overwrite the user's changes automatically. Instead, it presents a prompt to the user during the upgrade process, asking whether to keep the current version of the conffile, install the new version, or show a side-by-side comparison to help the user make a decision.

This approach ensures that the user's custom configurations are preserved during updates, allowing for a more controlled and user-centric management of configuration files within the Debian ecosystem.

### Configuration Scripts

The control.tar.gz archive for each Debian package may contain scripts called by `dpkg` at different stages in package processing. [The Debian Policy](https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html) details possible cases, specifying scripts and arguments. If a script fails, `dpkg` attempts to return to a satisfactory state by canceling installation or removal in progress.

The `preinst` script is executed prior to installation of the package, while `postinst` follows it. Likewise, `prerm` is invoked before removal of a package and `postrm` afterwards.

#### Installation and Upgrade

Here is what happens during an installation (or an update):

1. For an update, `dpkg` calls the `old-prerm upgrade new-version` then `new-preinst upgrade`.
2. For a first installation, it executes `new-preinst install`. It may add the old version in the last parameter, if the package has already been installed and removed since (but not purged, the configuration files having been retained).
3. The new package files are then unpacked. If a file already exists, it is replaced.
4. For an update, `dpkg` executes `old-postrm upgrade new-version`.
5. Finally, dpkg configures the package by executing `new-postinst configure last-version-configured`.

#### Package Removal

Here is what happens during a package removal:

1. `dpkg` calls `prerm remove`.
2. dpkg removes all of the package’s files, with the exception of the configuration files and configuration scripts.
3. `dpkg` executes `postrm remove`. All of the configuration scripts, except `postrm`, are removed. If the user has not used the “purge” option, the process stops here.
4. For a complete purge of the package (command issued with `dpkg --purge` or `dpkg -P`), the configuration files are also deleted, as well as a certain number of copies (`*.dpkg-tmp`,`*.dpkg-old`, `*.dpkg-new`) and temporary files; `dpkg` then executes `postrm purge`.
