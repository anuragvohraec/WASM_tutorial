# Terminologies in WebAssembly world
1. EMSCRIPTEN : Compiler and tool chain for using c/C++ code into web by compiling them to WASM.
2. WASI: Its API described to be used by the host environment to support Web assembly. WASM runtime supports this APIs.
3. WASIX: Extar APi on top of WASI
4. WASMER: An example WASM run time, which supports WASI/WASIX/EMSCRIPTEN

# Using EMSCRIPTEN as our tool chain and browser as our WASM engine
1. install EMSCRIPTEN using official doc. It will create a directory: emsdk
2. Open a prompt from this directory and run `source ./emsdk_env.sh`, this will set up emscripten in your current shell
3. Lets see this example CPP file (hello.cpp):
    ```cpp
    #include <emscripten.h>

    EMSCRIPTEN_KEEPALIVE
    int add(int x, int y) {
    return x + y;
    }

    EMSCRIPTEN_KEEPALIVE
    int subtract(int x, int y){
    return x-y;
    }
    ```
    Explanation of the code:
    * Consider this as a facade to your C++ library file, in here you will expose all your functions you wanted to call in your JS code.
    * `#include <emscripten.h>` is header file which needs to be included and `EMSCRIPTEN_KEEPALIVE` macro needs to be put upon every function which needs to be exported to JS Code.
4. Now change the directory to the location where hello.cpp is located
5. Run command `em++ -O3 --no-entry hello.cpp -o hello.wasm` to create a wasm file.
6. You will need [this VSCODE plugin](https://marketplace.visualstudio.com/items?itemName=dtsvet.vscode-wasm) to view WASM in human readable format
7. Right click on wasm and select `Show WebAssembly`, it will give that text readable format:
    ```
    �I �I         0X    unc (param i32 i32) (result i32)))
    (type (;1;) (func))
    (type (;2;) (func (result i32)))
    (type (;3;) (func (param i32)))
    (func (;0;) (type 1))
    (func (;1;) (type 0) (param i32 i32) (result i32)
        local.get 0
        local.get 1
        i32.add)
    (func (;2;) (type 0) (param i32 i32) (result i32)
        local.get 0
        local.get 1
        i32.sub)
    (func (;3;) (type 2) (result i32)
        global.get 0)
    (func (;4;) (type 3) (param i32)
        local.get 0
        global.set 0)
    (table (;0;) 2 2 funcref)
    (memory (;0;) 256 256)
    (global (;0;) (mut i32) (i32.const 66560))
    (export "memory" (memory 0))
    (export "_Z3addii" (func 1))
    (export "_Z8subtractii" (func 2))
    (export "__indirect_function_table" (table 0))
    (export "_initialize" (func 0))
    (export "stackSave" (func 3))
    (export "stackRestore" (func 4))
    (elem (;0;) (i32.const 1) func 0))
    ```
8. Our functions are `_Z3addii` and `_Z8subtractii`
9. In browser you can use this function as such:
```js
WebAssembly.instantiateStreaming(fetch("hello.wasm")).then(
      (mod) => {
        //export the C++ functions: The function name can be seen in WASM file using VSCODE plugin
        const add = mod.instance.exports._Z3addii;
        const subtract = mod.instance.exports._Z8subtractii;

        //Use C functions
        console.log("Add: ",add(4,2));
        console.log("Subtract: ",subtract(6,2));
      },
    );
```