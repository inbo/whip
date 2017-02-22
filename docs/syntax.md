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
* [Changing scope](#changing-scope)
    * [empty](#empty)
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
  allowed: 'male'           # Same as above

sex:
  allowed: [male]           # Same as above

sex:
  allowed: [male, female]   # A list of allowed values: separate by commas and 
                            # wrap in square brackets. Will accept "male" or 
                            # "female".

sex:
  allowed: [male, female, 'male, female'] # Use quotes to escape commas and 
                            # white space. Will accept "male", "female" or 
                            # "male, female", but not "male,female" (no 
                            # white space) or "female, male".
```

Note: to pass, a value needs to be literally the same (= same sequence of characters) as (one of) the allowed value(s). This means that `allowed` is sensitive to case and white space.

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
observation_id:
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

Note: regular expressions allow to craft very specific specifications, but are often frustratingly difficult to get right. Use a tool like https://regex101.com to verify that they will match/unmatch what you intend.

Note: Always wrap the regex specification in single quotes. Not quoting will fail expressions containing `[ ]`, as they are interpreted by YAML as a list. Double quoting can cause escaped characters to be escaped again.

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

Tests if a numeric value conforms to a specific number format:

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

Tests if a date value is equal to or later than a minimum date:

```yaml
date:
  mindate: 1985-11-29       # Will accept "1985-11-29" and "2012-09-12", but 
                            # not "1942-11-26".
```

### maxdate

Tests if a date value is equal to or earlier than a maximum date:

```yaml
date:
  maxdate: 2012-09-12       # Will accept "2012-09-12" and "1985-11-29", 
                            # but not "2016-12-07".
```

### dateformat

Tests if a date value conforms to a specific date format. Syntax follows [strftime](http://strftime.org/):

```yaml
date:
  dateformat: '%Y-%m-%d'    # Will accept "2016-12-07", but not "2016/12/07", 
                            # "07-12-2016", "2016-12", or "2016-12-32" 
                            # (invalid date).

date:
  dateformat: ['%Y-%m-%d', '%Y-%m', '%Y'] # Will accept valid ISO8601 dates 
                            # "2016-12-07", "2016-12", and "2016".

date:
  dateformat: ['%Y-%m-%d/%Y-%m-%d'] # Will accept valid day-precise ISO8601 
                            # date ranges, such as "2016-01-01/2017-02-13"
```

Note: Always wrap the dateformat specification in single quotes. 

## Changing scope

By default, whip specifications reject empty values, apply to the whole content of a single field, and are independent from other fields. There are three methods to change that scope: `empty` allows empty values to pass specifications, `delimitedvalues` restricts the scope of specifications to individual delimited values within a field, and `if` makes specifications dependent on the value of another field.

### empty

Makes specifications accept empty values:

```yaml
# Default
sex:
  allowed: [male, female]   # Will accept "male" and "female", but not empty 
                            # values.

sex:
  allowed: [male, female]
  empty: False              # Same as above, implied by default.

# With empty: True
sex:
  empty: True               # Makes all specifications of this field to accept 
                            # empty values.
  allowed: [male, female]   # Will now accept "male", "female", and empty 
                            # values.

sex:
  allowed: [male, female]
  empty: True               # Same as above, order does not matter.
```

Note: Whip specifications will only accept empty values when `empty: True` is **explicitly** added as a specification. That means that the following specifications **will not accept empty values**, even though you might intuitively think so:

```yaml
field:
  maxlength: 2              # Will not accept empty values.

field:
  maxlength: 0              # Will not accept anything.

field:
  minlength: 0              # Will not accept empty values.

field:
  allowed: ''               # Will not accept empty values.

field:
  allowed: [male, female, ''] # Will not accept empty values.

field:
  regex: '^\s*$'            # Regex to match an empty string, will not accept 
                            # empty values.
```

Note: to *only accept empty values* (and nothing else), use:

```yaml
required_to_be_empty:
  allowed: ''
  empty: True
```

### delimitedvalues

Makes specifications apply to delimited values within a field, rather than the whole field. Requires `delimiter`:

```yaml
sex:
  delimitedvalues:
    delimiter: ' | '        # Required. Will use this to separate content 
                            # of a field. All specifications within the 
                            # "delimitedvalues" group apply to values 
                            # delimited with this delimiter.
    
    allowed: [male, female] # Will accept "male" or "female".
                            # Valid values for the whole field thus are: 
                            # "male", "female", "male | female", and 
                            # "female | male", but not "male, female" (wrong 
                            # delimiter), "male|female" (missing spaces), or 
                            # "male | " (empty second value).
  
  empty: True               # It is still possible to set specifications for 
                            # the whole field. Here it is specified that the 
                            # whole field can be empty (but delimited values 
                            # cannot).
```

Note: to specify that a field cannot contain empty delimited values (but without defining other specifications for those values), use:

```yaml
list_of_names:
  empty: true               # The whole field can be empty...
  delimitedvalues:
    delimiter: ' | '        # .. but using this delimiter, delimited values 
                            # cannot be empty, since "empty: True" is implied 
                            # by default.
```

### if

Makes specifications conditional. This means that they are only verified if another field of the same record (i.e. the same row in tabular format) successfully passes certain specifications:

```yaml
lifestage:
  if:
    - sex:
        allowed: [male, female] # If sex is "male" or "female"...
      allowed: adult        # ... then lifestage needs to be "adult".
    - sex:
        allowed: ''         # If sex is empty (and nothing else)...
        empty: True
      allowed: ''           # ... then lifestage needs to be empty.
      empty: True
```

Note: Always use the correct indentation and `-` to define a new condition:

```yaml
field_1:
  if:
    - field_2:              # Condition A
        spec_for_field_2: value
        spec_for_field_2: value
      conditional_spec_for_field_1: value
      conditional_spec_for_field_1: value

    - field_2:              # Condition B
        spec_for_field_2: value
      conditional_spec_for_field_1: value
      conditional_spec_for_field_1: value
```

Note: conditions are independent from each other. This means that certain records can pass one condition, but fail another. In the example below a record with `sex = male` and `lifestage = adult` will pass the first condition, but fail the second. Avoid using very broad conditions:

```yaml
lifestage:
  if:
    - sex:
        allowed: [male, female] # If sex is "male" or "female"...
      allowed: adult        # ... then lifestage needs to be "adult".
    - sex:
        empty: False        # If sex is any non-empty value, which includes 
                            # "male" or "female"...
      allowed: unknown      # ... then lifestage needs to be "unknown".
```

Note: the value of a field on which a condition is based needs to successfully pass all specifications (i.e. they are combined with the `AND` operator) before the conditional specifications are tested:

```yaml
province:
  if:
    - postalcode:
        type: integer       # province needs to be an integer
        min: 8000           # AND province needs to be larger or equal to 8000
        max: 8999           # AND province needs to be smaller or equal to 8999
      allowed: 'West Flanders' # Only then it is tested if province is 
                            # "West Flanders".
```
