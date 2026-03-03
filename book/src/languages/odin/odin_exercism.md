![Under construction](../../construction.gif)

# Odin - Walkthroughs of Small Examples

As a supplement to the [Odin Introduction](odin_data_types.md), here are some walkthroughs of very small Odin programs.

Language features that weren't covered in the prior material will be explained as they come up.

In some cases, we'll include some tests to demonstrate basics of the testing API.

> [NOTE!]
> These code examples mainly come from the [Exercism project](https://exercism.org/tracks/odin) and are licensed under the [MIT License](https://opensource.org/license/mit)

## Setup

The Git repo is [here](https://github.com/exercism/odin/tree/main). The examples are found under `exercises/practice`.

The goal of each exercise is to pass the provided tests. To run the tests for exercise `foo`, run the command `odin test exercises/practice/foo` (assuming CWD is root of the repo).


## Exercise: Binary Search

The procedure

```go
package binary_search

// Returns index of the target value in the list. 
// Returns false if target value is not in list.
// Assumes list is sorted.
// #optional_ok means the caller doesn't have to capture the returned boolean
// list = a slice of T
// Because T is used in the procedure with the < operator, T must be a numeric type.
find :: proc(list: []$T, target: T) -> (int, bool) #optional_ok {
	// indexes denoting start and end of search range
	start, end := 0, len(list)
	
	for start < end {
		// mid point between start and end
		middle := (start + end) / 2

		val := list[middle]
		if val == target {
			return middle, true
		} else if target < val {
			// if target is left of middle...
			end = middle
		} else {
			// if target is right of middle...
			start = middle + 1
		}
	}

	return 0, false
}
```

Here are some tests from this exercise:

```go
package binary_search

import "core:testing"

// The @(test) attribute marks the procedure as a test.
// A test procedure must have one and only one parameter of type ^testing.T
// The parameter name does not matter, but 't' is used by convention.
// Most core procedures of the testing package take the ^T param as their 
// first argument, and this is used to track the test state.

@(test)
/// description = finds a value in an array with one element
test_finds_a_value_in_an_array_with_one_element :: proc(t: ^testing.T) {
	input := []u32{6}
	result := find(input, 6)
	expected := 0
    // a test fails if the arguments to expect_value() are not equal
	testing.expect_value(t, result, expected)
}

@(test)
/// description = finds a value in the middle of an array
test_finds_a_value_in_the_middle_of_an_array :: proc(t: ^testing.T) {
	input := []u32{1, 3, 4, 6, 8, 9, 11}
	result := find(input, 6)
	expected := 3
	testing.expect_value(t, result, expected)
}

@(test)
/// description = finds a value at the beginning of an array
test_finds_a_value_at_the_beginning_of_an_array :: proc(t: ^testing.T) {
	input := []u32{1, 3, 4, 6, 8, 9, 11}
	result := find(input, 1)
	expected := 0
	testing.expect_value(t, result, expected)
}

@(test)
/// description = identifies that a value is not included in the array
test_identifies_that_a_value_is_not_included_in_the_array :: proc(t: ^testing.T) {
	input := []u32{1, 3, 4, 6, 8, 9, 11}
	_, found := find(input, 7)
	testing.expect_value(t, found, false)
}

// etc...
```


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


## Exercise: Acronym


The procedure `abbreviate` converts a phrase to its acronym.

Hyphens are treated as word separators (like whitespace), but all other punctuation is ignored.

Examples:

| Input                     | Output |
| ------------------------- | ------ |
| As Soon As Possible       | ASAP   |
| Liquid-crystal display    | LCD    |
| Automated Teller Machine | ATM   |


```go
package acronym

import "core:strings"
import "core:text/regex"

abbreviate :: proc(phrase: string) -> string {
	// A backtick string ignores \ escape sequences.
	pattern :: `[^ _-]+`
	
	// Since we know the regex pattern is correct, we ignore
	// the return error value.
	iter, _ := regex.create_iterator(phrase, pattern)
	defer regex.destroy_iterator(iter)

	// We need a string builder to incrementally build a string.
	buffer := strings.builder_make()
	defer strings.builder_destroy(&buffer)

	// Each iteration evaluates match_iterator(), 
	// and the loop ends when it returns false
	// capture = the current match
	// _ = discard of the index
	for capture, _ in regex.match_iterator(&iter) {
		first_letter := capture.groups[0][0]
		strings.write_byte(&buffer, first_letter)
	}

	// Note that to_string does not make a new allocation.
	// Instead, it just returns a slice of the builder's internal buffer.
	acronym := strings.to_string(buffer)
 
	// Freeing the string newly allocated by to_upper
	// will be the caller's responsibility
	return strings.to_upper(acronym)
}
```

Here are some tests from this exercise:

```go
package acronym

import "core:testing"

@(test)
/// description = basic
test_basic :: proc(t: ^testing.T) {

	expected := "PNG"
	input := "Portable Network Graphics"
	result := abbreviate(input)
	defer delete(result)

	testing.expect_value(t, result, expected)
}

@(test)
/// description = punctuation
test_punctuation :: proc(t: ^testing.T) {

	expected := "FIFO"
	input := "First In, First Out"
	result := abbreviate(input)
	defer delete(result)

	testing.expect_value(t, result, expected)
}

@(test)
/// description = all caps word
test_all_caps_word :: proc(t: ^testing.T) {

	expected := "GIMP"
	input := "GNU Image Manipulation Program"
	result := abbreviate(input)
	defer delete(result)

	testing.expect_value(t, result, expected)
}

@(test)
/// description = punctuation without whitespace
test_punctuation_without_whitespace :: proc(t: ^testing.T) {

	expected := "CMOS"
	input := "Complementary metal-oxide semiconductor"
	result := abbreviate(input)
	defer delete(result)

	testing.expect_value(t, result, expected)
}
```

## Exercise: Anagram

The `find_anagrams` procedure takes a target word strings and a slice of candidate word strings. It returns a slice of the candidate words that are anagrams of the target.

For example, given the target `"stone"` and the candidate words `"stone"`, `"tones"`, `"banana"`, `"tons"`, `"notes"`, and `"Seton"`, the returned anagram words are `"tones"`, `"notes"`, and `"Seton"`.

```go
package anagram

import "core:slice"
import "core:strings"
import "core:unicode/utf8"

// Takes a target word and a list of candidate anagram words.
// Returns allocated slice of strings containing the candidate words
// that test postiive as anagrams of the target.
find_anagrams :: proc(word: string, candidates: []string) -> []string {
	anagrams := make([dynamic]string, 0, len(candidates))
	
	letters := letters_in_order(word)
	defer delete(letters)
	
	lc_word := strings.to_lower(word)
	defer delete(lc_word)

	for candidate in candidates {
		lc_candidate := strings.to_lower(candidate)
		defer delete(lc_candidate)

		// ignore exact matches
		if lc_word == lc_candidate { 
			continue 
		}

		candidate_letters := letters_in_order(candidate)
		defer delete(candidate_letters)
		
		if slice.equal(letters, candidate_letters) {
			// if sorted letters of the target and candidate are equal...
			append(&anagrams, candidate)
		}
	}

	return anagrams[:]
}

// Returns allocated slice of runes containing 
// the letters of the word in sorted order.
letters_in_order :: proc(word: string) -> []rune {
	lc_word := strings.to_lower(word)
	defer delete(lc_word)

	letters := utf8.string_to_runes(lc_word)
	slice.sort(letters)
	
	return letters
}
```

Here are some tests from this exercise:

```go
package anagram

import "core:fmt"
import "core:testing"

expect_slices_match :: proc(t: ^testing.T, actual, expected: []string, loc := #caller_location) {
	result := fmt.aprintf("%s", actual)
	exp_str := fmt.aprintf("%s", expected)
	defer {
		delete(result)
		delete(exp_str)
	}
	testing.expect_value(t, result, exp_str, loc = loc)
}

@(test)
/// description = no matches
test_no_matches :: proc(t: ^testing.T) {	
	result := find_anagrams("diaper", 
        []string{"hello", "world", "zombies", "pants"})
	defer delete(result)

	expect_slices_match(t, result, []string{})
}

@(test)
/// description = detects two anagrams
test_detects_two_anagrams :: proc(t: ^testing.T) {
	result := find_anagrams("solemn", 
        []string{"lemons", "cherry", "melons"})
	defer delete(result)

    expect_slices_match(t, result, []string{"lemons", "melons"})
}

@(test)
/// description = detects anagram
test_detects_anagram :: proc(t: ^testing.T) {
	result := find_anagrams("listen", 
        []string{"enlists", "google", "inlets", "banana"})
	defer delete(result)

	expect_slices_match(t, result, []string{"inlets"})
}
```


## Exercise: Clock

This exercise creates a struct `Clock` to represent the time of day and implements some basic operations for procedures.

```go
package clock

import "core:fmt"

// Implement this struct
Clock :: struct {
	mins: int,
}

// Compile time constant (by convention, has an all uppercase name)
MINS_PER_DAY :: 24 * 60

create_clock :: proc(hour, minute: int) -> Clock {
	return Clock{normalize(60 * hour + minute)}
}

// The @(private) attribute makes this 
// procedure private to the package.
@(private)
normalize :: proc(minutes: int) -> int {
    // minute values greater than the total 
    // minutes in a day are assumed to have wrapped
	return minutes %% MINS_PER_DAY
}

to_string :: proc(clock: Clock) -> string {
	h, m := clock.mins / 60, clock.mins % 60

    // the prefix 'a' indicates this print proc allocates
	return fmt.aprintf("%02d:%02d", h, m)
}

add :: proc(clock: ^Clock, minutes: int) {
	clock.mins = normalize(clock.mins + minutes)
}

subtract :: proc(clock: ^Clock, minutes: int) {
	add(clock, -minutes)
}

equals :: proc(clock1, clock2: Clock) -> bool {
	return clock1.mins == clock2.mins
}
```


Here are some tests from this exercise:

```go
package clock

import "core:testing"

@(test)
/// description = Create a new clock with an initial time -> on the hour
test_create_a_new_clock_with_an_initial_time__on_the_hour :: proc(t: ^testing.T) {
	clock := create_clock(hour = 8, minute = 0)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "08:00")
}

@(test)
/// description = Create a new clock with an initial time -> past the hour
test_create_a_new_clock_with_an_initial_time__past_the_hour :: proc(t: ^testing.T) {
	clock := create_clock(hour = 11, minute = 9)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "11:09")
}

@(test)
/// description = Create a new clock with an initial time -> midnight is zero hours
test_create_a_new_clock_with_an_initial_time__midnight_is_zero_hours :: proc(t: ^testing.T) {
	clock := create_clock(hour = 24, minute = 0)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "00:00")
}

@(test)
/// description = Create a new clock with an initial time -> hour rolls over
test_create_a_new_clock_with_an_initial_time__hour_rolls_over :: proc(t: ^testing.T) {
	clock := create_clock(hour = 25, minute = 0)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "01:00")
}

@(test)
/// description = Create a new clock with an initial time -> hour rolls over continuously
test_create_a_new_clock_with_an_initial_time__hour_rolls_over_continuously :: proc(t: ^testing.T) {
	clock := create_clock(hour = 100, minute = 0)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "04:00")
}

@(test)
/// description = Create a new clock with an initial time -> sixty minutes is next hour
test_create_a_new_clock_with_an_initial_time__sixty_minutes_is_next_hour :: proc(t: ^testing.T) {
	clock := create_clock(hour = 1, minute = 60)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "02:00")
}

@(test)
/// description = Create a new clock with an initial time -> minutes roll over
test_create_a_new_clock_with_an_initial_time__minutes_roll_over :: proc(t: ^testing.T) {
	clock := create_clock(hour = 0, minute = 160)
	result := to_string(clock)
	defer delete(result)

	testing.expect_value(t, result, "02:40")
}

// etc...
```


## Exercise: Flatten Array

The procedure

```go
package flatten_array

// This union is recursively defined as having 
// variants i32 and slices of itself.
// An Item effectively represents an ordered tree of i32 values.
Item :: union {
	i32,
	[]Item,   
}

// Returns the flattened list of all i32s within an Item. 
flatten :: proc(input: Item) -> []i32 {
	final := make([dynamic]i32)

	// Instead of calling flatten recursively, we use a stack 
	// to track the nested Items as we encounter them.
	// (While a proc recrusion solution might be a bit 
	// more familiar and simpler, creating our own stack
	// instead is often more effecient.)
	stack := make([dynamic]Item)
	defer delete(stack)

	// We start the loop with just the original item in the stack.
	append(&stack, input)
	
	for len(stack) > 0 {
		// The builtin proc pop removes the last 
		// element of a dynamic array.
		item := pop(&stack)
		switch v in item {
		case i32:
			// Append the actual ints to the final output array.
			append(&final, v)
		case []Item:
			// Append the Items in reverse because
			// the stack is consumed last-in-first-out.
			#reverse for child in v {
				append(&stack, child)
			}
		}
	}

	return final[:]
}
```


Here are some tests from this exercise:

```go
package flatten_array

import "core:slice"
import "core:testing"

@(test)
/// description = empty
test_empty :: proc(t: ^testing.T) {
	result := flatten([]Item{})
	defer delete(result)
	testing.expect(t, slice.equal(result, []i32{}))
}

@(test)
/// description = flattens a nested array
test_flattens_a_nested_array :: proc(t: ^testing.T) {
	result := flatten([]Item{[]Item{[]Item{}}})
	defer delete(result)
	expected := []i32{}
    // The expectf() proc logs a custom message if the check fails.
	testing.expectf(t, slice.equal(result, expected), "expected %v got %v", expected, result)
}
```


## Exercise: Circular Buffer

The procedure

```go
package circular_buffer

import "base:runtime"

Ring_Buffer :: struct {
	elements: []int,

	// Number of slots that are currently occupied
	size:     int,   
	
	// Index of the first element
	head:    int,   

	// Generally, an allocated data structure should 
	// reference its allocator so it can be freed or reallocated.
	allocator: runtime.Allocator,
}

Ring_Error :: enum {
	None,
	BufferEmpty,
	BufferFull,
}

new_buffer :: proc(capacity: int, 
		// The allocator parameter has a default value 
		// and inferred type (runtime.Allocator)
		allocator := context.allocator) -> Ring_Buffer {
	
	return Ring_Buffer{
		elements = make([]int, capacity, allocator),
		allocator = allocator,
	}
}

destroy_buffer :: proc(b: ^Ring_Buffer) {
	delete(b.elements, b.allocator)
	b.size = 0
	b.head = 0
}

clear :: proc(b: ^Ring_Buffer) {
	b.head = 0
	b.size = 0
}

// Pop the head element of the buffer
// Return .BufferEmpty if buffer is empty
read :: proc(b: ^Ring_Buffer) -> (int, Ring_Error) {
	if b.size == 0 {
		return 0, .BufferEmpty
	}

	value := b.elements[b.head]

	// advance head (wrap if necessary)
	b.head = (b.head + 1) % len(b.elements)

	b.size -= 1
	return value, .None
}

// Add an element to end of the buffer
// Return .BufferFull if buffer is full
write :: proc(b: ^Ring_Buffer, value: int) -> Ring_Error {
	if b.size == len(b.elements) {
		return .BufferFull
	}

	index := (b.head + b.size) % len(b.elements)
	b.elements[index] = value
	b.size += 1
	return .None
}

// Add an element to end of the buffer 
// If full, clobber current head and make second element the head.
overwrite :: proc(b: ^Ring_Buffer, value: int) {
	err := write(b, value)

	if err == .BufferFull {
		// if buffer was full...

		// overwrite oldest value and move head
		b.elements[b.head] = value

		// advance head (wrap if necessary)
		b.head = (b.head + 1) % len(b.elements)
	}
}
```

## Exercise: Linked List

This example defines a generic doubly-linked list type.

```go
package linked_list

List :: struct ($T: typeid) {
	head: ^Node(T),
	tail: ^Node(T),
}

Node :: struct ($T: typeid) {
	prev:  ^Node(T),
	next:  ^Node(T),
	value: T,
}

Error :: enum {
	None,
	Empty_List,
	Unimplemented,
}

// Create a new list, optionally with initial elements.
// The .. on the last parameter indicates this proc can be 
// called with 0 or more T arguments. The arguments are passed 
// to the last parameter in a alice of T.
new_list :: proc($T: typeid, elements: ..T) -> List(T) {

	list := List(T){}
	for element, index in elements {
		node := new(Node(T))
		node.value = element
		node.prev = list.tail
		if index == 0 {
			list.head = node
		} else {
			list.tail.next = node
		}
		list.tail = node
	}
	return list
}

// Deallocate the list
destroy_list :: proc(l: ^List($T)) {

	for node := l.head; node != nil; node = node.next {
		free(node)
	}
}

// Insert a value at the head of the list.
unshift :: proc(l: ^List($T), value: T) {

	node := new(Node(T))
	node.value = value
	node.next = l.head
	if l.head != nil {
		l.head.prev = node
	}
	l.head = node
	if l.tail == nil {
		l.tail = node
	}
}

// Add a value to the tail of the list
push :: proc(l: ^List($T), value: T) {

	node := new(Node(T))
	node.value = value
	node.prev = l.tail
	if l.tail != nil {
		l.tail.next = node
	}
	l.tail = node
	if l.head == nil {
		l.head = node
	}
}

// Remove and return the value at the head of the list.
shift :: proc(l: ^List($T)) -> (T, Error) {

	if l.head == nil {
		return 0, .Empty_List
	}

	shifted_node := l.head
	defer free(shifted_node)

	if l.head == l.tail {
		l.head = nil
		l.tail = nil
	} else {
		l.head.next.prev = nil
		l.head = l.head.next
	}

	return shifted_node.value, .None
}

// Remove and return the value at the tail of the list.
pop :: proc(l: ^List($T)) -> (T, Error) {

	if l.head == nil {
		return 0, .Empty_List
	}

	poped_node := l.tail
	defer free(poped_node)

	if l.head == l.tail {
		l.head = nil
		l.tail = nil
	} else {
		l.tail.prev.next = nil
		l.tail = l.tail.prev
	}

	return poped_node.value, .None
}

// Reverse the elements in the list (in-place).
reverse :: proc(l: ^List($T)) {

	// Start from the tail and move up to the head,
	// while swapping the nodes 'prev' and 'next' pointers.
	next_node := l.tail
	for next_node != nil {
		n := next_node
		// Increment the node before we modify it so we don't mess
		// up the loop.
		next_node = next_node.prev
		n.prev, n.next = n.next, n.prev
	}
	l.head, l.tail = l.tail, l.head
}

// Returns the number of elements in the list
count :: proc(l: List($T)) -> int {

	n := 0
	for node := l.head; node != nil; node = node.next {
		n += 1
	}
	return n
}

// Remove and free the first element from the list which has the given value.
// List is unchanged if there is no matching value.
remove_first :: proc(l: ^List($T), value: T) {

	for node := l.head; node != nil; node = node.next {
		if node.value == value {
			defer free(node)
			if node.prev != nil {
				node.prev.next = node.next
			} else {
				l.head = node.next
			}
			if node.next != nil {
				node.next.prev = node.prev
			} else {
				l.tail = node.prev
			}
			return
		}
	}
}
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


