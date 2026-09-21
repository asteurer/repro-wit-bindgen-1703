# Analysis of [wit-bindgen # 1703](https://github.com/bytecodealliance/wit-bindgen/issues/1703)

## Scenario 1: Reserved function name collisions with bindings generated from user-defined WIT types.

The issue is that `variants` and `resources` have reserved function names that have the potential to collide with user-defined WIT types.

To avoid unecessary breaking changes, I propose that we fix this by suffixing each colliding user-defined function with an underscore (`_`).
I think it would also be smart to add a doc comment to the colliding user-defined function and print a message to the console to notify the
user that their function has been mangled.

These are the current reserved functions that have a risk of collision:
- `variant.Tag()`
- `resource.TakeHandle()`
- `resource.SetHandle`
- `resource.Handle()`
- `resource.Drop()`
- `resource.OnDrop`

### Steps to recreate
1. Install [componentize-go](https://github.com/bytecodealliance/componentize-go)
2. Navigate to the repo root and run `componentize-go --world scenario-one bindings --generate-stubs --pkg-name bindings -o bindings`

## Scenario 2: Collisions between bindings generated from enum-like types and other user-defined types

The issue is specific to the enum-like WIT types (`enum`, `variant`, and `flags`) and any user-defined types that can be defined in a world or interface (excluding WIT `function`s). The WIT parser doesn't catch this because the generated bindings for these types create Go `const`s that are a concatenation of the name of the type and the type's variants, which then have the potential to collide with another user-defined type.

I propose we fix this by formatting the `const`s generated for the enum-like types as `format!("{TypeName}_{Case}")`. This will be a breaking change; however, I think this bug has a large enough surface area to justify the change.

### Steps to recreate
1. Install [componentize-go](https://github.com/bytecodealliance/componentize-go)
2. Navigate to the repo root and run `componentize-go --world scenario-two bindings --pkg-name bindings -o bindings`