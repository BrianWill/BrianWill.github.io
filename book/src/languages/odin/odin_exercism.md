# Odin Intro - Code Examples

[Video](https://youtu.be/X2Fy2zcfRhM)

As a supplement to the [Odin Introduction](odin_data_types.md), here are some walkthroughs of very small Odin code examples.

- These code examples mainly come from the [Exercism project](https://exercism.org/tracks/odin) and are licensed under the [MIT License](https://opensource.org/license/mit)
- Language features that weren't covered in the prior material will be explained as they come up.
- For some examples, we include a few tests to demonstrate basics of the testing API.

## Setup

The Git repo is [here](https://github.com/exercism/odin/tree/main). The examples are found under `exercises/practice`.

The goal of each exercise is to pass the provided tests. To run the tests for exercise `foo`, run the command `odin test exercises/practice/foo` (assuming CWD is root of the repo).


## Exercise: Binary Search

The procedure `binary_search` returns the index within a list of a target value.

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

	// Defines Alphabet as a bit set type which has a bit
	// for every character in the range 'a' up to (and including) 'z'
	Alphabet :: bit_set['a' ..= 'z']

    // The zero value of a bit set is empty (all bits unset)
    expected: Alphabet
	found: Alphabet

    // An Alphabet literal with all bits set
	expected = Alphabet {
		'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l',
		'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
	}

	// Alternatively, we can get the full set 
    // by doing a logical not operation on the empty set
	expected = ~Alphabet{}  // same result as prior assignment

	UPPER_TO_LOWER_DIFF :: 'a' - 'A'

	// For every rune in the string
	// (a "rune" is an unsigned integer representing a Unicode code point)
	for r in str {
		if r >= 'a' && r <= 'z' {     // if lowercase...
			// Adding two bit sets returns their union,
			// so this is effectively setting the bit for 'r' in 'found'
			found += Alphabet{r}
		} else if r >= 'A' && r <= 'Z' {   // if uppercase...
			found += Alphabet{r + UPPER_TO_LOWER_DIFF}
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
	// represents what a user perceives as an individual character. 
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


## Exercise: Word Count

The procedure `count_words` takes a string and returns a map of words and their count of occurrences in the string.

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
	data: map[string]u32,
	
	// The string from which all the keys are sliced. 
	// We need this when we free.
	keys_str: string,   
}

count_words :: proc(input: string) -> Word_Counts {
	
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

	// A slice of the delimiter characters 
	// we'll use to split the string.
	// (A slice literal points to content of a 
	// statically-allocated array.)
	delims := []string{" ", ",", ".", "\n"}

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
	for str in strings.split_multi_iterate(&input, delims) {

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
		word_counts.data[word] += 1
	}

	return word_counts
}

delete_word_counts :: proc(words: Word_Counts) {
	delete(words.keys_str)
	delete(words.data)
}
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

	// We need a string builder to incrementally build 
	// the output string.
	bsbffer := strings.builder_make()
	defer strings.builder_destroy(&sb)

	// Each iteration evaluates match_iterator(), 
	// and the loop ends when it returns false
	// capture = the current match
	// _ = discard of the index
	for capture, _ in regex.match_iterator(&iter) {
		first_letter := capture.groups[0][0]
		strings.write_byte(&sb, first_letter)
	}

	// Note that to_string does not make a new allocation.
	// Instead, it just returns a slice of the 
	// builder's internal buffer.
	result := strings.to_string(sb)
 
	// Freeing the string newly allocated by to_upper
	// will be the caller's responsibility
	return strings.to_upper(result)
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
	
	lc_word := strings.to_lower(word)
	defer delete(lc_word)
	
	letters := letters_in_order(lc_word)
	defer delete(letters)
	
	anagrams := make([dynamic]string, 0, len(candidates))

	for candidate in candidates {
		lc_candidate := strings.to_lower(candidate)
		defer delete(lc_candidate)

		// exact matches do not count as anagrams
		if lc_word == lc_candidate { 
			continue 
		}

		candidate_letters := letters_in_order(lc_candidate)
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
	
	letters := utf8.string_to_runes(word)
	slice.sort(letters)
	
	return letters
}
```

Here are some tests from this exercise:

```go
package anagram

import "core:fmt"
import "core:testing"

// When #caller_location is used as a default parameter value, 
// the compiler inserts the number of the line of code 
// where the call was made. Effectively here, errors logged by 
// the expect_value call will use the line number of where
// expect_slices_match itself was called (rather than where
// expect_value is called inside expect_slices_match).
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

	// An error logged in this call will cite the 
	// line number of this call.
	expect_slices_match(t, result, []string{})
}

// etc...
```



## Exercise: Flatten Array

The `flatten` procedure returns the list of integers from an ordered hierarchy.

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
	result := make([dynamic]i32)

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
			// Append the actual ints to the output array.
			append(&result, v)
		case []Item:
			// Append the Items in reverse because
			// the stack is consumed last-in-first-out.
			#reverse for child in v {
				append(&stack, child)
			}
		}
	}

	return result[:]
}
```


## Exercise: Circular Buffer

The `Ring_Buffer` struct represents a ring buffer of ints.

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

// Pop the head element of the buffer.
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

// Add an element to end of the buffer.
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

// Add an element to end of the buffer.
// If full, clobber current head and make 
// the index after the new head
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

import "base:runtime"

List :: struct ($T: typeid) {
	head: ^Node(T),
	tail: ^Node(T),
	allocator: runtime.Allocator
}

Node :: struct ($T: typeid) {
	prev:  ^Node(T),
	next:  ^Node(T),
	value: T,
}

Error :: enum {
	None,
	Empty_List,
}

// Create a new list, optionally with initial elements.
// The .. on the last parameter indicates this proc can be 
// called with 0 or more T arguments. The arguments are passed 
// to the last parameter in a alice of T.
new_list :: proc($T: typeid, allocator := context.allocator, elements: ..T) -> List(T) {

	list := List(T){ allocator = allocator }
	for element, index in elements {
		node := new(Node(T), allocator)
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
		free(node, l.allocator)
	}
}

// Insert a value at the head of the list.
unshift :: proc(l: ^List($T), value: T) {

	node := new(Node(T), l.allocator)
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

	node := new(Node(T), l.allocator)
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

	if l.head == l.tail {
		l.head = nil
		l.tail = nil
	} else {
		l.head.next.prev = nil
		l.head = l.head.next
	}

	defer free(shifted_node, l.allocator)
	return shifted_node.value, .None
}

// Remove and return the value at the tail of the list.
pop :: proc(l: ^List($T)) -> (T, Error) {

	if l.head == nil {
		return 0, .Empty_List
	}

	poped_node := l.tail

	if l.head == l.tail {
		l.head = nil
		l.tail = nil
	} else {
		l.tail.prev.next = nil
		l.tail = l.tail.prev
	}

	defer free(poped_node, l.allocator)
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

// Remove the first element from the list which has the given value.
// List is unchanged if there is no matching value.
remove_first :: proc(l: ^List($T), value: T) {

	for node := l.head; node != nil; node = node.next {
		if node.value == value {
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
			free(node, l.allocator)
			return
		}
	}
}
```