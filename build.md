``` ucm
.> pull unison.public.base.releases.M4d .base
.> fork .base .dhall.main.lib.base
.> cd .dhall.main
.> load _src/license.u
.> add
.> load _src/support_types.u
.> add
.> load _src/support.u
.> add
.> load _src/syntax_types.u
.> add
.> load _src/syntax.u
.> add
.> load _src/binary.u
.> add
.> load _src/utils.u
.> add
.> pull stew.public.projects.uniparsec.releases.v2 .lib.uniparsec
.> fork .lib.uniparsec .dhall.main.lib.uniparsec
.> load _src/parser_types.u
.> add
.> load _src/parser.u
.> add
.> load _src/shift.u
.> add
.> load _src/substitution.u
.> add
.> load _src/alpha_normalization.u
.> add
.> load _src/beta_normalization.u
.> add
.> load _src/type_inference_types.u
.> add
.> load _src/type_inference.u
.> add
.> pull stew.public.projects.httpclient.releases.v10 .lib.httpclient
.> fork .lib.httpclient .dhall.main.lib.httpclient
.> load _src/import.u
.> add
.> load _src/testsuite.u
.> add
.> run runTestSuite ./dhall-lang
.> load _src/dhall_types.u
.> add
.> load _src/dhall.u
.> add
```
