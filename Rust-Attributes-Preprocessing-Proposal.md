## Items
Items are top level things. According to the Rust reference, these can be:

* Modules: `mod`
* Import/Exports: `import`/`export`
* Functions: `fn`
* Predicates: `pred`
* Iterators: `iter`
* Object Items: `obj`
* Type Definitions: `type`
* Variant Records: `tag`

There are some others that aren't in the reference PDF like `res`, I think. Oh well. Importantly, **attributes are not considered items.**

## Attributes
Attributes are enclosed in `#[]` at the top level. They decorate the **item immediately following them**, skipping intervening attributes. The contents of `#[]` form a small, simple, command based language.

Symbols in this language may only be set to strings. Whether or not they are set can be used for boolean flagging.

The attribute language consists of:

* Attributes: #[_cmd_ _i1_ _i2_ _..._]
* Attribute Items:
   - Symbols: _sym_
      + Alphabetic only.
      + Can be considered _set_ or _unset_.
         * A _set_ symbol has a string value (defaults to the empty string).
      + May contain a string value. This automatically makes the symbol "set".
   - String Literals: _"contents"_
   - Boolean Literal: true, false
   - Pairs: _i1_=_i2_
   - Named Lists: _name_(_i1_, _i2_, ...)
   - Boolean Operators:
      + `and`, `or`, `not`
* Values
   - Booleans
   - Strings 

## Commands

The first symbol in an attribute is the `command` of that attribute. You cannot (at least yet) define your own commands. They are all built-ins. 

* `#[when _pred_ _attr_]`
   - Inserts the attribute in its place if the predicate is true. Must be a boolean value.
   - **IMPORTANT: This is the only way to insert new static attributes.** 
* `#[omit]`
   - Omits the attributed item from the generated code.
* `#[set _sym_ _val_]`
   - Sets the symbol to the value (boolean or string)
* `#[unset _sym_]`
   - Unsets a symbol.

During each preprocessing phase, all of the top level attributes which are known to be commands are executed. **Since attributes inserted by `when` are always placed immediately after the `when`, they will be evaluated by preprocessor immediately following it.** This prevents the need for multiple preprocessing passes.

## Predicates

Some reserved named lists are defined as simplistic predicates. At the moment you cannot define your own. When "evaluated" they are textually replaced with a true/false value. Existing ones will include:

 * Config Predicate: `cfg(_foo_=_bar_, _baz_=_qux_)`
 * Flags Predicate: `flags("-foo", "--bar"
 * Set/Unset Predicate: `is_set(_sym_)`
 * Version String Predicates: `v_gt(_i1_, _i2_)`, `v_eq(_i1_, _i2_)`, `v_lt(_i1_, _i2_)`
    * These compare versions of the form "x.y.z", and understand alphanumeric ordering.

## Functions

There are also a very limited set of functions for obtaining string values.

  * Flags: `get_flag(_"name"_)`
  * Config: `get_cfg(_"name"_)`