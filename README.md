
# Bazel to CMake

This project converts Bazel `BUILD` and `WORKSPACE` files to
`CMakeLists.txt`.  The Bazel BUILD file serves as your source of
truth, and this tool generates an idiomatic CMakeLists.txt so users
can do a CMake-based build.

This is not an official Google product.

## Project Status: Experimental

This project is experimental, and is only being used by a few projects
currently.  It is not part of the official Bazel ecosystem or roadmap.

The tool is quite incomplete at the moment, but I am working on adding
more features so that it is effective for more and more projects.  The
goal is to support a wide range of Bazel rules and features, including
advanced features like aspects, but how to make this work well is an
area of open experimentation and research.

## Usage

From the root directory of a Bazel project (where your `BUILD` and
`WORKSPACE` files live) run the following command.

    $ path/to/bazel_to_cmake.py CMakeLists.txt

You can check the generated `CMakeLists.txt` file into your repository
if you wish.  If you take this approach you will probably want a test
that ensures it stays up-to-date with your Bazel `BUILD` file.  You
may even want to create a `cc_test()` in Bazel that runs the CMake
build to continuously verify that it works.

(TODO: create a Bazel rule that does all of the above conveniently
without the user having to fuss).

## Why convert Bazel to CMake?

Bazel `BUILD` files are a nice way to write build systems.  `BUILD`
files are high level, mostly declarative, and generally easy on the
eyes.  `WORKSPACE` files are likewise a nice declarative way of
describing your dependencies.  When you need imperative logic, you can
write supporting functions in separate `.bzl` files.  All of these
files are written in a subset of Python, a nice readable language
that many people know.

The Bazel tool itself has nice features too.  It builds your code in a
sandbox, which ensures that each build step has properly declared all
of its dependencies.  It also enforces access control, ensuring that
your private rules and headers stay private.  Of major build systems,
Bazel gives you the most tools for imposing structure on your
build and enforcing this structure.  When I port build systems to
Bazel, I often catch things that weren't quite right before, but
happened to work in a more permissive build system.  A correct
build is easier to reproduce, parallelize, distribute across a
cluster, and is just generally easier to reason about.

These attributes make Bazel a great "source of truth" build system,
in my opinion.  During development you can build with the most
rigorous checking, and your build system can be written in a nice,
readable language.

On the other hand, Bazel is inconvenient for some uses:
 * when you are developing a C++ library, oftentimes the expectation
   from your clients is to be able to build it using one of the more
   established C++ build systems, one of which is CMake. Automatically
   providing a CMake build for the convenience of these users is
   preferable to manually maintaining support for another build system.
 * there are some platforms that Bazel does not support, but you still
   want to enable your users to build on and for those platforms without
   setting up cross-compilation.  Automatically generated CMake build
   can be a great escape hatch for those users.

This tool is designed to let you write your build system in Bazel,
then generate first-class CMakeLists.txt output so that CMake users
can conveniently build your code in idiomatic CMake ways.

## How the tool works

This tool does not use Bazel implementation at all.  Instead it takes
advantage of the fact that Bazel input files are a subset of Python by
using Python to evaluate them.  This means that the translation is
fast and has no dependencies except Python.  However it also means
that every kind of Bazel rule needs specific support in this tool.

We aim to generate CMake output that is as idiomatic as possible.
The tool tries to generate the CMake that a CMake expert would
have written manually, based on best practices for CMake.

To translate idiomatically, we translate at a high level.  For
example `cc_library()` translates directly to `add_library()` in
CMake.  But we try to capture subtleties; for example a header-only
library needs to have the "INTERFACE" attribute in CMake.
