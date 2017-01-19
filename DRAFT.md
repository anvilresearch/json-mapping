# JSON Mapping

## Abstract

JSON Mapping defines a declarative method for specifying transformations of JSON data structures using JSON Pointers, JSON Schemas, and JSON References. JSON Mappings are useful where a service may wish to present multiple JSON representations of its data and for clients that consume similar data from multiple sources and wish to normalize heterogeneous input structures for processing or storage. 

## Introduction

JSON Mapping operations can rename properties, select subsets of an object, destructure and restructure complex nested objects, coerce types using JSON Schema Type Coercion Rules [to be drafted], assign default values using the JSON Schema “default” keyword, and incorporate referenced values declared as JSON References.

A JSON Mapping object is a JSON object or array in which keys (in the case of an object) and values contain JSON Pointer strings representing paths to read from a source object and write to a target object. The value of a JSON Mapping can be a JSON Pointer string or an object with properties that declare the method for deriving an appropriate value to map. 

## Mapping Documents

JSON Mappings can be expressed in JSON as objects or arrays. In JSON Mapping objects, keys are strings representing JSON Pointers and values can be either JSON Pointer strings or descriptive objects containing one or more properties representing specified keywords, "pointer", "type", "default", or "$ref". String and object values can be used in the same mapping. 

	{
		"/a": "/a/0",
		"/b": {
			"pointer": "/b",
			"type: "string",
			"default": "I am a value"
			// additional or alternative keywords
		}, 
		"/c": {
			"$ref": "https://example.com/data.json"
		}
	}

In a mapping such that all keys-value pairs in the mapping have equality between key and value, the mapping can be expressed more succinctly as an array of JSON Pointers. The following two mappings are equivalent:

	{
		"/a": "/a", 
		"/c/d": "/c/d", 
		"/e/2": "/e/2"
	}

	[ "/a", "/c/d", "/e/2" ]

## Keywords

Implementations MUST ignore members of the mapping that are not JSON Pointers or defined as valid property names in this document.

### Property keywords

$id
$uri
$schemas

### Descriptor properties

pointer

	The `pointer` property MUST be present for any descriptor that does not contain the `$ref` property. It 
	MUST NOT be present when the `$ref`. When present it MUST be a valid JSON Pointer string.

type

	The `type` property describes the expected type to be assigned. Implementations MUST attempt to coerce
	types written to target objects according to JSON Schema Type Coercion Rules. When types cannot be coerced
	the original value MUST be assigned to the target and error conditions SHOULD be ignored. Use of this property
	is OPTIONAL.

default

	The `default` property specifies the value to be assigned to a target object in the event a corresponding value is 
	not present in the source object at the JSON Pointer defined in the descriptor. Implementations MUST respect this 
	property. In the event a `type` is defined in the same descriptor, default values MAY be of a different type. 
	Implementations MUST NOT attempt to coerce types. Use of this property is OPTIONAL. 

$ref

## Mapping Operations

There are two primary operations that can be performed using a JSON Mapping: “map” and “project”. A third operation, "select", is a specialization of "map". Operations read from a source document and write to a target document, according to a mapping. The target can represent an empty object, an empty array, an object with existing properties, or an array with existing elements. In case the source and target object are equivalent, operations MUST leave existing properties intact while performing transformations described by the mapping. Operations MUST ignore error conditions that arise during JSON Pointer-based read and write operations.

Keys in a mapping object that are not JSON Pointers MUST be ignored by operations over the mapping.

### Map

The map operation iterates over key-value pairs in a mapping and reads values from a source object using the “right side” of a mapping (its value) and assigns those values to a target object using the corresponding “left side” of the mapping (its key). 

#### Mapping

	{
		“/a”: “/b/0”,
		“/b”: “/b/1/bar”,
		“/c/d”: “/c/def”
	}

#### Source
		
	{
		b: [ 
			{ “foo”: true }, 
			{ “bar”: false } 
		],
		c: { 
			“def”: 1337 
			},
		q: 3
	}

#### Target
	
	{
		a: { 
			“foo”: true 
		},
		b: false,
		c: { 
			d: 1337 
		}
	}

### Project

The "project" operation reverses "map". It iterates over key-value pairs in a mapping and reads values from a source object using the “left side” of a mapping (its key) and assigns those values to a target object using the corresponding “right side” of the mapping (its value). 

#### Mapping
	
	{
		“/a”: “/b/0”,
		“/b”: “/b/1/bar”,
		“/c/d”: “/c/def”
	}
		
#### Source
		
	{
		a: { 
			“foo”: true 
		},
		b: false,
		c: { 
			d: 1337 
		}
	}

#### Target

	{
		b: [ 
			{ “foo”: true }, 
			{ “bar”: false } 
		],
		c: { “def”: 1337 }
	}	


### Select

A mapping is a “selection” if all of its key-value pairs have equal keys and values. Such mappings can be abbreviated as an array of pointers. The “select” operation is equivalent to the map operation, except that it takes an array of JSON Pointers instead of a mapping object.

#### Mapping
	
	{
		“/a”: “/a”,
		“/d/f”: “/d/f”,
	}

#### Equivalent Selection

	[ “/a”, “/d/f” ]

#### Source

	{
		"a": 1,
		"b": 2,
		"c": 3,
		"d": {
			"e": 4,
			"f": 5
		}
	}

#### Target

	{
		"a": 1,
		"d": {
			"f": 5
		}
	}

## Property Inclusion

Default behavior for operations is to only write values to the source object from the target object which are explicitly specified by the mapping. This default behavior can be thought of as "selection" or "filtering".

- filtering (default behavior)
- allowing additional properties (like a "deep copy" with partial transformations)
- difference between additional properties and mapping from source to itself as the target


## Type Coercion

The "type" keyword indicates the expected type of value to be assigned to a given property on a target. Mappings that include this keyword MUST attempt to coerce corresponding values from the source object to the expected type according to JSON Schema Type Coercion Rules [to be drafted]. In the event a value is present and cannot be coerced to the expected type, it should be assigned as is without creating an error condition.

## Default Values

Mappings MAY use the "default" keyword to indicate a value that should be assigned for a given pointer in the absence of a corresponding value on the source object. Implementations MUST respect default value declarations.

## JSON References

Data external to the source document can be incorporated into the result of a mapping operation by resolving a JSON Reference on the right side of a mapping. 

### Canonical References

### Inline References

## Patterns

Mappings can use Regular Expressions in a variety of ways. 

### Pointer Patterns

- include property from source if pointer matches a pattern?
- limited to source side of mapping? 

### Value Patterns

- include a value if matches a regular expression
- use a regular expression match from the read value as the write value

## Expressions

### Contexts

- pointer expressions
- reference expressions
- value expressions

### Operators

- math operators
- math functions?
- string operators
- string templates
- date format transforms

## Mapping metadata

Additional non-mapping properties can be included in a mapping document.

Any property name that is not a JSON Pointer is ignored.

### Mapping Identifiers


## Mapping Validation

JSON Mappings implicitly represent a pair of schemas. Mappings should therefore hypothetically be validatable against a pair of JSON Schemas for the respective source and target objects.

## Security Considerations

Because JSON Mappings are expressed as JSON, they can be transmitted between parties. Users should take care not to disclose sensitive information about data contained on any system.

## References

### Normative References

* [IETF] JavaScript Object Notation
* [IETF] JSON Schema
* [IETF] JSON Pointer
* [IETF] JSON Reference
* [TBD] JSON Schema Type Coercion Rules
* [TBD] JSON Schema-based Object Initialization

### Informative References

## Appendix

### A.1 – Rename properties

#### Mapping
	
	{ “/changed”: “/original” }

#### Source
	
	{ "original": "value" }

#### Target
	
	{ "changed": "value" }

### A.2 – Select subsets of an object

#### Mapping
	
	{
		"/a": "/a",
		"/c/e": "/c/e",
		"/g": "/g"
	}

#### Selection

Since each key-value pair has equality of keys and values, we can express the mapping as a selection. 

	["/a", "/c/e", "/g"]

#### Source
	
	{
		"a": 1,
		"b": 2,
		"c": {
			"d": 3,
			"e": 4,
			"f": 5
		},
		"g": 6,
		"h": 7
	}

#### Target
	
	{
		"a": 1,
		"c": {
			"e": 4
		},
		"g": 6
	}


### A.3 Destructuring

### A.4 – Restructure complex nested objects

#### Mapping
	
	{ 
		“/a/b/c/d”: “/c/d”, 
		“/a/b/c/e”: “/g”,
		"/f": "/c/f",
		"/g/h/0": "/c/e" 
	}

#### Source
	
	{
		"a": 1,
		"b": 2,
		"c": {
			"d": 3,
			"e": 4,
			"f": 5
		},
		"g": 6,
		"h": 7
	}

#### Target
	
		{
		"a": {
			"b": {
				"c": {
					"d": 3,
					"e": 6
				}
			}
		},
		"f": 5,
		"g": {
			"h": [4]
		}
	}

### A.5 – Coerce types 

#### Mapping
	
	{ 
		“/coerced”: {
			"pointer": “/original”,
			"type": "number"
		}
	}

#### Source
	
	{ "original": "1337" }

#### Target
	
	{ "coerced": 1337 }

### A.6 – Assign default values 

#### Mapping
	
	{
		“/a”: “/g”,
		“/d/f”: {
			"pointer": "/d/f",
			"default": 6
		},
	}

#### Source

	{
		"a": 1,
		"b": 2,
		"c": 3,
		"d": {
			"e": 4,
			"f": 5
		}
	}

#### Target

	{
		"d": {
			"f": 6
		}
	}

### A.7 – Resolve JSON References




### Scratch

	{
	
		“/foo”: “/foo + /bar”
	
	}

	{
		 “/a/b”: “/c”,
		“/d/e”: {
			“pointer”: “/f/g/3”,
			“type”: “boolean”, 
			“default”: false
		},
		“h”: { “$ref”: “https://..." },	
		“I”:  {
			pointer: “/f/%r/%r”,
			patterns: [
				/^foo.*/I,
				/^bar.*/
			]
			match: { …} // JSON Schema
		}
	}


	query = {
		select: [“/a”, “/b/c”],
		match: {
			// json schema
		}
	
	}


	{
		‘/a/c’, ‘/e’,
		‘/a/b’: {
			pointer: ‘/c/d/3’,
			schema: { type: ‘boolean’, default: false }
		},
		‘/a/f/0’: {
			$ref: ‘https://...’
		}
	}





