Work in progress.

Version 2 will extend Version 1, ideally in a compatible fashion.  Here's what's going on.

### Tables-of-anyref

(Note, this is just a first step; it is further generalized in the next section when `anyfunc` becomes a value type.)

A table can now be `anyref` in addition to `anyfunc`.  The code for `anyref` is 0x6F, its standard type code.

Denote table-of-anyref as `T(anyref)` and table-of-anyfunc as `T(anyfunc)`.

`anyref` can be used in the wasm text format for the type of the table.

`"anyref"` can be used as the type name passed to the JS `WebAssembly.Table` constructor.

Setting elements in a `T(anyref)` from JS will store JS objects.  If the value being set is not already an object we perform the same coercion as we do on a function boundary when JS passes a non-object to a wasm function that takes an anyref.  (Currently this is a ToObject operation but this is subject to change.)

Element segments are *not* further extended, they can reference only function values.

We can `table.copy` only between tables of the same type.

`table.init` can only init `T(anyfunc)` since the source in this case is an elem segment which can only reference function values.

`call_indirect` requires `T(anyfunc)`.

`table.get` can target only `T(anyref)`, the result is `anyref`.

`table.set` can target only `T(anyref)` and the static type of the value must be some `ref` type.

### anyfunc as value type, and fallout from that

(Work in progress)

We introduce a new value type `anyfunc`, where `anyfunc` <: `anyref`.

We can `table.copy` from table T1 to table T2 if the element type of T1 is a subtype of the element type of T2.  Specifically, we can `table.copy` from `T(anyfunc)` to `T(anyref)`.  TODO: under what conditions?  Exported functions?

We can `table.init` a `T(anyref)` since anyfunc <: anyref.  TODO: under what conditions?  Exported functions?

(Eventually)  `T(anyref)` can be targeted by element segments holding function values,  and the values stored in such tables are the function values that would be obtained from the host side if the host side reached into a corresponding anyfunc table and extracted functions.

When `table.get` targets a `T(anyfunc)` what do we return?  probably a function object.  But under what restrictions?  Exported function?

this is fairly obvious; value must be anyref.  on `T(anyfunc)` the static type must be anyfunc.

TODO: no downcast function at this point, so if we have an `anyref` we can't cast it dynamically to `anyfunc` or test whether we could do that safely.  


### Tentative cleanup

Change the current `(ref.null T)` construction into `ref.null` and introduce the nullref type.  For backward compatibility, continue to accept the old encoding?  Tricky, because then we'll need a new opcode.

Rename or create alias for `ref.is_null` as `ref.isnull` (as spec requires).  Superficial text format change.