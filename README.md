Package validator
================

Package validator implements variable validations

This differs from the original gopkg.in/validator.v2 in the following ways:

- Uses pseudo-querystring for parameters
- Properly marshals errors as JSON
- Can read `json` struct tag for field names
- Validator functions accepts a map[string]string instead of a string

Installation
============

Just use go get.

	go get gopkg.in/mgutz/validator.v2

And then just import the package into your own code.

	import (
		"gopkg.in/mgutz/validator.v2"
	)

Usage
=====

Please see http://godoc.org/gopkg.in/mgutz/validator.v2 for detailed usage docs.
A simple example would be.

	type NewUserRequest struct {
		Username string `validate:"min?3,max?40,regexp?^[a-zA-Z]*$"`
		Name string     `validate:"nonzero&err=is required"`
		Age int         `validate:"min?21"`
		Password string `validate:"min?8&err=is too short"`
	}

	nur := NewUserRequest{Username: "something", Age: 20}
	if errs := validator.Validate(nur); errs != nil {
		// values not valid, deal with errors here
	}

Parameters are passed as a comma delimited pseudo-querystring
(deviating from go convention of key=value). The key name is optional.
If a key name is not provided then a zero-based ordinal value in string
form will be assigned in the order it was discovered. For example,

    min?123&foo&err=custom error message,nonzero

will be parsed as two validators

    validator == "min"
    param == {"0": "123", "1": "foo", "foo": "bar"}
    // err key is used to assign a custom error message
    customError == "custom error message"

    functionName == "nonzero"
    param2 == nil


Builtin validators

Here is the list of validators buildin in the package.

	len
		For numeric numbers, max will simply make sure that the
		value is equal to the parameter given. For strings, it
		checks that the string length is exactly that number of
		characters. For slices,	arrays, and maps, validates the
		number of items. (Usage: len?10)

	max
		For numeric numbers, max will simply make sure that the
		value is lesser or equal to the parameter given. For strings,
		it checks that the string length is at most that number of
		characters. For slices,	arrays, and maps, validates the
		number of items. (Usage: max?10)

	min
		For numeric numbers, min will simply make sure that the value
		is greater or equal to the parameter given. For strings, it
		checks that the string length is at least that number of
		characters. For slices, arrays, and maps, validates the
		number of items. (Usage: min?10)

	nonzero
		This validates that the value is not zero. The appropriate
		zero value is given by the Go spec (e.g. for int it's 0, for
		string it's "", for pointers is nil, etc.) For structs, it
		will not check to see if the struct itself has all zero
		values, instead use a pointer or put nonzero on the struct's
		keys that you care about. (Usage: nonzero)

	regexp
		Only valid for string types, it will validator that the
		value matches the regular expression provided as parameter.
		(Usage: regexp=^a.*b$)

Custom validators

It is possible to define custom validators by using SetValidationFunc.
First, one needs to create a validation function.

	// Very simple validator
	func notMatch(v interface{}, param map[sring]string) error {
		st := reflect.ValueOf(v)
		if st.Kind() != reflect.String {
			return errors.New("notMatch only validates strings")
		}
		if st.String() != param["0"] {
			return errors.New("value does not match")
		}
		return nil
	}

Then one needs to add it to the list of validators and give it a "tag"
name.

	validator.SetValidationFunc("notMatch", notMatch)

Then it is possible to use the notzz validation tag. This will print
"Field A error: value cannot be ZZ"

	type T struct {
		A string  `validate:"nonzero,notMatch?"secret" json:"a"`
	}
	t := T{"ZZ"}
	if errs := validator.Validate(t); errs != nil {
		fmt.Printf("Field A error: %s\n", errs["a"][0])
	}

You can also have multiple sets of validator rules with SetTag().

	type T struct {
		A int `foo:"nonzero" bar:"min?10"`
	}
	t := T{5}
	SetTag("foo")
	validator.Validate(t) // valid as it's nonzero
	SetTag("bar")
	validator.Validate(t) // invalid as it's less than 10

SetTag is probably better used with multiple validators.

	fooValidator := validator.NewValidator()
	fooValidator.SetTag("foo")
	barValidator := validator.NewValidator()
	barValidator.SetTag("bar")
	fooValidator.Validate(t)
	barValidator.Validate(t)

This keeps the default validator's tag clean. Again, please refer to
godocs for a lot of more examples and different uses.


JSON support

To use `json` struct tag names instead of the struct field name, set
`validator.SetReadJSONTag(true)`.

    func init() {
        validator.SetReadJSONTag(true)
    }

    type Foo stuct {
        A int `validate:"nonzero" json:"littleA"`
    }

    // returns {"littleA": "zero value"} instead of {"A": "zero value"}
    validator.Validate(&Foo{})

The `error` return from Validate will marshal properly as a JSON object.

Pull requests policy
====================

tl;dr. Contributions are welcome.

The repository is organized in version branches. Pull requests to, say, the
`v2` branch that break API compatibility will not be accepted. It is okay to
break the API in master, *not in the branches*.

As for validation functions, the preference is to keep the main code simple
and add most new functions to the validator-contrib repository.

https://github.com/go-validator/validator-contrib

For improvements and/or fixes to the builtin validation functions, please
make sure the behaviour will not break existing functionality in the branches.
If you see a case where the functionality of the builtin will change
significantly, please send a pull request against `master`. We can discuss then
whether the changes should be incorporated in the version branches as well.

License
=======

Copyright 2014 Roberto Teixeira <robteix@robteix.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
