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

If you are intersted in the sources you will also need `stew.parser`:
```
.> pull git@github.com:stew/codebase:.parser.trunk
```
together with this patch https://github.com/stew/codebase/issues/6

## Usage

`external.dhall.trunk.Dhall.evaluate : Text -> Either Text DhallValue` will evaluate a Dhall program into a DhallValue or an error message. 

### Example

Let's have a look at this Dhall program from the [Dhall homepage][dhall-lang]

```dhall
let Config : Type =
      { home : Text
      , privateKey : Text
      , publicKey : Text
      }

let makeUser : Text -> Config = \(user : Text) ->
      let home       : Text   = "/home/${user}"
      let privateKey : Text   = "${home}/.ssh/id_ed25519"
      let publicKey  : Text   = "${privateKey}.pub"
      let config     : Config = { home, privateKey, publicKey }
      in  config

let configs : List Config =
      [ makeUser "bill"
      , makeUser "jane"
      ]

in  configs
```

Assume we want to use this Dhall configuration to obtain a list of UserConfig
```unison
--  type for the user configuration data
unique type UserConfig = UserConfig Text Text Text
```

This mapping function converts the evaluated DhallValue into UserConfig by pattern matching on the structure.
Since the Dhall typechecker verifies that the result has the correct structure, we can use non-exhaustive pattern matching:

```unison
convertConfigs : DhallValue -> List UserConfig
convertConfigs = cases
  DhallList list ->
    List.map (cases DhallRecord map ->
      getString key = Map.get key map |> cases (Optional.Some (DhallText text)) -> text
      UserConfig (getString "home") (getString "publicKey") (getString "privateKey")) list

-- evaluate and convert input in a watch expression
> evaluate input |> Either.mapRight convertConfigs
```
This will give the following output
```ucm
> evaluate input |> Either.mapRight convertConfigs
⧩
Right
  [ UserConfig
    "/home/bill" "/home/bill/.ssh/id_ed25519.pub" "/home/bill/.ssh/id_ed25519",
    UserConfig
    "/home/jane" "/home/jane/.ssh/id_ed25519.pub" "/home/jane/.ssh/id_ed25519" ]
```

This example is also available in the Unison doc `external.dhall.trunk.README`.

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
658 total tests ( ✅ 604 passed, 🚫 54 failed )
```

### Limitations 

The following features are not yet supported
* type-checking
* importing of external files
* date & time types are not exposed in the resolved DhallValue, since there are no matching type in the Unsion standard library

### Future directions

Once Unison has some reflection/meta-programming possibilties it should be possible to
* generate a Dhall type for a Unison type (giving type checking & editor support for config files)
* automatically convert a Dhall value into a Unison value when their types are compatible

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
