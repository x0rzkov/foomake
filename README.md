# foomake

Wouldn't it be nice to have a more grokkable way to express CMake build requirements? So that, instead of&hellip;

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.2)

project(example VERSION 0.1.0 LANGUAGES CXX)

set(DEFAULT_BUILD_TYPE "Release")

add_library(grok STATIC src/laserpants/grok.cpp)

target_include_directories(grok PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include")
target_compile_features(grok PUBLIC cxx_std_17)

add_executable(main src/main.cpp)

target_link_libraries(main grok)
```

&hellip;we could write something like:

```yaml
name:                   example
version:                0.1.0
description:            An example of a declarative CMakeLists configuration
homepage:               http://github.com/laserpants/foomake#readme
languages:              CXX
variables:
  DEFAULT_BUILD_TYPE:   Release

executables:
  main:
    source-files:
      - src/main.cpp
    link-libraries:
      - grok

libraries:
  grok:
    type: static
    source-files:
      - src/laserpants/grok.cpp
    include-directories:
      public:
        - '${CMAKE_CURRENT_SOURCE_DIR}/include'
    compile-features:
      public:
        - cxx_std_17
```

## Top-level keys

| Key                     | Type                     | Description                                     |
|-------------------------|--------------------------|-------------------------------------------------|
| name<sup>&dagger;</sup> | string                   | Set the `PROJECT_NAME` variable                 |
| version                 | string                   | Set the `PROJECT_VERSION` variable              |
| description             | string                   | Set the `CMAKE_PROJECT_DESCRIPTION` variable    |
| homepage                | string                   | Set the `CMAKE_PROJECT_HOMEPAGE_URL` variable   |
| languages               | list (or string)         | Set the project `LANGUAGES`                     |
| cmake-minimum-required  | dict (or string)         | Set the minimum required version of CMake       |
| executables             | dict                     | Executable targets (binaries)                   |
| libraries               | dict                     | Library targets                                 |
| variables               | dict                     |                                                 |
| options                 | dict                     |                                                 |
| install                 | dict                     |                                                 |
| configure               | list                     | Copy and perform variable substitution on files |
| dependencies            | list                     |                                                 |

&dagger;) Required if any of `version`, `description`, `homepage`, or `languages` are set.

---

### `name`, `version`, `description`, `homepage`

These properties set the project details. For example,

```yaml
name: Your project
version: '1.3'
description: One project to rule them all
```

translates to:

```cmake
project(Your project
  VERSION
    1.3
  DESCRIPTION
    One project to rule them all
  )
```

Surround the `version` string in single or double quoutes to avoid the value being interpreted as a number.

---

### `languages`

A list of languages that your project supports. For example,

```yaml
name: example
languages:
  - CXX
  - Fortran
```

translates to:

```cmake
project(example LANGUAGES CXX Fortran)
```

A string can also be used: 

```yaml
languages: CXX
```

---

### `cmake-minimum-required`

```yaml
cmake-minimum-required:
  version: '3.2'
```

The following form is also accepted:

```yaml
cmake-minimum-required: '3.2'
```

Or to specify a range:

```yaml
cmake-minimum-required: '3.1...3.13'
```

---

### `executables`

```yaml
executables:
  <target-name>: <dictionary>
  <target-name>: <dictionary>
  ...
```

See [Targets](#Targets)

---

### `libraries`

```yaml
libraries:
  <target-name>: <dictionary>
  <target-name>: <dictionary>
  ...
```

See [Targets](#Targets)

---

### `variables`

A dictionary where each key corresponds to the name of a variable.

```yaml
variables:
  <variable-name>: <value>
  <variable-name>: <value>
  ...
```

The values are either strings (the variable's value), or objects of the following form:

| Key          | Type                               | Required | Description                                   |
|--------------|------------------------------------|----------|-----------------------------------------------|
| value        | string                             | yes      |                                               |
| TODO         |                                    |          |                                               |

```yaml
variables:
  MY_VARIABLE: rocks
```

```cmake
set(MY_VARIABLE "rocks")
```

```yaml
variables:
  MY_VARIABLE:
    value: rocks
```

---

### `options`

```yaml
options:
  <option-name>: <dictionary>
  <option-name>: <dictionary>
  ...
```

| Key            | Type                               | Required | Description                                   |
|----------------|------------------------------------|----------|-----------------------------------------------|
| description    | string                             |          |                                               |
| initial-value  | string                             |          |                                               |

```yaml
options:
  DISCO_PANTS:
    description: Whether to clothe oneself in disco attire or not
    initial-value: 'YES'
```

---

### `install`

---

### `configure`

A list of files to perform variable substitution on. List entries should be dictionaries of the following form:

| Key                  | Type                     | Required | Description                                        |
|----------------------|--------------------------|:--------:|----------------------------------------------------|
| file                 | list                     | yes      | A two-element list of the form [ input, output ]   |
| arguments            | dict                     |          | Arguments accepted by the `configure_file` command |

Example:

```yaml
configure:
  - file: ['config.h.in', 'config.h']
    arguments:
      @ONLY: true
```

### `file`

### `arguments`

---

## Targets

Targets are specified under the `executables`, and `libraries` top-level keys respectively. 

```yaml
  executables:
    <target-name>: <dictionary>
    <target-name>: <dictionary>
    ...

  libraries:
    <target-name>: <dictionary>
    <target-name>: <dictionary>
    ...
```

The following keys appear in mappings of both types.

| Key                  | Type                               | Required | Default | Alias        | Description                                   |
|----------------------|------------------------------------|:--------:|---------|--------------|-----------------------------------------------|
| source-files         | list                               |          |         |              |                                               |
| include-directories  | dict or list                       |          | []      | include-dirs |                                               |
| link-libraries       | list                               |          |         | link-libs    |                                               |
| compile-features     | dict                               |          |         |              |                                               |

### `include-directories`

```yaml
    include-directories:
      public:
        - 'include'
        - 'include/stuff'
```

```yaml
    include-directories:
      - 'include'
      - 'include/stuff'
```

```yaml
    include-directories:
      public:
        - path: 'include'
      private:
        - path: 'include/stuff'
```

---

### `link-libraries`

---

### `source-files`

---

### Executables

| Key                  | Type                     | Required | Default | Alias        | Description                                    |
|----------------------|--------------------------|:--------:|---------|--------------|------------------------------------------------|

---

### Libraries

| Key                  | Type                               | Required | Default | Alias        | Description                                   |
|----------------------|------------------------------------|:--------:|---------|--------------|-----------------------------------------------|
| type                 | static &vert; shared &vert; module |          | static  |              |                                               |
---

### `type`

---

## Dependencies

```yaml
name: example
find-package: 
  - name: Boost
    version: '1.55'
    components: 'asio'

executables:
  main:
    link-libraries:
      - Boost::boost
```

```yaml
name: example
find-package: 
  - Boost
```


## Examples

The following examples are adapted from https://cmake.org/cmake-tutorial/

### Step 1

```cmake
cmake_minimum_required (VERSION 2.6)
project (Tutorial)

# the version number
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the binary tree to the search path for include files so that we will find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")

# add the executable
add_executable(Tutorial tutorial.cxx)
```

```yaml
name: Tutorial
version: '1.0'                      # the version number

cmake-minimum-required:
  version: '2.6'

executables:
  Tutorial:                         # add the executable
    source-files:
      - tutorial.cxx
    include-directories:
      public:
        - '${PROJECT_BINARY_DIR}'   # add the binary tree to the search path for include files
                                    # so that we will find TutorialConfig.h

# configure a header file to pass some of the CMake settings to the source code
configure:
  - file: ['${PROJECT_SOURCE_DIR}/TutorialConfig.h.in',
           '${PROJECT_BINARY_DIR}/TutorialConfig.h']
```

### Step 2

```cmake
# CMakeLists.txt

cmake_minimum_required (VERSION 2.6)
project (Tutorial)

# the version number
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the binary tree to the search path for include files so that we will find TutorialConfig.h
include_directories ("${PROJECT_BINARY_DIR}")

# add the MathFunctions library?
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif ()

# add the executable
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial  ${EXTRA_LIBS})
```

```cmake
# MathFunctions/CMakeLists.txt

add_library(MathFunctions mysqrt.cxx)
```


```yaml
name: Tutorial
version: '1.0'

cmake-minimum-required:
  version: '2.6'

options:
  USE_MYMATH:
    description: Use tutorial provided math implementation
    initial-value: 'ON'

executables:
  Tutorial:
    source-files:
      - tutorial.cxx
    include-directories:
      public:
        - '${PROJECT_BINARY_DIR}'
    .if-USE_MYMATH-link-libraries:
      - MathFunctions

libraries:
  .if-USE_MYMATH-MathFunctions:
    source-files:
      - MathFunctions/mysqrt.cxx
    include-directories:
      public:
        - '${PROJECT_SOURCE_DIR}/MathFunctions'

configure:
  - file: ['${PROJECT_SOURCE_DIR}/TutorialConfig.h.in',
           '${PROJECT_BINARY_DIR}/TutorialConfig.h']
```
