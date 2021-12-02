# dhall-unison
A [Unison][unison] implementation of the [Dhall configuration language][dhall-lang].

## Table of contents

* [Installation](#installation)
* [Status](#status)
* [Usage](#usage)
* [Copyright and license](#copyright-and-license)

## Installation 

To install the latest version in your Unison codebase use the following ucm command:
```
pull https://github.com/hagl/dhall-unison:.dhall.trunk .external.dhall.trunk
```


## Usage

`external.dhall.trunk.Dhall.evaluate : Text -> Either Text DhallValue` will evaluate a Dhall program into a DhallValue or an error message. 

A complete code example is given in `external.dhall.trunk.README`

## Status 

This project is currently in development and doesn't have any releases yet.

The [Dhall Acceptance Tests][dhall-tests] of the sections 
* parser tests
* normalization tests
* alpha-normalization tests
can be run with the the from ucm with the command

```ucm
run .external.dhall.trunk.testsuite.runTestSuite <path to local copy of github.com/dhall-lang/dhall-lang>
```
At the time of this writing this will give the following results
```
658 total tests ( ✅ 565 passed, 🚫 93 failed )
```

### Limitations 

The following features are not yet supported
* type-checking
* importing of external files
* date & time types

## Copyright and license

All code in this repository is available under the [3-Clause BSD License][license].

Copyright 2021 [Harald Gliebe][hagl]

This project is based on and uses code from the projects
- [dhall-lang][dhall-lang-project] ([3-Clause BSD License][dhall-lang-license])
- [dhall-haskell][dhall-haskell] ([3-Clause BSD License][dhall-lang-license])
- [stew/parser][stew-parser]
  
[license]: https://github.com/hagl/dhall-unison/blob/main/LICENSE
[unison]: https://www.unisonweb.org/
[dhall-lang]: https://dhall-lang.org/
[dhall-lang-project]: https://github.com/dhall-lang/dhall-lang
[dhall-lang-license]: https://github.com/dhall-lang/dhall-lang/blob/main/LICENSE
[dhall-haskell]: https://github.com/dhall-lang/dhall-haskell
[dhall-haskell-license]: https://github.com/dhall-lang/dhall-haskell/blob/main/LICENSE
[dhall-tests]: https://github.com/dhall-lang/dhall-lang/tree/master/tests
[stew-parser]: [https://share.unison-lang.org/latest/namespaces/stew/parser]
[hagl]: [https://twitter.com/hagl]
