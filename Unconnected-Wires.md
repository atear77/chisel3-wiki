The Invalidate API [#645](https://github.com/freechipsproject/chisel3/pull/645) adds support to chisel
to generate errors for unconnected wires.

Prior to this pull request, chisel automatically generated a firrtl `is invalid` for `Module IO()`, and each `Wire()` definition.
This made it difficult to detect cases where output signals were never driven.
Chisel now supports a `DontCare` element, which may be connected to an output signal, indicating that that signal is intentionally not driven.
Unless a signal is driven by hardware or connected to a `DontCare`, Firrtl will complain with a "not fully initialized" error.

###API

Output signals may be connected to DontCare, generating a `is invalid` when the corresponding firrtl is emitted.
```scala
io.out.debug := true.B
io.out.debugOption := DontCare
```
This indicates that the signal `io.out.debugOption` is intentionally not driven and firrtl should not issue a "not fully initialized"
error for this signal.

NOTE: This says nothing about (and cannot be connected to input signals).

This feature is controlled by `CompileOptions.explicitInvalidate` and is set to `false` in `NotStrict` (Chisel2 compatibility mode),
and `true` in `Strict` mode.

You can selectively enable this for Chisel2 compatibility mode by providing your own implicit `compileOptions`,
either for a group of Modules (via inheritance):
```scala
abstract class ExplicitInvalidateModule extends Module()(chisel3.core.ExplicitCompileOptions.NotStrict.copy(explicitInvalidate = true))
```
or on a per-Module basis:
```scala
class MyModule extends Module
  override val compileOptions = chisel3.core.ExplicitCompileOptions.NotStrict.copy(explicitInvalidate = true)
  ...
}
```

Or conversely, disable this stricter checking which is now the default in pure chisel3:
```scala
abstract class ImplicitInvalidateModule extends Module()(chisel3.core.ExplicitCompileOptions.Strict.copy(explicitInvalidate = false))
```
or on a per-Module basis:
```scala
class MyModule extends Module
  override val compileOptions = chisel3.core.ExplicitCompileOptions.Strict.copy(explicitInvalidate = false)
  ...
}
```

Please see the corresponding [API tests](https://github.com/freechipsproject/chisel3/blob/master/src/test/scala/chiselTests/InvalidateAPISpec.scala)
for examples.
