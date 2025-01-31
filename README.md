elm-test [![Build Status](https://travis-ci.org/elm-community/elm-test.png?branch=master)](https://travis-ci.org/elm-community/elm-test)
========

A unit testing framework for Elm

## Creating Tests

Creating a test case is very simple. You only need a name and an assertion:
```elm
myTest = test "Example Test" (assert True)
```
For convenience, there is a function to create a name for you based on the inputs:
```elm
-- Test name will be "5 == 5"
myTest = defaultTest (assertEqual 5 5)
```
As well as a function to create an `assertEqual` tests, again deriving a name based on the inputs:
```elm
myTest = defaultTest (5 `assertEqual` 5)
```
There are five different functions to create assertions:
```elm
assert : Bool -> Assertion
assertEqual : a -> a -> Assertion
assertNotEqual : a -> a -> Assertion
lazyAssert : (() -> Bool) -> Assertion 
assertionList : List a -> List a -> List Assertion
```
Example usage of these functions might be:
```elm
assert        (a > 5)             -- Returns an AssertTrue assertion
assertEqual    a b                -- Returns an AssertEqual assertion
assertNotEqual a b                -- Returns an AssertNotEqual assertion
lazyAssert (\_ -> a > 5)          -- Same as the assert example, but delays execution until test runtime
assertionList [a, b, c] [d, e, f] -- Shorthand for [assertEqual a d, assertEqual b e, assertEqual c f]
```
The `lazyAssert` function can be useful for testing functions which might possibly cause a runtime error. With all the
other assertion functions, the tests are actually run when the file is loaded, which can cause runtime errors
on page load, but with `lazyAssert`, any runtime errors are delayed until actual test execution. Note that for this to
work, you must manually write an anonymous function of type `() -> Bool`;

## Grouping Tests

Writing many tests as a flat list quickly becomes unwieldy. For easier maintenance you can group tests into logical
units called test suites. The following function will create a test suite from a suite name and a list of tests:
```elm
suite : String -> List Test -> Test
```
The type of a test suite is simply `Test`, allowing use of all the test runners with either a single test or a suite of
tests. Test suites can also contain subsuites, of course.

The other benefit of grouping tests into suites is that the test runners described in the following sections will greatly
simplify the output, showing only detailed information in suites that contain failed tests, making it easier to quickly spot
the failures instead of being flooded with irrelevant data.

## Running Tests

The simplest way to run tests and display the output is the `elementRunner : Test -> Element` function, which is an easy way
to run your tests and report the results in-browser, as a standard Elm module. A full example could be:
```elm
-- Example.elm
import String
import Graphics.Element exposing (Element)

import ElmTest exposing (..)


tests : Test
tests = 
    suite "A Test Suite"
        [ test "Addition" (assertEqual (3 + 7) 10)
        , test "String.left" (assertEqual "a" (String.left 1 "abcdefg"))
        , test "This test should fail" (assert False)
        ]


main : Element
main = 
    elementRunner tests
```
Compile this with `elm-make Example.elm --output Example.html` and open the resulting file in your browser, and you'll see
the results.

Another method is the `stringRunner : Test -> String` function. This is almost the same, but it returns a `String` instead of
an `Element`. The `String` is a summary of the overall test results. Here's the same example as before, but modified for
`stringRunner`:
```elm
-- Example.elm
import String
import Graphics.Element exposing (Element, show)

import ElmTest exposing (..)


tests : Test
tests = 
    suite "A Test Suite"
        [ test "Addition" (assertEqual (3 + 7) 10)
        , test "String.left" (assertEqual "a" (String.left 1 "abcdefg"))
        , test "This test should fail" (assert False)
        ]


main : Element
main = 
    show (stringRunner tests)
```

You can also run these tests from command line with `runSuite : Test -> Program Never`, see the below section on **Testing from the Command Line** for details.

## Demo

For a quick demo, you can compile the `ElementExample.elm` file, or continue to the next section:

## Testing from the Command Line
Make a file that uses the `runSuite` function:
```elm
module Tests exposing (..)

import ElmTest exposing (..)

tests : Test
tests =
    suite "A Test Suite"
        [ test "Addition" (assertEqual (3 + 7) 10)
        , test "String.left" (assertEqual "a" (String.left 1 "abcdefg"))
        , test "This test should fail" (assert False)
        ]

main : Program Never
main =
    runSuite tests
```
Then compile it to JS file and run it with node:
```bash
$ elm-make Tests.elm --output tests.js
$ node tests.js
```

While the `elementRunner` display is nicest to read, the `runSuite` function is amenable to automated testing. If a test
suite passes the script will exit with exit code 0, and if it fails it will exit with 1.

## Integrating With Travis CI

With elm-test and elm-console, it is possible to run continuous integration tests with Travis CI on
your Elm projects. Just set up Travis CI for your repository as normal, write tests with elm-test,
and include a `.travis.yml` file based on the following:
```
language: haskell
install:
  - npm install -g elm
  - elm-package install -y
before_script: 
  - elm-make --yes --output test.js tests/Tests.elm
script: node test.js
```
You can look at the `.travis.yml` file in this repository to see a real example.
