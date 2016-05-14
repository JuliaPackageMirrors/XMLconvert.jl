
# XMLconvert
[![Build Status](https://travis-ci.org/bcbi/XMLconvert.jl.svg?branch=master)](https://travis-ci.org/bcbi/XMLconvert.jl)

This Julia package implements a few simple XML conversions. As of now, we can convert XMLs to nested `MultiDict` objects from the [DataStructure](https://github.com/JuliaLang/DataStructures.jl) package. We can also convert XMLs to JSONs. Note that as of this writing, we drop the attributes of the XML.


### Installation
```{Julia}
Pkg.clone("https://github.com/bcbi/XMLconvert.jl.git")
```

### Examples
For our examples we consider the following simple XML document. This toy example was borrowed (with slight modification) from the [LightXML](https://github.com/JuliaLang/LightXML.jl) package.
```{XML}
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book>
    <title>Biography of John Adams</title>
    <author>David Smith</author>
    <author>James Jones</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
  <book>
    <title>Introduction to Templates in C++</title>
    <author>Samantha Black</author>
    <year>2005</year>
    <price>29.99</price>
  </book>
  <owner>
    <name>Henry</name>
    <address>
      <state>CA</state>
      <street>123 Jones Avenue</street>
      <zip>12345</zip>
    </address>
    <age>59</age>
  </owner>
</bookstore>
```

Suppose we copy and paste the above into a file called `ex1.xml`.

### Reading in XML document
```{Julia}
using XMLconvert

filename = "ex1.xml"

# Read in XML and get its root
xdoc = parse_file(filename)
xroot = root(xdoc)

display(xroot)
```

### Convert to `MultiDict`
In many cases, it is desirable to convert an XML to a more native Julia object. This can be useful for unpacking elements of the XML and flattening out the structure of data. The `xml2dict()` function takes an XML's root (from above example) and converts the XML to a nested `MultiDict` object.
```{Julia}
# convert to MultiDict
xdict = xml2dict(xroot)
```
We can take a look at the structure of of the `MultiDict`.
```{Julia}
# print key structure of the MultiDict
show_key_structure(xdict)
```

```
-book
    -author
    -author
    -price
    -year
    -title
-book
    -author
    -price
    -year
    -title
-owner
    -name
    -age
    -address
        -zip
        -street
        -state
```

### Extracting Elements from `MultiDict`
Knowing the key structure of the XML we have parsed into a `MultiDict`, we can now access the elements much like we would using a standard `Dict` from Base Julia.
```{Julia}
xdict["book"][2]["Title"]
```

### Convert to JSON
If we wanted to convert the above XML to JSON we simply pass the parsed XML's root to the `xml2json()` function.
```{Julia}
json_string = xml2json(xroot)
print(json_string)
```

This produces a the following:
```{JSON}
{
"bookstore":

    {
    "book":[
        {
        "author":["David Smith","James Jones"],
        "price":30.0,
        "year":2005,
        "title":"Biography of John Adams"
        },
        {
        "author":"Samantha Black",
        "price":29.99,
        "year":2005,
        "title":"Introduction to Templates in C++"
        }
    ],
    "owner":
        {
        "name":"Henry",
        "age":59,
        "address":
            {
            "zip":12345,
            "street":"123 Jones Avenue",
            "state":"CA"
            }
        }
    }
}
```

### Write JSON to Disk
Finally, we can simply write that string to disk using Julia's standard `write()` function.
```{Julia}
f = open("ex1.json", "w")
write(f, json_string)
close(f)
```

### Spacing and Newline Characters
Note that the `xml2json()` function takes two optional arguments. The first controls the spacing of the indentation in the resulting JSON. This defaults to 4 (some prefer 8). The second optional argument (and therefore, third positional argument) controls how newline characters are handled. By default, this replaces `\n` with `\\n` in the JSON's text fields. This produces valid JSON documents.


### _Caveats_
This package is under active development. Please notify us of bugs or proposed improvements by submitting an issue or pull request.
