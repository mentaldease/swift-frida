# swift-frida

Swift runtime interop from Frida. (See [frida-swift](https://github.com/frida/frida-swift) instead, if you're looking to talk to Frida from Swift code you write.)

## Status

This is a work-in-progress, don't expect anything to work yet! 

I'm mainly testing things on iOS 11.1.2 (64bit), every now and then also on iOS 9.3.5 (32bit). I haven't looked at any other platform at all. I'm only testing with Apps using Swift 4.0 at the moment.


## Available APIs

Right now, the following functions are available in the `Swift` namespace, when the script is loaded, and should be mostly working:

 * `available` allows you to check if the current process uses Swift. Don't use any of the other functions here if this property is `false`.
 * `isSwiftName(name)` Takes a function/symbol name (like you can get from `Module` objects), and returns a boolean indicating whether it is a mangled Swift name or not.
 * `demangle(name)` Takes a mangled Swift name (like you can get from `Module` objects), and returns a demangled, human-readable String for it.
 * `enumerateTypesSync()` Returns an array with the information about all Swift data types defined in the Swift program. The returned objects have a `toString` method returning the fully qualified name of the type, including generic bounds (if possible) and a string property `kind` that tells you which kind of type (e.g. `"Class"`, `"Struct"`) it is. Depending on the kind of type, additional methods are available:
 
    Kind             | available method
    -----------------|---------------------------------------------------------------------------------------------------------------
    Enum             | `enumCases` returns an array with all defined cases, with their names, types, and attributes.
    String, Struct   | `fields` returns an array with all fields of this class or struct, with names, offsets, types, and attributes.
    Tuple            | `tupleElements` returns an array with the types and labels of the elements in the tuple.
    Function         | `returnType` returns the return type of the function.
    Function         | `functionFlags` returns an object telling you the calling convention and whether the function throws or not.
    Function         | `getArguments` returns an array with the type and flags (`inout`) for every function argument.
    ObjCClassWrapper | `getObjCObject` returns an `ObjC.Object` describing this class.
   
    Uninstantiated generic types (you can check this with the `isGeneric` method), do not have any of these methods available. You must call `withGenericParams` to get a fully concrete type.

But, again, this is completely unstable and might change at any time. There's also some other stuff there, but those are just API sketches of what might maybe come in future.

## Test Setup

For testing, you want to run a command like this, to recompile whenever you change the scripts:

    frida-compile -w -o /tmp/swift.js loader

Then, you can just run Frida interactively, with the Swift module loaded:

    frida -U -n Foo -l /tmp/swift.js

You can now play with the functions of the `Swift` object.  (After making a change to a file in this repo, use `%reload` to reload the Swift script after recompiling.)

