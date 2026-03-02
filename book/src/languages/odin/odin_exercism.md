# Odin - Walkthroughs of Small Examples

As a supplement to the [Odin Introduction](odin_data_types.md), here are some walkthroughs of very small Odin programs.

Language features that weren't covered in the prior material will be explained as they come up.

> [NOTE!]
> These code examples mainly come from the [Exercism project](https://exercism.org/tracks/odin) and are licensed under the [MIT License](https://opensource.org/license/mit)

## Setup

The Git repo is [here](https://github.com/exercism/odin/tree/main). The examples are found under `exercises/practice`.

The goal of each exercise is to pass the provided tests. To run the tests for exercise `foo`, run the command `odin test exercises/practice/foo` (assuming CWD is root of the repo).

## Exercise: Pangram

The `is_pangram` procedure determines if its string argument contains every letter of the English alphabet (case insensitive).

```go
package pangram

is_pangram :: proc(str: string) -> bool {

	// defines Alphabet as a bit set type which has a bit
	// for every character in the range 'a' up to (and including) 'z'
	Alphabet :: bit_set['a' ..= 'z']

    // zero value of a bit set is empty (all bits unset)
    expected: Alphabet
	found: Alphabet

    // an Alphabet literal with all bits set
	expected = Alphabet {
		'a',
		'b',
		'c',
		'd',
		'e',
		'f',
		'g',
		'h',
		'i',
		'j',
		'k',
		'l',
		'm',
		'n',
		'o',
		'p',
		'q',
		'r',
		's',
		't',
		'u',
		'v',
		'w',
		'x',
		'y',
		'z',
	}

	// alternatively, we can get the full set 
    // by doing a logical not operation on the empty set
	expected = ~Alphabet{}  // same result as prior assignment

	UPPER_TO_LOWER_DIFF :: 'a' - 'A'

	// for every rune in the string
	// (a "rune" is an unsigned integer representing a Unicode code point)
	for r in str {
		if r >= 'a' && r <= 'z' {     // if lowercase...
			// adding two bit sets returns their union,
			// so this is effectively setting the bit for 'r' in 'found'
			found += Alphabet{r}
		} else if r >= 'A' && r <= 'Z' {   // if uppercase...
			found += Alphabet{r + UPPER_TO_LOWER_DIFF}
			// found += Alphabet{r - 'A' + 'a'}
		} else {  // if not a letter...
			continue
		}
	}
	return found == expected
}
```


Here are some of the tests for this exercise:

```go
package pangram

import "core:testing"

// The @(test) attribute marks the procedure as a test.
// A test procedure must have one and only one parameter of type ^testing.T
// The parameter name does not matter, but 't' is used by convention.
// Most core procedures of the testing package take the ^T param as their 
// first argument, and this is used to track the test state.

@(test)
/// description = empty sentence
test_empty_sentence :: proc(t: ^testing.T) {
    // a test fails if expect() is passed false
	testing.expect(t, !is_pangram(""))
}

@(test)
/// description = only lower case
test_only_lower_case :: proc(t: ^testing.T) {
	testing.expect(t, is_pangram("the quick brown fox jumps over the lazy dog"))
}

@(test)
/// description = missing the letter 'x'
test_missing_the_letter_x :: proc(t: ^testing.T) {
	testing.expect(t, !is_pangram("a quick movement of the enemy will jeopardize five gunboats"))
}

// etc.... 
```

## Exercise: Reverse String

The `reverse` procedure takes a string and returns the string which is the reverse of its characters (or more accurately, its grapheme clusters).

```go
package reverse_string

import "core:strings"
import "core:unicode/utf8"

reverse :: proc(str: string) -> string {
	// A Grapheme is a cluster of one or more Unicode code points that 
	// represent what a user perceives as an individual character. 
	// (Not all Unicode code points represent individual, 
    // complete renderable characters.)
	graphemes: [dynamic]utf8.Grapheme

	// Returns allocated dynamic array of graphemes.
	// Also returns the grapheme count, rune count, and monospacing width, 
	// but we discard these values by assigning them to _
	graphemes, _, _, _ = utf8.decode_grapheme_clusters(str)

	// We want to deallocate the dynamic array when leaving this procedure, 
	// so we call delete with a defer statement.
	defer delete(graphemes)

	sb := strings.builder_make()

	// #reverse makes this loop iterate through the array backwards
	// g = utf8.Grapheme
	// i = int index
	#reverse for g, i in graphemes {
		data: string
		if i == len(graphemes) - 1 {
			// if last grapheme in the array...
			data = str[g.byte_index:]
		} else {
			// Determine size of the current grapheme in bytes.
			next := graphemes[i + 1]
			num_bytes := next.byte_index - g.byte_index
			
			// Slicing a string produces a new string.
			// To get the bytes of the current grapheme, we slice 
            // twice to get the string starting at g.byte_index 
            // and having length num_bytes.
			data = str[g.byte_index:][:num_bytes]
		}

		// Copy the bytes of the current grapheme to the string builder.
		strings.write_string(&sb, data)
	}

	// Return the string builder's data as a regular string.
	return strings.to_string(sb)
}
```

Here are some tests from this exercise:

```go
package reverse_string

import "core:testing"

@(test)
/// description = a capitalized word
test_a_capitalized_word :: proc(t: ^testing.T) {
	input := "Ramen"
	result := reverse(input)
	defer delete(result)
	expected := "nemaR"
	testing.expect_value(t, result, expected)
}

@(test)
/// description = wide characters
test_wide_characters :: proc(t: ^testing.T) {
	input := "子猫"
	result := reverse(input)
	defer delete(result)
	expected := "猫子"
	testing.expect_value(t, result, expected)
}

@(test)
/// description = grapheme clusters
test_grapheme_clusters :: proc(t: ^testing.T) {
	input := "ผู้เขียนโปรแกรม"
	result := reverse(input)
	defer delete(result)
	expected := "มรกแรปโนยขีเผู้"
	testing.expect_value(t, result, expected)
}

// etc...
```

## Exercise: Word Count

The procedure `count_word` takes a string and returns a map of words and their count of occurrences in the string.

- A word consists of adjacent ASCII letters, numerals, and apostraphes.
- Words are case insensitive.

Example words:

- `Hello`
- `won't`
- `foo1`
- `123`

```go
package word_count

import "core:strings"

Word_Counts :: struct {
	_map: map[string]u32,  // _ prefix to avoid clash with reserved word
	
	// The string from which all the keys are sliced. 
	// We need this when we we free.
	keys_str: string,   
}

count_word :: proc(input: string) -> Word_Counts {
	
	// to_lower() returns a newly allocated string
	// We want a copy of the parameter string anyway because
	// we want the Word_Counts to own its string keys.
	// Also, we redeclare 'input' as a regular local variable
	// because we cannot assign to a parameter and because we 
	// cannot use the address operator on a parameter.
	// (This declaration effectively shadows the 
	// parameter for the rest of the scope.)
	input := strings.to_lower(input)
	
	word_counts := Word_Counts{ keys_str = input }

	// Array of the delimiter characters.
	// The [?] syntax makes the array size be inferred 
	// from the number of elements in the literal.
	delims := [?]string{" ", ",", ".", "\n"}

	// Most commonly, the in-clause of a for loop is an expression 
    // which returns a collection, enum, or string, and then iterates
    // over the elements, named values, or runes. In these cases, 
	// the in-clause expression is evaluated just once.

	// In this example, however, split_multi_iterate returns a 
    // string and boolean, and so the in-clause expression is evaluated
    // before each iteration.
    
    // Each iteration:
	// 1. The returned string is assigned to str 
	// 2. If the returned bool is false, the next iteration is 
    // skipped and the loop ends.
	for str in strings.split_multi_iterate(&input, delims[:]) {

		// trim() returns a string which is a slice of the original 
		word := strings.trim(str, "'\"()!&@$%^:")
	
		// ignore empty words
		if len(word) <= 0 {
			continue
		}

		// If the map has not yet been allocated, this 
		// operation first allocates the map.
		// If the key does not yet exist, it is created 
		// and initialized to 0 before the += operation.
		word_counts._map[word] += 1
	}

	return word_counts
}

delete_map_and_key :: proc(words: Word_Counts) {
	delete(words.keys_str)
	delete(words._map)
}
```

Here are some tests from this exercise:

```go
package word_count

import "core:testing"

@(test)
/// description = ignore punctuation
test_ignore_punctuation :: proc(t: ^testing.T) {
	input := "car: carpet as java: javascript!!&@$%^&"
	word_counts := count_word(input)
	defer delete_word_counts(word_counts)
	testing.expect_value(t, len(word_counts._map), 5)
	testing.expect_value(t, word_counts._map["car"], 1)
	testing.expect_value(t, word_counts._map["carpet"], 1)
	testing.expect_value(t, word_counts._map["as"], 1)
	testing.expect_value(t, word_counts._map["java"], 1)
	testing.expect_value(t, word_counts._map["javascript"], 1)
}

@(test)
/// description = include numbers
test_include_numbers :: proc(t: ^testing.T) {
	input := "testing, 1, 2 testing"
	word_counts := count_word(input)
	defer delete_word_counts(word_counts)
	testing.expect_value(t, len(word_counts._map), 3)
	testing.expect_value(t, word_counts._map["testing"], 2)
	testing.expect_value(t, word_counts._map["1"], 1)
	testing.expect_value(t, word_counts._map["2"], 1)
}

@(test)
/// description = with apostrophes
test_with_apostrophes :: proc(t: ^testing.T) {
	input := "'First: don't laugh. Then: don't cry. You're getting it.'"
	word_counts := count_word(input)
	defer delete_word_counts(word_counts)
	testing.expect_value(t, len(word_counts._map), 8)
	testing.expect_value(t, word_counts._map["first"], 1)
	testing.expect_value(t, word_counts._map["don't"], 2)
	testing.expect_value(t, word_counts._map["laugh"], 1)
	testing.expect_value(t, word_counts._map["then"], 1)
	testing.expect_value(t, word_counts._map["cry"], 1)
	testing.expect_value(t, word_counts._map["you're"], 1)
	testing.expect_value(t, word_counts._map["getting"], 1)
	testing.expect_value(t, word_counts._map["it"], 1)
}

// etc...
```

## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


## Exercise: 

The procedure

```go

```


Here are some tests from this exercise:

```go
```


