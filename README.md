# TOMLKit

**A small, simple TOML parser for Swift. Powered by [toml++](https://github.com/marzer/tomlplusplus/).**

[![Swift 5.4](https://img.shields.io/badge/Swift-5.4-brightgreen?logo=swift)](https://swift.org)
[![SPM Compatible](https://img.shields.io/badge/SPM-compatible-brightgreen.svg)](https://swift.org/package-manager)
[![](https://img.shields.io/github/v/tag/LebJe/TOMLKit)](https://github.com/LebJe/TOMLKit/releases)
[![Build and Test](https://github.com/LebJe/TOMLKit/workflows/Build%20and%20Test/badge.svg)](https://github.com/LebJe/TOMLKit/actions?query=workflow%3A%22Build+and+Test%22)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2FLebJe%2FTOMLKit%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/LebJe/TOMLKit)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2FLebJe%2FTOMLKit%2Fbadge%3Ftype%3Dplatforms)](https://swiftpackageindex.com/LebJe/TOMLKit)

TOMLKit is a [Swift](https://swift.org) wrapper around [toml++](https://github.com/marzer/tomlplusplus/), allowing you to read and write [TOML](https://toml.io) files in Swift.

## Table of Contents

<!--ts-->

-   [TOMLKit](#tomlkit)
    -   [Table of Contents](#table-of-contents)
    -   [Installation](#installation)
        -   [Swift Package Manager](#swift-package-manager)
    -   [Usage](#usage)
        -   [Creating TOML Values](#creating-toml-values)
            -   [Tables](#tables)
                -   [From a Dictionary](#from-a-dictionary)
                -   [From a TOML String](#from-a-toml-string)
                -   [Conversion](#conversion)
            -   [Arrays](#arrays)
            -   [Dates](#dates)
            -   [Times](#times)
            -   [Date and Time](#date-and-time)
            -   [Data](#data)
            -   [Integers](#integers)
        -   [Retrieving TOML values](#retrieving-toml-values)
            -   [From Tables](#from-tables)
            -   [From Arrays](#from-arrays)
        -   [Encoding struct to TOML](#encoding-struct-to-toml)
        -   [Decoding TOML to struct](#decoding-toml-to-struct)
    -   [Dependencies](#dependencies)
    -   [Licenses](#licenses)
    -   [Contributing](#contributing)

<!-- Added by: lebje, at: Sun Sep 26 12:32:23 EDT 2021 -->

<!--te-->

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

Documentation is available [here](https://lebje.github.io/TOMLKit).

## Installation

### Swift Package Manager

Add this to the `dependencies` array in `Package.swift`:

```swift
.package(url: "https://github.com/LebJe/TOMLKit.git", from: "0.3.0")
```

Also add this to the `targets` array in the aforementioned file:

```swift
.product(name: "TOMLKit", package: "TOMLKit")
```

## Usage

Once you have installed TOMLKit, you will usually start with a `TOMLTable`, or by decoding a TOML document into a Swift structure.

### Creating TOML Values

#### Tables

TOML tables are key-value pairs, similar to a Swift `Dictionary`. They can also be embedded inside one another.

To create a empty `TOMLTable`, use the `TOMLTable.init(inline:)` initializer, where `inline` makes the table an inline table:

```toml
# `inline` is true.
inlineTable = { field1 = "", ... }
```

or a regular table:

```toml
# `inline` is false.
[regularTable]
field1 = ""
...
```

---

**NOTE**

Use `myTable.inline = true/false` to make a table inline/not inline,
and `myTable.inline` to check if a table is inlined.

---

To create a `TOMLTable` with values, use one of the below methods:

-   [From a Dictionary](#from-a-dictionary)
-   [From a TOML String](#from-a-toml-string)

##### From a Dictionary

```swift
let table = TOMLTable(["string": "Hello, World!", "int": 345025, "double": 025307.350])

// Or use `TOMLTable`'s `ExpressibleByDictionaryLiteral` conformance.
let table = ["string": "Hello, World!", "int": 345025, "double": 025307.350] as TOMLTable
```

##### From a TOML String

```swift
let toml = """
string = "Hello, World!"
int = 523053
double = 3250.34
"""

do {
   let table = try TOMLTable(string: toml)
} catch let error as TOMLParseError {
   // TOMLParseError occurs when the TOML document is invalid.

   /// `error.source.begin` contains the line and column where the error started,
   /// and `error.source.end` contains the line where the error ended.
    print(error.source.begin)
    print(error.source.end)
}
```

To insert values, make sure the value conforms to `TOMLValueConvertible`, then use the `subscript`, or the `insert(_ newElement:at:)` method:

```swift
let table = TOMLTable()

table["string"] = "Hello, World!"
table.insert(127, at: "int")
table["table"] = TOMLTable(["string 1": "Hello, Again!"], inline: true)
table["table"] = TOMLTable(["string 2": "Hello, Again!"], inline: false)

// Insert an integer using an octal representation.
table.insert(TOMLInt(0o755, options: .formatAsOctal), at: "octalInt")
```

##### Conversion

```swift
let table: TOMLTable = ...

// Convert to TOML using default settings.
let toml1 = table.convert()

// Convert to JSON using default settings.
let json = table.convert(to: .json)

// Convert to TOML with custom settings.
let toml2 = table.convert(to: .toml, options: [.quoteDateAndTimes, .allowMultilineStrings])
```

#### Arrays

TOML arrays are similar to a Swift `Array`. They can be embedded inside one another.

To create a empty `TOMLArray`, use the `TOMLArray.init` initializer.

To create a `TOMLArray` with values, use one of the below methods:

```swift
let array = TOMLArray(
	[
		"Hello, World!",
		"Hello, Again!",
		3294923,
		2350.53,
		TOMLTable(["string": "string 1"])
	]
)

// Or use `TOMLArray`'s `ExpressibleByArrayLiteral` conformance.
let array = [
	"Hello, World!",
	"Hello, Again!",
	3294923,
	2350.53,
	TOMLTable(["string": "string 1"])
] as TOMLArray
```

To insert values, use the `subscript`, the `append(_ value:)` method, or the `insert(_ value:at:)` method:

```swift
let array = TOMLArray()

array.append("Hello, World")
array.insert(TOMLInt(0x123abc, options: .formatAsHexadecimal), at: 1)
array[0] = TOMLTable(["double": 02734.23])
```

#### Dates

```swift
// Create a `TOMLDate` from numerical values.
let date = TOMLDate(year: 2021, month: 6, day: 10)

// Create a `TOMLDate` from `Foundation.Date`
import Foundation

let date2 = TOMLDate(date: Date())
let table = TOMLTable(["Date1": date, "Date2": date2])
```

#### Times

```swift
// Create a `TOMLTime` from numerical values.
let time = TOMLTime(hour: 12, minute: 27, second: 49)
let table = TOMLTable(["time": time])
```

#### Date and Time

```swift
// Create a `TOMLDateTime` from numerical values.
let dateTime = TOMLDateTime(
	date: TOMLDate(year: 2021, month: 5, day: 20),
	time: TOMLTime(hour: 4, minute: 27, second: 5, nanoSecond: 294),
	offset: TOMLTimeOffset(offset: 0)
)

// Create a `TOMLDateTime` from `Foundation.Date`
import Foundation
let dateTime2 = TOMLDateTime(date: Date())

let table = TOMLTable(["DateTime": dateTime, "DateTime2": dateTime2])
```

#### `Data`

```swift
import Foundation

let data = Data([0x01, 0x02, 0x03])

// `data` will be encoded as Base64.
let table = TOMLTable(["data": data])
```

#### Integers

Integers that don't need to be formatted will be converted to a `TOMLInt` automatically.
Use the `TOMLInt.init(_ value:options:)` initializer to format integers as octal, hexadecimal, or binary values.

### Retrieving TOML values

#### From Tables

```swift
let table: TOMLTable = ...

if let string = table["string"]?.string {
   print(string)
}

if let bool = table["bool"]?.bool {
   print(bool)
}

if let double = table["InnerTable"]?["InnerInnerTable"]?["InnerInnerInnerTable"]?["double"]?.double {
print(double)
}

...
```

#### From Arrays

```swift
let array: TOMLArray = ...

if let string = array[0].string? {
   print(string)
}

if let bool = array[1].bool? {
   print(bool)
}

if let double = array[2][0]?.double? {
   print(double)
}

for value in array {
   print(value)
}
```

### Encoding `struct` to TOML

```swift
struct TestStruct: Encodable {
	let string: String = "Hello, World!"
	let int: Int = 200823
}

let toml = try TOMLEncoder().encode(TestStruct())

print(toml)
```

### Decoding TOML to `struct`

```swift
struct TestStruct: Decodable {
	let string: String
	let int: Int
}

let testStruct = try TOMLDecoder().decode(TestStruct.self, from: "string = \"Hello, World!\"\nint = 405347")

print(testStruct)
```

## Dependencies

-   [toml++](https://github.com/marzer/tomlplusplus/)

## Licenses

The [toml++](https://github.com/marzer/tomlplusplus/) license is available in the `tomlplusplus` directory in the `LICENSE` file.

## Contributing

Before committing, please install [pre-commit](https://pre-commit.com), [swift-format](https://github.com/nicklockwood/SwiftFormat), [clang-format](https://clang.llvm.org/docs/ClangFormat.html), and [Prettier](https://prettier.io), then install the pre-commit hook:

```bash
$ brew bundle # install the packages specified in Brewfile
$ pre-commit install

# Commit your changes.
```

To install pre-commit on other platforms, refer to the [documentation](https://pre-commit.com/#install).
