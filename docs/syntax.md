# Whip syntax

This document specifies how to express data specifications in whip. How meta! :metal:

* [General](#general)
    * [A single specification](#a-single-specification)
    * [Multiple specifications](#multiple-specifications)
    * [A specification file](#specification-file)
* [Specification types](#specification-types)
    * [type](#type)
    * [minlength](#minlength)
    * [maxlength](#maxlength)
    * [min](#min)
    * [max](#max)
    * [allowed](#allowed)
    * [empty](#empty)
    * [mindate](#mindate)
    * [maxdate](#maxdate)
    * [numberformat](#numberformat)
    * [dateformat](#dateformat)
    * [regex](#regex)
* [Defining scope](#defining-scope)
    * [delimitedvalues](#delimitedvalues)
    * [if](#if)

## General

Whip specifications are expressed in [YAML](https://en.wikipedia.org/wiki/YAML).

### A single specification

A specification in whip describes what you want a data value in a single field to adhere to. For example, to express that the field `date` should always contain the literal value `2016-12-06`, you write:

```yaml
date:                   # name of the field
  allowed: 2016-12-06   # specification
```

### Multiple specifications

You can define multiple specifications for a single field. For example, if you want to express that `date` values should fall between `2016-01-01` and `2016-12-31`, you write:

```yaml
date:
  mindate: 2016-01-01
  maxdate: 2016-12-31
```

To add specifications for another field, just add the name of the field and its specification(s):

```yaml
date:
  mindate: 2016-01-01
  maxdate: 2016-12-31

sex:
  allowed: [male, female]
```

### A specification file

Together, all these field/term-based specifications form a specification file (e.g. `my_specifications.yaml`), which can be used by a whip validator to test how well data meets certain specifications. In other words, you test if your data cracks under your whip, which can range from a feather :sweat_smile: to a chain whip :scream:.

## Specification types

### type (deprecated)

Tests if a value is of a specific data type:

```yaml
myNumberField:
  type: number          # Will accept 2 or 2.1 or "2", but not "Two"
```

The options for `type` are:

```yaml
myField:
  type: string
  type: integer
  type: float
  type: number           # An integer or a float
  type: boolean
  type: url
  type: json

# Obviously, you wouldn't add all these type specifications for a single field
```

Remarks:

* A `date` or `datetime` option is not included. See [dateformat](#dateformat) instead.
* See the examples below for how certain values are interpreted.

Example | string | integer | float | number | boolean | url | json
--- | --- | --- | --- | --- | --- | --- | --- 
`Hello World!` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:
`1` | :heavy_check_mark: | :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: | :x: | :x:
`1.0` | :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: | :x: | :x: | :x:
`1.3` | :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: | :x: | :x: | :x:
`0` | :heavy_check_mark: | :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: | :x: | :x:
`True` | :heavy_check_mark: | :x: | :x: | :x: | :heavy_check_mark: | :x: | :x:
`Yes` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:
`http://github.com/inbo/whip` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :heavy_check_mark: | :x:
`{"length": 2.0}` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :heavy_check_mark:
`Zero` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:
`Null` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:
`""` | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:
empty value | :heavy_check_mark: | :x: | :x: | :x: | :x: | :x: | :x:

### minlength

Is the length of the data string larger than the given value (inclusive the value itself)?

```YAML
# Expects: integer
# Records without data: are ignored
# Records of wrong data type: only active with strings

minlength: 8  # Character length is larger than 8
```

### maxlength

Is the length of the data string smaller than the given value (inclusive the value itself)?

```YAML
# Expects: integer
# Records without data: are ignored
# Records of wrong data type: only active with strings

maxlength: 20  # Character length is smaller than 20
```

**Remark:**

The behaviour of length (`minlength` and `maxlength`) depends on the usage of the `length` validation in combination with the `type` validation or not. When no `type` validation is added (or tests for string, which is default), the length will interpret the field as string:

```YAML
maxlength: 2  
type : string  
```
will invoke an error for the field : `{'individualCount' : '100'}`. However, when requiring an integer value:

```YAML
maxlength: 2
type : integer
```
the field `{'individualCount' : '100'}` will be converted to `{'individualCount' : 100}` (100 as integer) and `length` will ignore the integer. It makes more sense to test this with the `min` and `max` validators (see further).

### min

Minimum value allowed for integer/float and number formats. Whereas this works for different numeric types, evaluation is done on a float representation.

```YAML
# Expects: int/float; values will be compared as floats
# Records without data: are ignored
# Records of wrong data type: ignore (data types are tested separately with `type`)

min: 0.5     # float
min: 20     # integer
```

**Remark**

It is important to combine the test with an appropriate data type validation to enable the test when using numeric values

The usage is of particular interest if values should be accepted, but the decimal precision is not important. For example: 0.75 will also accept 0.750 and 200 also 200.0. When the value need to be exactly the same, the usage of `allowed` (works on strings) is advisable (see next).

### max

Maximum value allowed for integer/float and number formats. Whereas this works for different numeric types, evaluation is done on a float representation.

```YAML
# Expects: int/float; values will be compared as floats
# Records without data: are ignored
# Records of wrong data type: ignore (data types are tested separately with `type`)

max: 0.75     # float
max: 200     # integer
```

**Remark**

It is important to combine the test with an appropriate data type validation to enable the test when using numeric values

The usage is of particular interest if values should be accepted, but the decimal precision is not important. For example: 0.75 will also accept 0.750 and 200 also 200.0. When the value need to be exactly the same, the usage of `allowed` (works on strings) is advisable (see next).

### allowed

Does the data is the same sequence of characters?

```YAML
# Expects: list with one or more strings
# Records without data: are ignored
# Records of wrong data type: all considered strings

allowed: [male]
allowed: [male, female] # male or female
```

### empty

Empty values are default not accepted. If empty values are allowed for a particular field, `empty` can be put to `True`. Hence, this is a shortcut to `allowed: ['']`.

```YAML
# Expects: boolean
# Records of wrong data type: only considered strings (default in Dwc)

empty: True
```

### mindate

Does the date/datetime object happens before a given date?

```YAML
# Expects: date string
# Records without data: are ignored
# Records of wrong data type: fail test

mindate: 1830-01-01  # After 1 Jan 1830
mindate: 2014-10-20 # After 20 October 2014
```

### maxdate

Does the date/datetime objects happens after a given date?

```YAML
# Expects: date string
# Records without data: are ignored
# Records of wrong data type: fail test

maxdate: 1830-01-01  # After 1 Jan 1830
maxdate: 2014-10-20 # After 20 October 2014
```

### numberformat

Does the float number conform to a specific precision format?

```YAML
# Expects: string and need to be combined with `type` : float validator
# Records without data: are ignored
# Records of wrong data type: fail test

numberformat: '.5' # 5 decimals, left side of the decimal not specified, e.g. 1.12345 or 12.12345
numberformat: '3.5' # 3 digits at the left side of the decimal and 5 decimal digits, e.g. 123.12345
numberformat: '4.' # 4 digits at the left side, right side not specified, e.g. 1234., 1234.12 or also the integer 1234
```

### dateformat

Does the data conform to a specific date format? Possibilities provided at
http://strftime.org/

```YAML
dateformat:['%Y-%m-%d', '%Y-%m', '%Y'] # Will match specific date formats
dateformat:['%Y-%m-%d/%Y-%m-%d'] # Will match a date range
```

### regex

Does the data match a specific regex expression?

```YAML
# Expects: regex expression
# Records without data: are ignored!
# Records of wrong data type: all considered strings

regex: # No example yet
```

## Defining scope

By default, whip specifications are field-based (i.e. the apply to the whole content of a single field) and independent from other fields. There are two methods to change that scope: `delimitedvalues` restricts the scope of specifications to individual delimited values within a field, while `if` makes specifications dependent on the value in another field.

### delimitedvalues

Environment wrapper to work on delimited data within a field. Will alter the evaluation of all specifications to work with the delimited data instead of the whole string. Requires `delimiter`.

```YAML
delimitedvalues:
  delimiter: " | "  # Will use this delimiter for separating values.
                    # Depending on how well data is delimited, the
                    # following tests will fail or succeed
  required: true   # No empty delimited values
  type: url
  allowed: [male, female] # Delimited values equal male or female
  minlength: 8
  maxlength: 8
  min: 1
  max: 1
  numberformat: .3f
  regex: ...
  listvalues: true  # List unique delimited values across all records - TODO
  dateformat: ...   # Use datevalues subfunction
```

### if

Environment wrapper to define condition specifications, i.e. specifications of a term are  based on the specifications of another term. All conditional tests on the other term must succeed (i.e. they are combined with `AND`) before the specifications are evaluated.

```YAML
if:
    basisOfRecord:
        allowed: [HumanObservation]
    allowed: [Event]
    empty: False
```

To combine multiple if-statements, these need to be grouped as follows:

```YAML
if:
    - age:
          min: 20
          type: integer
      allowed: [adult]
    - age:
          min: 20
          type: integer
      maxlength: 6
```


