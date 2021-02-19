---
description: >-
  A Wasm module can export entities, like functions, memories, globals and
  tables. This example illustrates how to use exported globals.
---

# ✏️ Using guest \(exported\) globals

In this example we'll be using a simple Wasm module which exports some globals.

Globals are probably the simplest entity we'll encounter in Wasm modules but there is still some interesting things to talk about. For example, globals come in two flavors:

* Immutable globals \(`const`\)
* Mutable globals \(`var`\)

We will cover both in this example.

First we are going to want to initialize a new project. To do this we can navigate to our project folder, or create one. In this example, we will create a new project. Lets create it and navigate to it:

{% tabs %}
{% tab title="Rust" %}
{% hint style="info" %}
The final **Rust** code for this example can be found on Github: [exports_global.rs](https://github.com/wasmerio/wasmer/blob/master/examples/exports_global.rs).

_Please take a look at the_ [_setup steps for Rust_](../rust/setup.md)_._
{% endhint %}

```bash
cargo new exports-global
cd exports-global
```

We have to modify `Cargo.toml` to add the Wasmer dependencies as shown below:

```yaml
[dependencies]
# The Wasmer API
wasmer = "1.0"
```
{% endcode %}
{% endtab %}

{% tab title="Go" %}
{% hint style="info" %}
The final **Go** code for this example can be found on Github: [exports_global.go](https://github.com/wasmerio/wasmer-go/blob/master/examples/example_exports_global_test.go).

_Please take a look at the_ [_setup steps for Go_](../go/setup.md)_._
{% endhint %}

```bash
mkdir wasmer-example-imports-exports
cd wasmer-example-imports-exports
go mod init github.com/$USER/wasmer-example-imports-exports
```
{% endtab %}

{% tab title="Python" %}
{% hint style="info" %}
The final **Python** code for this example can be found on Github: [exports_global.py](https://github.com/wasmerio/wasmer-python/blob/master/examples/exports_global.py).

_Please take a look at the_ [_setup steps for Python_](../python/setup.md)_._
{% endhint %}

```bash
mkdir wasmer-example-imports-exports
cd wasmer-example-imports-exports
pip install wasmer wasmer_compiler_cranelift
```
{% endtab %}

{% tab title="PHP" %}
{% hint style="info" %}
The final **PHP** code for this example can be found on Github: [exports-global.php](https://github.com/wasmerio/wasmer-python/blob/master/examples/exports-global.php).

_Please take a look at the_ [_setup steps for PHP_](../php/setup.md)_._
{% endhint %}

```bash
mkdir wasmer-example-exports-globals
cd wasmer-example-exports-globals
composer init --name=wasmer-example-exports-globals
composer require wasm/wasm
```
{% endtab %}

{% tab title="C/C++" %}
{% hint style="info" %}
The final **C** code for this example can be found on Github: [exports-global.c](https://github.com/wasmerio/wasmer/blob/master/lib/c-api/examples/exports-global.c).

_Please take a look at the_ [_setup steps for C/C++_](../c/setup.md)_._
{% endhint %}

```bash
mkdir wasmer-example-exports-global
cd wasmer-example-exports-global
vim Makefile
```

Let's create a simple `Makefile`:

```bash
CFLAGS = -g -I$(shell $(WASMER_DIR)/bin/wasmer config --includedir)
LDFLAGS = -Wl,-rpath,$(shell $(WASMER_DIR)/bin/wasmer config --libdir)
LDLIBS = $(shell $(WASMER_DIR)/bin/wasmer config --libs)

.SILENT: exports-global exports-global.o
exports-global: exports-global.o

.PHONY: clean
.SILENT: clean
clean:
    rm -f exports-global.o exports-global
```
{% endtab %}
{% endtabs %}

Now that we have everything set up, let's go ahead and try it out!

## Querying types information

The first interesting thing to do is to query their type information in order to know if they are mutable or not. Our module exports two globals, `one` and `some`. Which one is mutable and which one is not?

{% tabs %}
{% tab title="Rust" %}
```rust
l​​et one = instance.exports.get_global("one")?;
let some = instance.exports.get_global("some")?;

​let one_type = one.ty();
let some_type = some.ty();

​println!("one type: {:?} {:?}", one_type.mutability, one_type.ty);
println!("some type: {:?} {:?}", some_type.mutability, some_type.ty);
```
{% endtab %}

{% tab title="Go" %}
```go
one, err := instance.Exports.GetGlobal("one")

if err != nil {
    panic(fmt.Sprintln("Failed to retrieve the `one` global:", err))
}

some, err := instance.Exports.GetGlobal("some")

if err != nil {
        panic(fmt.Sprintln("Failed to retrieve the `some` global:", err))
}
​
oneType := one.Type()
someType := some.Type()
​
fmt.Printf(
    "`one` type: %s %s\n", 
    oneType.Mutability(), 
    oneType.ValueType().Kind().String()
)
fmt.Printf(
    "`some` type: %s %s\n", 
    someType.Mutability(), 
    someType.ValueType().Kind().String()
)
```
{% endtab %}

{% tab title="Python" %}
```python
one = instance.exports.one
some = instance.exports.some

one_type = one.type
assert one_type.type == Type.F32
assert one_type.mutable == False

some_type = some.type
assert some_type.type == Type.F32
assert some_type.mutable == True
```
{% endtab %}

{% tab title="Python" %}
```php
$exports = $instance->exports();
$one = (new Wasm\Module\Extern($exports[0]))->asGlobal();
$some = (new Wasm\Module\Extern($exports[1]))->asGlobal();

$oneType = $one->type();
assert($oneType->mutability() === GlobalType::MUTABILITY_CONST);

$someType = $some->type();
assert($oneType->mutability() === GlobalType::MUTABILITY_VAR);
```
{% endtab %}

{% tab title="C/C++" %}
```c
wasm_mutability_t one_mutability = wasm_globaltype_mutability(one_type);
const wasm_valtype_t* one_content = wasm_globaltype_content(one_type);
wasm_valkind_t one_kind = wasm_valtype_kind(one_content);

wasm_mutability_t some_mutability = wasm_globaltype_mutability(some_type);
const wasm_valtype_t* some_content = wasm_globaltype_content(some_type);
wasm_valkind_t some_kind = wasm_valtype_kind(some_content);

printf(
    "`one` type: %s %hhu\n", 
    one_mutability == WASM_CONST ? "const" : "", 
    one_kind
);

printf(
    "`some` type: %s %hhu\n", 
    some_mutability == WASM_CONST ? "const" : "", 
    some_kind
);
```
{% endtab %}
{% endtabs %}

## Getting globals values

The global API is straightforward: it provides a dedicated method to get the value of a given global. Look how easy it is:

{% tabs %}
{% tab title="Rust" %}
```rust
let some_value = some.get();

println!("`some` value: {:?}", some_value);
```
{% endtab %}

{% tab title="Go" %}
```go
someValue, err := some.Get()

if err != nil {
    panic(fmt.Sprintln("Failed to get the `some` global value:", err))
}

fmt.Printf("`some` value: %.1f\n", someValue)
```
{% endtab %}

{% tab title="Python" %}
```python
some_value = some.value

assert some_value == 0.0
```
{% endtab %}

{% tab title="PHP" %}
```php
$someValue = $some->get()->value();

assert($someValue === 0);
```
{% endtab %}

{% tab title="C/C++" %}
```c
wasm_val_t some_value;
wasm_global_get(some, &some_value);

printf("`some` value: %.1f\n", some_value.of.f32);
```
{% endtab %}
{% endtabs %}

## Setting globals

As we said before, globals come in two flavor. Immutable globals, for which we can only set a value once and mutable ones.

First we'll try to set the value of a immutable global and see what happens:

{% tabs %}
{% tab title="Rust" %}
```rust
let result = one.set(Value::F32(42.0));

assert_eq!(
    result.expect_err("Expected an error").message(),
    "Attempted to set an immutable global"
);
```
{% endtab %}

{% tab title="Go" %}
```go
err = one.Set(float32(42.0), wasmer.F32)

if err == nil {
    panic(fmt.Sprintln("Setting value to `one` did not error"))
}
```
{% endtab %}

{% tab title="Python" %}
```go
try:
    one.value = 42.0
except RuntimeError as err:
    assert str(err) == 'The global variable is not mutable, cannot set a new value'
else:
    assert False
```
{% endtab %}

{% tab title="PHP" %}
```php
try {
    $one->set(42.0);   
} catch(\Wasm\Exception\RuntimeException $exception) {
    assert($exception->getMessage() === 'RuntimeError: Attempted to set an immutable global');
}
```
{% endtab %}

{% tab title="C/C++" %}
```c
wasm_val_t one_set_value = WASM_F32_VAL(42);
wasm_global_set(one, &one_set_value);

int error_length = wasmer_last_error_length();
if (error_length > 0) {
    char *error_message = malloc(error_length);
    wasmer_last_error_message(error_message, error_length);

    printf("Attempted to set an immutable global: `%s`\n", error_message);
}
```
{% endtab %}
{% endtabs %}

As you can see here, trying to set a value on a immutable global will always lead to an error.

Now let's see how to correctly set a value on a mutable global:

{% tabs %}
{% tab title="Rust" %}
```rust
some.set(Value::F32(21.0))?;
```
{% endtab %}

{% tab title="Go" %}
```go
err = some.Set(float32(42.0), wasmer.F32)

if err != nil {
    panic(fmt.Sprintln("Failed to set the `some` global value:", err))
}
```
{% endtab %}

{% tab title="Python" %}
```python
some.value = 21.0
```
{% endtab %}

{% tab title="PHP" %}
```php
$some->set(42.0);
```
{% endtab %}

{% tab title="C/C++" %}
```c
wasm_val_t some_set_value = WASM_F32_VAL(21);
wasm_global_set(some, &some_set_value);
```
{% endtab %}
{% endtabs %}

## Running

We now have everything we need to run the Wasm module, let's do it!

{% tabs %}
{% tab title="Rust" %}
You should be able to run it using the `cargo run` command. The output should look like this:

```text
Compiling module...
Instantiating module...
Getting globals types information...
`one` type: Const F32
`some` type: Var F32
Getting global values...
`one` value: 1.0
`some` value: F32(0.0)
Setting global values...
`one` value after `set`: F32(1.0)
`some` value after `set_some`: F32(21.0)
`some` value after `set`: F32(42.0)
```

{% hint style="info" %}
If you want to run the examples from the Wasmer [repository](https://github.com/wasmerio/wasmer/) codebase directly, you can also do:

```bash
git clone https://github.com/wasmerio/wasmer.git
cd wasmer
cargo run --example exported-function --release --features "cranelift"
```
{% endhint %}
{% endtab %}

{% tab title="Go" %}
You should be able to run it using the `go run main.go` command. The output should look like this:

```text
Compiling module...
Instantiating module...
Getting globals types information...
`one` type: const f32
`some` type: var f32
Getting global values...
`one` value: 1.0
`some` value: 0.0
Setting global values...
`one` value: 1.0
`some` value after `set_some`: 21.0
`some` value after `set`: 42.0
```

{% hint style="info" %}
If you want to run the examples from the Wasmer [repository](https://github.com/wasmerio/wasmer/) codebase directly, you can also do:

```bash
git clone https://github.com/wasmerio/wasmer-go.git
cd wasmer-go
go test examples/example_exports_global_test.go
```
{% endhint %}
{% endtab %}

{% tab title="Python" %}
You should be able to run it using the `python exports_global.py` command.

{% hint style="info" %}
If you want to run the examples from the Wasmer [repository](https://github.com/wasmerio/wasmer-python/) codebase directly, you can also do:

```bash
git clone https://github.com/wasmerio/wasmer-python.git
cd wasmer-python
python examples/exports_global.py
```
{% endhint %}
{% endtab %}

{% tab title="PHP" %}
You should be able to run it using the `php exports-global.php` command.

{% hint style="info" %}
If you want to run the examples from the Wasmer PHP [repository](https://github.com/wasmerio/wasmer-php/) codebase directly, you can also do:

```bash
git clone https://github.com/wasmerio/wasmer-php.git
cd wasmer-php
make EXAMPLE=exports-global test-doc-examples
```
{% endhint %}
{% endtab %}

{% tab title="C/C++" %}
You should be able to run it using the `make clean exports-global && ./exports-global` command. The output should look like this:

```text
Creating the store...
Compiling module...
Creating imports...
Instantiating module...
Retrieving exports...
Getting globals types information...
`one` type: const 2
`some` type:  2
Getting global values...`one` value: 1.0
`some` value: 0.0
Setting global values...
Attempted to set an immutable global: `RuntimeError: Attempted to set an immutable global`
`some` value: 0.0
```

{% hint style="info" %}
If you want to run the examples from the Wasmer [repository](https://github.com/wasmerio/wasmer/) codebase directly, you can also do:

```bash
git clone https://github.com/wasmerio/wasmer.git
cd wasmer/lib/c-api/examples/exports-global.c
make clean exports-global
./exports-global
```
{% endhint %}
{% endtab %}
{% endtabs %}

