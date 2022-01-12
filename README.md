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
pull https://github.com/hagl/dhall-unison:.dhall.releases._v2 .external.dhall.v2
```

If you are interested in the sources you will also need `stew.parser` and `stew.http`:
```
.> pull git@github.com:stew/codebase:.parser.trunk .external.stew.parser
.> pull git@github.com:stew/codebase:.http.trunk .external.stew.http
```

## Usage

The main interface for this library are the functions
```
evaluate : Text ->{IO} Either Text DhallValue
evaluateSimple : Text -> Either Text DhallValue
```


Both evaluate a text containing a dhall expression into an error or a value of type `DhallValue`.

```
unique type DhallValue
  = DhallInteger Integer
  | DhallNatural Natural
  | DhallFloat Float
  | DhallList [DhallValue]
  | DhallBoolean Boolean
  | DhallText Text
  | DhallRecord (Map Text DhallValue)
  | DhallOptional (Optional DhallValue)
```

`evaluate` supports all of Dhall's features. In order to resolve imports (i.e. environment variables, local or remote files) it needs the `IO` ability.

`evaluateSimple` is a pure function and can be used to evaluate a self-contained dhall expression (i.e. without any imports). It will fail and return a `Left` value when encountering an import during evaluation.


### Example 1: Converting a Dhall expression to a Unison value

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

Assume we want to use the above Dhall program to generate a list of Unison UserConfig values
```unison
unique type UserConfig = UserConfig Text Text Text
```

This mapping function converts the evaluated DhallValue into UserConfig by pattern matching on the structure.
Since the Dhall type-checker verifies that the result has the correct structure, we can use non-exhaustive pattern matching:

```unison
convertConfigs : DhallValue -> List UserConfig
convertConfigs = cases
  DhallList list ->
    List.map (cases DhallRecord map ->
      getString key = Map.get key map |> cases (Optional.Some (DhallText text)) -> text
      UserConfig (getString "home") (getString "publicKey") (getString "privateKey")) list

-- evaluate and convert input in a watch expression
> evaluateSimple input |> Either.mapRight convertConfigs
```
The watch expression will give the following output in `ucm`
```ucm
> evaluateSimple input |> Either.mapRight convertConfigs
⧩
Right
  [ UserConfig
    "/home/bill" "/home/bill/.ssh/id_ed25519.pub" "/home/bill/.ssh/id_ed25519",
    UserConfig
    "/home/jane" "/home/jane/.ssh/id_ed25519.pub" "/home/jane/.ssh/id_ed25519" ]
```

This example is also available online on [Unison Share][unison-share-hagl-dhall]

### Example 2: Evaluating Dhall code in ucm

This library also ships an unsupported runnable `dhallRun` function that can be used to quickly test a dhall expression directly from `ucm`.

The implementation uses a bit of a hack to not require quoting or escaping dhall code inside ucm.
Here are a few examples what you can do:

Simple calculations
```
.> run dhallRun 47 * 71
3337
```

Text interpolation, access to environment variables and calling built-in functions
```
.> run dhallRun "Hello ${env:USER as Text}!\n 3 + 4 = ${Natural/show (3 + 4)}"
"Hello harald!
 3 + 4 = 7"
```

Defining and applying functions
```
.> run dhallRun let add = \(x: Natural) -> \(y: Natural) -> x + y in add 1 2
3
```

Using remote dhall expressions (e.g. fold from Dhall prelude)
```
.> run dhallRun let sum = \(l: List Natural) -> https://prelude.dhall-lang.org/List/fold.dhall Natural l Natural (\(x: Natural) -> \(y: Natural) -> x + y)  0 in sum [1,2,3,4,5]
15
```

Loading and evaluating the remote expression takes some time. By adding a semantic hash to the import, the function can be cached locally. This will reduce the runtime of the second call below. You can use `dhallHash` to calculate the hash value of an import expression:

```
.> run dhallRun https://prelude.dhall-lang.org/List/fold.dhall
sha256:10bb945c25ab3943bd9df5a32e633cbfae112b7d3af38591784687e436a8d814

.> run dhallRun let sum = \(l: List Natural) -> https://prelude.dhall-lang.org/List/fold.dhall sha256:10bb945c25ab3943bd9df5a32e633cbfae112b7d3af38591784687e436a8d814 Natural l Natural (\(x: Natural) -> \(y: Natural) -> x + y)  0 in sum [1,2,3,4,5]
15

.> run dhallRun let sum = \(l: List Natural) -> https://prelude.dhall-lang.org/List/fold.dhall sha256:10bb945c25ab3943bd9df5a32e633cbfae112b7d3af38591784687e436a8d814 Natural l Natural (\(x: Natural) -> \(y: Natural) -> x + y)  0 in sum [1,2,3,4,5]
15
```

See the [Dhall documentation](dhall-lang) for more details about the language.


## Status

This project is currently in development, release v3 has alpha status.

The [Dhall Acceptance Tests][dhall-tests] can be run with the the from `ucm` with the command

```ucm
run .external.dhall.trunk.testsuite.runTestSuite <path to local copy of github.com/dhall-lang/dhall-lang>
```
At the time of this writing this will give the following results
```
1468 total tests ( ✅ 1461 passed, 🚫 7 failed) in directory /Users/harald/projects/dhall-unison/dhall-lang
Duration: 869.17s
```
The remaining failing tests are

* `dhall-lang/tests/type-inference/success/preludeA.dhall` : This test recursively loads all of prelude and shows some performance problems in the current implementation. It would probably succeed, but would take hours/days to do so. See the issues labeled [performance](https://github.com/hagl/dhall-unison/issues?q=is%3Aissue+label%3Aperformance+) for some ideas how to improve the performance.
* `./dhall-lang/tests/import/success/unit/cors/*` : The test.dhall-lang.org server that is used for CORS tests is currently not sending the correct response headers. See https://github.com/dhall-lang/dhall-lang/pull/1265 for details
* `./dhall-lang/tests/import/success/unit/headerForwarding` : This test requires https://github.com/dhall-lang/dhall-lang/pull/1263 to be merged.

### Limitations

The following features are not yet supported
* date & time types are not exposed in the resolved DhallValue, since there are no matching type in the Unsion standard library

### Future directions

Once Unison has some reflection/meta-programming possibilities it should be possible to
* generate a Dhall type for a Unison type (giving type checking & editor support for config files)
* automatically convert a Dhall value into a Unison value when their types are compatible

## Copyright and license

All code in this repository is available under the [3-Clause BSD License][license].

Copyright 2021 [Harald Gliebe][hagl]

This project is based on and uses code from the projects
- [dhall-lang][dhall-lang-project] ([3-Clause BSD License][dhall-lang-license])
- [dhall-haskell][dhall-haskell] ([3-Clause BSD License][dhall-lang-license])
- [stew/parser][stew-parser]
- [stew/http][stew-http]

[license]: https://github.com/hagl/dhall-unison/blob/main/LICENSE
[unison]: https://www.unisonweb.org/
[dhall-lang]: https://dhall-lang.org/
[dhall-lang-project]: https://github.com/dhall-lang/dhall-lang
[dhall-lang-license]: https://github.com/dhall-lang/dhall-lang/blob/main/LICENSE
[dhall-haskell]: https://github.com/dhall-lang/dhall-haskell
[dhall-haskell-license]: https://github.com/dhall-lang/dhall-haskell/blob/main/LICENSE
[dhall-tests]: https://github.com/dhall-lang/dhall-lang/tree/master/tests
[stew-parser]: https://share.unison-lang.org/latest/namespaces/stew/parser
[stew-http]: https://share.unison-lang.org/latest/namespaces/stew/http
[hagl]: https://twitter.com/hagl
[unison-share-hagl-dhall]: https://share.unison-lang.org/latest/terms/hagl/dhall/README
