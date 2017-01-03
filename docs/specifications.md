# DwCASpecifications

## Validation

When performing an evaluation, the evaluation of the DwCA specifications are performed per row, per term (field) and act independent of each other (however, some test combinations require an order in the evaluation, see [specification order](#order) section). Which tests to perform is defined in a settings file (e.g. settings.yaml), which has the following syntax:

```YAML
Term 1:
  Specification 1
  Specification 2

Term 2:
  Specification 1
  Specification 3
```

The set of specifications for each term describes the data characteristics of each of the terms (i.e. columns). These rules should be tool (software) independent, but anyone can develop software or applications that are able to understand these specification files and executes tests to evaluate a data against these specification files.

In order to provide maximum interoperability amongst different developments, a common error description (providing information about the individual failures) is proposed. The validation information will be stored as json, referring to the line of the data set, the term and the type of test as a minimal exchangeable output format:

```json
{
lineID1:
    {
    term1: [specification failure 1,
            specification failure 2,
            ...],
    term2: [specification failure 1,
            specification failure 2,
            ...],
    ...},
lineID2: {...},
...
}
```

This `json` object can be processed by a report creator, which provides an overview of the current problems for each of the performed tests. By having a common error description model, different reporting styles and tools can easily be exchanged.

Hence, development of tools based on these specifications can be split into two categories:
* Evaluation tools: from specification file (settings.yaml) to error reporting (json error description object)
* Reporting tools: from error reporting (json error description object) to any kind of user-friendly summary of the data set disagreements to the specification file

## Specification rules

The current proposed set of rules are:

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
* [delimitedvalues](#delimitedvalues)
* [if](#if)

### type

Does the data conform to a specific field type?

The defined DwCA specifications `type` provides a set of options to test the data type property of the field:

* string
* integer
* float
* number (integer or float)
* boolean
* url
* json

**Remark** datetime or date is not included as an available `type` specification, as date formats are provided as a separate rule for dates, embedded in the test [#dateformat](#dateformat).

```YAML
# Expects: string
# Records without data: are ignored
# Records of wrong data type: is being tested

type: integer
type: float
type: boolean
type: json
type: url
```

It is important to understand that any tool using the specification file should expects all incoming fields as string types initially (so no automatic coercing or interpretation of data types). When no `type` specification is applied, the value will be interpreted and tested as a string value. By incorporating a `type` specification, the validation tool will **first** interpret the value as the defined `type` to test it for (e.g. integer, float). When succeeded, the other tests will be applied on the interpreted value (integer, float).


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

## Order
Testing some of these specifications do rely on the presence or properties of other specifications, e.g. the interpretation of a `numberformat` is only possible if the appropriate type is present, as defined by a `type` specification. Instead of implicitly deriving this information (i.e. automatically testing for a number data type when  a `min`, `max` or `numberformat`), this should be explicitly defined by the user by adding a `type` specification. 

The priority in the order of testing the different specifications, is as follows:

**empty > type > other specifications**

The `if` and `delimitedvalues` specifications are providing an environment fr which the specification need to be tested multiple times (taking into account this order for the individual tests):
* `if: First, the testing of the condition (does the *if* condition apply?) and secondly - if true - , the evaluation of the conditional specification of the term itself
* `delimitedvalues`: The defined specifications is evaluated for each of the individual terms as split by the delimiter, taking into account the order for each test individually.

## Remarks

### min/max combination
In order to keep the number of rules to a minimum,  it is decided to not have a separate test `equals`, as this can easily be achieved by combining a `min` and `max`  test:

```
equals: 10
```
is equal to this test description:
```
min: 10
max: 10
```

The same is true for the specification  `length`, which is a combination of `minlength` and  `maxlength`.

