+++
title = "Documentation with Doxygen and CMake: a (slightly) improved approach"
description = "Generating Doxygen documentation with CMake with proper targets and dependency checking."
date = "2013-01-19"
categories = ["Dev", "Tools"]
tags = ["cmake", "doxygen", "build system"]
+++

After days of hard work we ended up with our lovely crafted code base, a
portable CMake build system and we are now facing the task to generate a proper
Doxygen documentation for our anxious customer. The solution, widely adopted,
seems to be the one and only that can be found while googling and, to me, looks
not so neat. So I tried to come up with a different approach.

<!--more-->

Let's first have a look at the widespread solution:

```cmake
#-- Adds an Option to toggle the generation of the API documentation
option(BUILD_DOCUMENTATION "Use Doxygen to create the HTML based API documentation" OFF)
IF(BUILD_DOCUMENTATION)
  FIND_PACKAGE(Doxygen)
  IF(NOT DOXYGEN_FOUND)
    message(FATAL_ERROR "Doxygen is needed to build the documentation")
  ENDIF()
  #-- Configure the Template Doxyfile for our specific project
  configure_file(Doxyfile.in ${PROJECT_BINARY_DIR}/Doxyfile @ONLY IMMEDIATE)
  #-- Add a custom target to run Doxygen when ever the project is built
  add_custom_target (Docs ALL
    COMMAND ${DOXYGEN_EXECUTABLE} ${PROJECT_BINARY_DIR}/Doxyfile
    SOURCES ${PROJECT_BINARY_DIR}/Doxyfile)
  # IF you do NOT want the documentation to be generated EVERY time you build the project
  # then leave out the 'ALL' keyword from the above command.
ENDIF()
```

Well, even if the above solution works like a charm, it exposes a flaw that
makes its adoption very annoying: **Doxygen will be triggered every time we run
make, even if nothing has changed**. Oh, *every time* means exactly *every
time*: each make invocation (make, make install, etc...) will flood your shell
with the utterly verbose output from Doxygen even if that step is obviously
useless. Yes, we have added that `ALL` flag to `ADD_CUSTOM_TARGET` but this is
the only way to have our documentation generated along with the default targets
(otherwise our `BUILD_DOCUMENTATION` flag would be ignored except for an
explicit `make doc`). This behavior is correct and the point is well highlighted
in the CMake reference for
[`ADD_CUSTOM_TARGET`](https://cmake.org/cmake/help/v2.8.10/cmake.html#command:add_custom_target):

> The target has no output file and is ALWAYS CONSIDERED OUT OF DATE even if the commands
> try to create a file with the name of the target. Use ADD_CUSTOM_COMMAND to generate a
> file with dependencies.

So, it seems that we are using the wrong tool. **How can we add a Doxygen target
with a real dependency checking?** The neat trick lies in the combined use of
two different commands:

1. `ADD_CUSTOM_COMMAND` will take care of dependency checking, firing up Doxygen
if and only if something (which the documentation depends on) has really changed
since last build;
2. `ADD_CUSTOM_TARGET` will allow us to have a convenient and
elegant make doc target, eventually added to default set. After a bit of
trial-and-error, I ended up with a bunch of lines of code that seems to work as
we wanted:

```cmake
option(BUILD_DOCUMENTATION
    "Create and install the HTML based API documentation (requires Doxygen)" OFF)
IF(BUILD_DOCUMENTATION)
  FIND_PACKAGE(Doxygen)
  IF(NOT DOXYGEN_FOUND)
    MESSAGE(FATAL_ERROR "Doxygen is needed to build the documentation.")
  ENDIF()

  SET( doxyfile_in          ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
  SET( doxyfile             ${PROJECT_BINARY_DIR}/Doxyfile)
  SET( doxy_html_index_file ${CMAKE_CURRENT_BINARY_DIR}/html/index.html)
  SET( doxy_output_root     ${CMAKE_CURRENT_BINARY_DIR}) # Pasted into Doxyfile.in
  SET( doxy_input           ${PROJECT_SOURCE_DIR}/src) # Pasted into Doxyfile.in
  SET( doxy_extra_files     ${CMAKE_CURRENT_SOURCE_DIR}/mainpage.dox) # Pasted into Doxyfile.in

  CONFIGURE_FILE( ${doxyfile_in} ${doxyfile} @ONLY )

  ADD_CUSTOM_COMMAND(
    OUTPUT ${doxy_html_index_file}
    COMMAND ${DOXYGEN_EXECUTABLE} ${doxyfile}
    # The following should be ${doxyfile} only but it
    # will break the dependency.
    # The optimal solution would be creating a
    # custom_command for ${doxyfile} generation
    # but I still have to figure out how...
    MAIN_DEPENDENCY ${doxyfile} ${doxyfile_in}
    DEPENDS project_targets ${doxy_extra_files}
    COMMENT "Generating HTML documentation"
  )

  ADD_CUSTOM_TARGET( doc ALL DEPENDS ${doxy_html_index_file} )

  INSTALL( DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/html DESTINATION share/doc )
ENDIF()
```

The point is that we still have our `doc` custom target (added to default
targets) that is going to be fired every time but the actual build step is
protected by the custom command that will check for the right dependencies. Just
a couple of remarks:

* the `DEPENDS` option on the custom command must list all of the targets, files
and whatever your documentation depends on;
* the `${doxyfile_in}` dependency is an overshoot but, as I say in the comment,
removing it will break the dependency checking. The ideal solution would be
specifying `${doxyfile}` only, letting it to trigger a lower-level custom
command intended to configure and generate `${doxyfile_in}`. I'm still trying to
find an elegant solution to this little aesthetic flaw.

----------

Resources:

* [CMake Reference Manual](http://www.cmake.org/cmake/help/v2.8.10/cmake.html)
* [Use Doxygen with a CMake Based Project](http://www.bluequartz.net/projects/EIM_Segmentation/SoftwareDocumentation/html/usewithcmakeproject.html)
