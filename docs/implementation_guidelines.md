# Implementation guidelines

## Validation

Tests are performed per row, per term (field) and are independent of each other, i.e. they can be performed in any order. Which tests to perform is defined in a settings file ([example](settings.yaml)), which has the following syntax:

```YAML
Term 1:
  Test 1
  Test 2

Term 2:
  Test 1
  Test 3
```
All validation information will be stored in a dictionary object, referring to the term, the type of test, and some logging information, in order to provide the user with feedback on possible improvements. 

* Number of tests that failed or the record IDs of those.
* Unique values that failed

```python
{
    Term: {
        TestType: {
            "ids": [id1, id2,...],
        }
    }
}
```

The resulting object will be processed by a report creator, which provides an overview of the current problems for each of the performed tests.

* [Implementation guidelines](#implementation-guidelines)
    * [Scope](#validation-scope)
    * [Order](#)
    * [Error reporting](#error-reporting)

## Coercing

It is important to understand that any tool using the specification file should expects all incoming fields as string types initially (so no automatic coercing or interpretation of data types). All implemented validation tests should be implemented based on a string as input. Hence, the `min` and `max` test will coerce the value to a `float` to make the numeric comparison. 

## Validation and reporting

## Validation

When validating data, the specifications are executed per row, per term (field) and independently (however, some test combinations require an order in the evaluation, see [specification order](#order) section). Which tests to perform is defined in a settings file (e.g. settings.yaml), which has the following syntax:

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

## Remarks

### Order
Testing some of these specifications do rely on the presence or properties of other specifications. This mainly involves the usage of the `empty: True` validation test, as this ccompares to the other tests as an `or`: either the value is `empty` OR it is not empty and all other tests need to be checked. 

In other words, the priority in the order of testing the different specifications, is as follows:

**empty > other specifications** where other tests are aborted when the value is empty.

The `if` and `delimitedvalues` specifications are providing an environment for which the specification need to be tested multiple times (taking into account this order for the individual tests):
* `if: First, the testing of the condition (does the *if* condition apply?) and secondly - if true - , the evaluation of the conditional specification of the term itself
* `delimitedvalues`: The defined specifications is evaluated for each of the individual terms as split by the delimiter, taking into account the order for each test individually.

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
