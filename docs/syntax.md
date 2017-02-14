# Whip syntax

This document specifies how to express data specifications in whip. How meta! 🤘

* [General](#general)
    * [A single specification](#a-single-specification)
    * [Multiple specifications](#multiple-specifications)
    * [A specification file](#specification-file)
* [Specification types](#specification-types)
    * [allowed](#allowed)
    * [minlength](#minlength)
    * [maxlength](#maxlength)
    * [stringformat](#stringformat)
    * [regex](#regex)
    * [min](#min)
    * [max](#max)
    * [numberformat](#numberformat)
    * [mindate](#mindate)
    * [maxdate](#maxdate)
    * [dateformat](#dateformat)
    * [empty](#empty)
* [Defining scope](#defining-scope)
    * [delimitedvalues](#delimitedvalues)
    * [if](#if)

## General

Whip specifications are expressed in [YAML](https://en.wikipedia.org/wiki/YAML), a human and machine-readable data serialization language.

### A single specification

A specification in whip describes what a data value in a field should adhere to. For example, to express that the field `age` should always contain the literal value `33`, use:

```yaml
age:                       # Name of the field
  allowed: 33              # Specification
```

### Multiple specifications

Multiple specifications can be defined for a field. For example, to express that `age` values should fall between `9` and `99`, use:

```yaml
age:
  min: 9
  max: 99
```

To add specifications for another field, just add the name of the field and its specification(s):

```yaml
age:
  min: 9
  max: 99

sex:
  allowed: [male, female]
```

### A specification file

Together, all these field/term-based specifications form a specification file (e.g. `my_specifications.yaml`), which can be used by a validator to test how well data meets certain specifications.

## Specification types

### allowed

Tests if a value is the same as an allowed value or belongs to a list of allowed values:

```yaml
sex:
  allowed: male             # A single allowed value. Will accept "male", but 
                            # not "Male" (case-sensitive) or anything else.

sex:  
  allowed: "male"           # Same as above

sex:
  allowed: [male]           # Same as above

sex:
  allowed: [male, female]   # A list of allowed values: separate by commas and 
                            # wrap in square brackets. Will accept "male" or 
                            # "female".

sex:
  allowed: [male, female, "male, female"] # Use quotes to escape commas and 
                            # whitespace. Will accept "male", "female" or 
                            # "male, female", but not "male,female" (no 
                            # whitespace) or "female, male".
```

Note: to pass, a value needs to be literally the same (= same sequence of characters) as (one of) the allowed value(s). This means that `allowed` is sensitive to case and whitespace.

### minlength

Tests if a value has a minimum number of characters:

```yaml
postal_code:
  minlength: 4              # Will accept "9050" and "B-9050", but not "905".
```

### maxlength

Tests if a value has a maximum number of characters:

```yaml
license_plate:
  maxlength: 6              # Will accept "AF8934" and "AF893", 
                            # but not "AF8-934" (note the dash)
```

### stringformat

Tests if a value conforms to a specific string format (`url` or `json`):

```yaml
website:
  stringformat: url         # Will accept "http://github.com/inbo/whip", 
                            # including urls with http, querystrings and 
                            # anchors, but not "github.com/inbo/whip"

measurements:
  stringformat: json        # Will accept {"length": 2.0} and 
                            # {"length": 2.0, "length_unit": "cm"}, but not 
                            # {'length': 2.0} or {length: 2.0} (use double 
                            # quotes) or "length": 2.0 (use curly brackets).
```

### regex

Tests if a value matches a regular expression (regex):

```yaml
observation_id
  regex: 'INBO:VIS:\d+'     # Will accept "INBO:VIS:12" and "INBO:VIS:456", 
                            # but not "INBO:VIS:" or "INBO:VIS:ABC"

issue_url:
  regex: 'https:\/\/github\.com\/inbo\/whip\/issues\/\d+' # Don't forget to 
                            # escape (using "\") reserved characters like "." 
                            # and "/". Will accept "https://github.com/inbo/
                            # whip/issues/4"

utm1km:
  regex: '31U[D-G][S-T]\d\d\d\d' # Will accept UTM 1km codes for Flanders, 
                            # e.g. "31UDS8748"
```

Note: regular expressions allow to craft very specific specifications, but are often frustrately difficult to get right. Use a tool like https://regex101.com to verify that they will match/unmatch what you entend.

Note: Always single quote the regex. Not quoting will fail expressions containing `[ ]`, as they are interpreted by YAML as a list. Double quoting can cause escaped characters to be escaped again.

### min

Tests if a numeric value is equal to or higher than a minimum value:

```yaml
age:
  min: 9                    # Will accept "9", "9.0", "9.1", "10", but not 
                            # "8.99999" or "-9".

age:
  min: 9.0                  # Same as above
```

### max

Tests if a numeric value is equal to or lower than a maximum value:

```yaml
age:
  max: 99                   # Will accept "99", "99.0", "89.9", "88", "-99", 
                            # but not "99.1".

age:
  max: 99.0                 # Same as above
```

### numberformat

Tests if a value conforms to a specific number format:

```yaml
length:
  numberformat: .3          # Will accept numbers with 3 digits to the right 
                            # of the decimal point, such as ".123", "1.123" 
                            # and "12.123", but not "1.12", "1.1234" or 
                            # "a.abc".

length:
  numberformat: 2.          # Will accept numbers with 2 digits to the left 
                            # of the decimal point, such as "12", "12." and 
                            # "12.1", but not "123".

length:
  numberformat: 2.3         # Will accept numbers with 2 digits to the left 
                            # and 3 digits to the right of the decimal point, 
                            # such as "12.123".
```

### mindate

Tests if a date is equal to or later than a minimum date:

```yaml
date:
  mindate: 1985-11-29       # Will accept "1985-11-29" and "2012-09-12", but 
                            # not "1942-11-26".
```

### maxdate

Tests if a date is equal to or earlier than a maximum date:

```yaml
date:
  maxdate: 2012-09-12       # Will accept "2012-09-12" and "1985-11-29", 
                            # but not "2016-12-07".
```

### dateformat

Does the data conform to a specific date format? Possibilities provided at
http://strftime.org/

```YAML
dateformat:['%Y-%m-%d', '%Y-%m', '%Y'] # Will match specific date formats
dateformat:['%Y-%m-%d/%Y-%m-%d'] # Will match a date range
```

### empty

Empty values are default not accepted. If empty values are allowed for a particular field, `empty` can be put to `True`. Hence, this is a shortcut to `allowed: ['']`.

```YAML
# Expects: boolean
# Records of wrong data type: only considered strings (default in Dwc)

empty: True
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



