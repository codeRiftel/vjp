# Verbose JSON Parser
VJP is a verbose JSON parser and generator written in C# 2.0 which heavily relies on type system.

## Why?
Unity has very bad support for JSON and I have to use Unity, so I decided to make my life a little bit less miserable by creating my own parser.

## Features
* uses C# 2.0
* codebase is less than 800 SLOC
* no OOP
* no exceptions
* heavy usage of type system

## Usage
First of all, drop **vjp** directory into your project. If it's a Unity project, then you probably want to put it in *Assets/Scripts*.

As stated earlier, this parser makes heavy usage of type system. In particular, it is based on *vjp/Option.cs* structure, so I recommend you to read it first. After that, you should read *vjp/Result.cs* and **JSONType** which is located in *vjp/VJP.cs*.

Suppose you have this JSON in a **string** named **input**
```javascript
{
    "foo": [
        "bar",
        -42,
        true,
        null
    ]
}
```
Let's parse it.
```csharp
// second parameter represents depth limit
Result<JSONType, JSONError> typeRes = VJP.Parse(input, 1024);
if (typeRes.IsErr()) {
    JSONError error = typeRes.AsErr();
    // do something about this error
} else {
    JSONType type = typeRes.AsOk();
    if (type.Obj.IsSome()) {
        Dictionary<string, JSONType> obj = type.Obj.Peel();
        if (obj.ContainsKey("foo")) {
            JSONType foo = obj["foo"];
            if (foo.Arr.IsSome()) {
                List<JSONType> arr = foo.Arr.Peel();
                for (int i = 0; i < arr.Count; i++) {
                    JSONType element = arr[i];
                    if (element.Str.IsSome()) {
                        // element.Str.Peel() == "bar"
                    } else if (arr[i].Num.IsSome()) {
                        // element.Num.Peel() == -42
                    } else if (arr[i].Bool.IsSome()) {
                        // element.Bool.Peel() == true
                    } else if (arr[i].Null.IsSome()) {
                        // element.Null.Peel() == JSONNull
                    }
                }
            }
        }
    }
}
```
To generate JSON you need to fiil **JSONType**
```csharp
Dictionary<string, JSONType> root = new Dictionary<string, JSONType>();
root["foo"] = JSONType.Make("bar");
JSONType type = JSONType.Make(root);

string json = VJP.Generate(type); // {"foo":"bar"}
```
I hope you understand why it is called verbose now.
