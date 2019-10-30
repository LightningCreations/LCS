# Lightning Creations LCSD 3 - LCIR Specification

Draft Date: 2019-10-14

## §1 Definitions

This document refers to several Internet Specifications released and maintained by the Internet Engineering Task Force (IETF). 

### §1.1 Augmented Backus-Naur Form

This document uses a modified version of Backus-Naur Form for defining the syntax of LCIR, called Augmented BNF, or ABNF.
This modified Backus-Naur Form is defined by [[RFC 5234]](https://tools.ietf.org/html/rfc5234). 

### §1.2 Requirement Levels

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
 "OPTIONAL", when in all caps, used in this document are to be interpreted as described in [[RFC 2119]](https://tools.ietf.org/html/rfc2119). 
 
### §1.3 Undefined Behavior

Undefined Behavior occurs in an LCIR program when a construct of LCIR is violated, but no result is prescribed by the LCIR Specification. 

In such an occurance, no limitations are posed on the result of that violation by the LCIR Specification. 
Cases where this occurs will be explicitly documented using the declarative wording "the behavior is undefined", or "the behavior of ... is undefined"
 
### §1.4 Unspecified Behavior

Unspecified Behavior is Behavior for which the implementation is given a choice on how to proceed.  
 
## §2 LCIR Syntax

An LCIR file is a collection of several symbols, type declarations, functions, and pragma declarations. 
The following sections define the Base and Compound Syntax of an LCIR File. 

In this document, programs are considered to parse in two phases:
 a grammar parse which simply attempts to build a tree of the file's syntax,
  and a semantic parse, which checks for errors in the constructs used. 

The Grammar parsing step is described using the Augmented Backus-Naur Form and sometimes in prose,
 whereas the semantic parsing steps are described in prose. 

### §2.1 Base Syntax

The following rules are used throughout this specification to
describe basic parsing constructs. The US-ASCII coded character set
is defined by ANSI X3.4-1986. 

```
OCTET          = <any 8-bit sequence of data>
CHAR           = <any valid character in the source charset, except for a Control Character, or the backward slash character "\">
UPALPHA        = <any US-ASCII uppercase letter "A".."Z">
LOALPHA        = <any US-ASCII lowercase letter "a".."z">
ALPHA          = UPALPHA / LOALPHA
DIGIT          = <any US-ASCII digit "0".."9">
CR             = <US-ASCII CR, carriage return (13)>
LF             = <US-ASCII LF, linefeed (10)>
SP             = <US-ASCII SP, space (32)>
HT             = <US-ASCII HT, horizontal-tab (9)>
<">            = <US-ASCII double-quote mark (34)>
WS             = SP / HT / WS WS
LS             = [CR] LF 
NUL            = <US-ASCII NUL Character (0)>
```

Whitespace in ABNF Declarations are not considered part of the grammar, and are purely to indicate separation. 
The presence of the rule WS or `<WS>` indicates that an unlimited number of whitespace characters may appear. 
Linefeed characters are considered distinct from Whitespace characters. 

The Carriage return character, when not immediately followed by a Linefeed character, may never appear in an LCIR file. 
It is optional that a Line Separator contain a Carriage Return character. 

### §2.2 Source Charset

LCIR Files may be encoded in any character set, known as the source character set. 
The source character set may be any character set, which contains the Unaccented Latin Letters ("A".."Z" and "a".."z"), 
The Latin Decimal Digits ("0".."9"), the Space Character, Horizontal Tab, and Linefeed Characters, 
and the Special Characters `"`, `'`, `$`, `_`, `-`, `+`, `*`, `>`, `&`, `\`, `[`, and `]`. 

The Octet definitions used in the Base Syntax refer to the US-ASCII Character Set. 
If the source character set is not US-ASCII,
 then the appropriate corresponding character MUST be used in place of the US-ASCII equivalent used above. 

There is a different charset, called the execution charset, which is only used for generation of string literals. 
The execution charset shall contain all characters the Source Charset contains, and an Implementation-Defined Null character. 


### §2.3 LCIR Compound Syntax

The following rules combine the above Basic Constructs to form standard constructs of the language

```
UPPERHEXLETTER = "A" / "B" / "C" / "D" / "E" / "F"
LOWERHEXLETTER = "a" / "b" / "c" / "d" / "e" / "f"
HEXDIGIT = DIGIT / UPPERHEXLETTER / LOWERHEXLETTER
DECINTEGER = DIGIT / DECINTEGER DECINTEGER
HEXINTEGER = HEXDIGIT / HEXINTEGER HEXINTEGER
INTEGER = ["-" / "+"] (DECINTEGER / "0x" HEXINTEGER)
FLOAT = (DECINTEGER ["." DECINTEGER] ["e" INTEGER] 
IDENT = (ALPHA / "$" / "_") *(ALPHA / DIGIT / "$" / "_").
ESCSEQ = "\" ("x" HEXINTEGER \ DECINTEGER \ "n" \ "t" \ "r" \ <"> \ "\" \ "e" \ "'")
STRING = <">*( CHAR \ ESCSEQ)<">
LITCHAR = "'"(CHAR \ ESCSEQ) "'"
POWEROFTWO = DECINTEGER
```


When Parsing Numerical Escapes, the character shall be interpreted according to the execution charset, except that 
the escape sequence `\0` shall always parse as the Nul Character. 

POWEROFTWO only parses numbers which are a power of two. 

## §3 Operations

Operations are the result of Instructions, or combining multiple operations as instructed to form Composite Operations. 

Operations can have any number of relations, or no relations, to other operations. These relations are:
1. *Observes the Result of*, and the conjugate *has its Result Observed by*
2. *follows*, and the conjugate *is followed by*
3. *is computed from* , and the conjugate *is used to compute*
4. *is an unrelated write to*
5. *negates*, and the conjugate *is negated by*
6. *apply to the same memory location*

All of the primary relations, except for 6, imply the first operation in the list is preceeded by the second operation. 
The conjuncates imply the opposite, and are rarely used within this document. 

### §3.1 Kinds and Types of Operations

Operations have various different Kinds, which instruct the Semantics of the Operation. The kinds are:
* Read Operation, which loads a value from some memory location
* Write Operation, which stores a value to some memory location
* Computation, which transforms a value in some way
* Branch Operation, which may break linear operation sequences allowing code execution to be redirected
* Call Operation, which temporarily moves code redirection to a subroutine, until a Corresponding Return Operation
* Return Operation, which returns from a Call Operation
* Fence Operation, which is used to semantically sequence operations in different threads
* Composite Operation, which is composed of multiple operations of the various kinds noted above

Additionally, Operations can have Types, which affect optimization, and relative sequencing. The types are:
* Normal Operation, which is none of the following
* Machine Specific Operation, which has Implementation-defined Semantics
* Atomic Operation, which are coherent across threads of execution
* Synchronous Operations, which can guard Atomic Operations, and other Synchronous Operations
* Volatile Operations, which has unpredictable results, and therefore cannot be optimized by the Implementation, or resequenced relative to other Volatile Operations. 

For brevity, the term Memory Operation may also be used to describe operations. This is not a descrete type, 
rather any Operation which is either a Read Operation, a Write Operation, or composed of at least one such operation, 
is a Memory Operation. 
The term Volatile Access may also be used in such a way, and any Memory Operation that is also a Volatile Operation is a Volatile Access. 

### §3.2 Follows 

Within a single thread of execution, operations can be said to follow other operations. 
If an Operation A *follows* B, this simply means that, strictly speaking, A occurs before B. 
A *follows* relation does not necessarily have an impact on sequencing, it is simply used to describe other relations. 

*follows* is strictly not reflexive between operations in the same thread of execution. 
That is, for any Operations A and B within the same thread of execution, either A *follows* B, or B *follows* A, but not both. 

An Operation A *follows* another Operation B, if and only if A and B occur within the same thread of execution, and at least one of the following cases is satisfied:
* A and B both result from adjacent instructions within the same Function, B is not a return Operation, B is either not a branch operation, or the Branch is not taken, and the instruction which causes A is strictly written after the instruction which causes B
* B is a Branch Operation, A results from the Instruction which is the target of B, and the Branch is Taken
* B is a Call Operation, and A results from the first instruction of the function called by B
* B is a return operation, and A follows the Call Operation which B returns to. 
* There exists some Operation C, such that C *follows* B, and A *follows* C.

### §3.3 Observes the Result of

An Operation can be said to Observe the Result of Some Other Operation. 
When this occurs, the Operation that has its result observed by the other MUST be allowed to complete execution before the other starts. 
This relation is strict, though purely semantic. 

An Operation A *Observes the Result of* another Operation B, if at least one of the following cases is satisified:
* Both A and B are Volatile Operations, and A *follows* B.
* A is a Write Operation, B is a Read Operation, and A *is Computed by* B.
* B is a Read Operation, A is a Write Operation, B and A *apply to the same memory region*, and A *follows* B. 
* There exists some operation C, such that C *Observes the result of* B, and A *Observes the result of* C

If An Operation A *Observes the Result of* another Operation B, then the following occurs:
* If both A and B are Read Operations that apply to the same memory region, and A is not a volatile access, then A will load the same value as B, unless there exists some Write Operation C, such that B does not *Observe the result of* C, and A *Observes the result of* C.
* If B is a Write Operation and A is a Read Operation that applies to the same memory region, and A is not a volatile access, then A will load the value stored by B, unless there exists some Write Operation C, such that C *Observes the result of* B, or C *is an Unreleated Write* to B, and A *Observes the result of * C.
* Any Observable Side effects of A will have occurred before B.

### §3.4 Unrelated Write To and Negates

An Operation can be said to be an Unrelated Write to some other operation. 
If this occurs, then those operations occur sequentially, and without a data race. 

An Operation A *is an Unrelated Write To* another operation B if
* A and B are both Write Operations
* A *follows* B, and
* A *does not Observe the Result of* B

In these cases, A can additionally be said to *Negate* B if:
* A *is an Unrelated Write To* B
* B is not a Volatile Access, and
* There exists no operation C, such that C does not *Observe the result of* A, and C *Observes the result of* B

If A *Negates* B, then B can be elided by the implementation, as the value written by B will never be read. 

### §3.5 Applies to the Same Memory Region as

A Memory Region is any region designated by a given memory address, which is consumed by the value of:
* A scalar Type (See §4.1 Scalar Types)
* A Pointer Type, possibly restrict qualified (See §4.3 Pointer Types)
* The value part of an Atomic Type (See §4.6 Atomic Types)
* A vector type (See §4.4 Vector Types), where the component type of the vector is one of the above, or
* A cv-qualified (See §4.2 Qualified Types) version of any of the above

Or any memory address that is being consumed by a structure type for padding, or the lock portion of an Atomic Type (See §4.6),
 when that memory address is examined as an array of u8. 

An Operation A *applies to* a memory region M if:
* A is a Memory Operation, which loads from or stores to the value at any M, or any address that falls within M
* A is a Composite Operation, an either Operation which composes A *applies to* M
 
Two Operations A and B *apply to the same memory region* if there is some memory region M, such that both A and B *applies to* M.
 
Unlike other Relations *applies to the same memory region as* is symmetric. 

### §3.6 Volatile Operations

Volatile Operations are operations which must be performed AS-IS. They cannot be optimized by the implementation. 
Volatile Operations are considered to have observable behavior. 

Volatile Accesses are Volatile Operations which are also Memory Operations. 

A Volatile Access A, which *follows* another Volatile Access B, *observes the result of* B.

### §3.7 Atomic Operations

Atomic Operations are operations which have coherency between threads. 

For any two Atomic Operations A and B, which *apply to the same memory region*:
 Either, A *observes the result of* B, or B *observes the result of* A (but not both). If this is not enforced by other rules, then it is unspecified which occurs. 

Atomic Operations may be said to acquire a lock. It is unspecified when locks are used to enforce the coherency of atomic operations, 
 however, a non-composite atomic operation which is a memory operation does not use a lock if:
* The type of the value affected by the operation is an (possibly Atomic) Scalar type, or a cv-qualified version thereof, and
* The Atomic version of the scalar type is defined to be Lock-free.


### §3.8 Fence Operations

Fence Operations are operations which provide coherency between threads, even when such coherency would otherwise not exist. 

There are 3 kinds of Fence Operations: Sequence Fence, Acquire Fence, and Release Fence.

Fence Operations are not Memory Operations. 

The properties of a Fence Operation depend on the Kind of Fence.

#### §3.8.1 Sequence Fence

For any Sequence Fence S, if there is a Memory Operation A such that S *follows* A, then S *observes the result of* A.
Additionally, if there is a Memory Operation B, such that B *follows* S, then B *observes the result of* S. 

If a Sequence Fence A *follows* another Sequence Fence B, then A *observes the result of* B.

If there are two Sequence Fences A and B, then either A *observes the result of* B, or B *observes the result of* A (but not both). 
Additionally, given such Sequence Fences A and B form a Composite Operation AB, which is a Sequence Fence. 

If there are three Sequence Fences A, B, and C; then the operations are ordered in some unspecified way,
 such that the 2nd *observes the result of* the 1st, and the 3rd *observes the result of* of the composite operation from the 1st and the 2nd. 
 
By these rules, Sequence Fences are said to be *sequentially-consistent* across threads. 

#### §3.8.2 Acquire and Release Fence

For any Acquire Fence A, if there is a memory operation B, such that B *follows* A, then B *observes the result of* A.

Additionally, if there is an Atomic Operation C, for which A *follows* C, which is either:
* An Read Operation, or
* A Write Operation that *is computed from* An Atomic Operation D that is a Read Operation. 

and C is the last such operation (that is, there is no Operation D that qualifies for which D *follows* C), 
 then A *observes the result of* C (C is said to *acquire* A).
 
For any Release Fence R, if there is a memory operation A, such that R *follows* A, then R *observes the result of* A.

Additionally, if there is an Atomic Operation C, which is Read Operation, C *follows* R, 
 and C is the last such operation, then C *observes the result of* R (C is said to *release* R). 
 
If there is an Acquire Fence FA, which is acquired by an Atomic Operation A, and a Release Fence FR,
 which is released by an Atomic Operation R, and A *observes the result of* R, then FA *observes the result of* FR. 

### §3.9 Is Computed From

An Operation A *is computed from* B, if:
* A is a Computation, B is a Read Operation and the value loaded by B is an operand in A
* Both A and B are computations, and the result of B is an operand in A
* A is a Write Operation, and B is a Read Operation, and the value loaded by B is the value stored by A
* A is a Write Operation, B is a Computation, and the result of B is the value stored by A

## §4 Types

### §4.1 Scalar Types

Scalar Types are the fundamental building blocks of values in LCIR. 
Scalar Types have indivisible values, there is no discrete value that can be read from the representation
 of a value of a scalar type, except that value itself. 
 
With the exception of u8, i8, and types which are defined to have the same size and alignment as u8,
 the alignment of scalar types is unspecified. 
 
A type uN is unsigned, with its signed version being iN. iN has the same size and alignment as uN,
 but stores a 2s compliment signed integer, rather than an unsigned integer. 
 
The scalar types are the following:
* u8, the smallest scalar type, which has a size of 1 byte, aligned to 1 byte, and can store any integer in the range `[0,255]`
* i8, which stores any integer in the range `[-128,127]` using 2s compliment
* u16, which has a size 2, and stores any integer in the range `[0,65535]`
* i16, which stores any integer in the range `[-32768,32767]`
* u32, which has size 4, and stores any integer in the range `[0,4294967295]`
* i32, which stores any integer in the range `[-2147483648,2147483647]`
* u64, which has size 8, and stores any integer in the range `[0,2^64-1]`
* i64, which stores any integer in the range `[-2^63,2^63-1]`
* The OPTIONAL types u128 and i128, which have size 16, and the ranges `[0,2^128-1]` and `[-2^127,2^127-1]`
* ul and il, which have an implementation-defined size N. If N is 8, 16, 32, 64, or 128, then ulong and ilong also has the same alignment as uN, and the ranges of uN and iN respectively.
* uptr and iptr, which have the same size and alignment as the pointer type, and implementation-defined ranges. 
* f32, which has size 4, and stores an IEEE 754 single-precision binary floating-point number
* f64, which has size 8, and stores an IEEE 754 double-precision binary floating-point number
* fl, which has an implementation-defined size and representation.
* OPTIONAL f16, which has size 2, and stores an IEEE 754 half-precision binary floating-point number
* OPTIONAL f128, which has size 16, and stores an IEEE 754 quad-precision binary floating-point number
* Additional Implementation-defined Extension Types (see §4.8 Extension Types)
* bool, which has the same size and alignment as u8, and is suitable for storing either 0 (for false), or 1 (for true)

Scalar types larger than 1 byte have an implementation-defined endianness.
 Big Endian stores the most significant byte in the lowest number address,
 Little Endian stores the most significant byte in the highest numbered address. 
 
Alignment of Scalars determines where implementations are easily able to load and store values of that type. 
A program that attempts to load a scalar value from, or store a scalar value to a memory region which is not aligned
 at least as strictly as the scalar type requires, the behavior is undefined. 
 
All types have alignment requirements which are a power of 2.
 The maximum valid alignment requirements for a type is implementation-defined. 

```
SCALARTYPE = "u8" \ "i8" \ "u16" \ "i16" \ "u32" \ "i32" \ "u64" \ "i64" \ "ul" \ "il" \ "uptr" \ "lptr" \ "f32"
                      \ "f64" \ "fl" \ "bool" \ "u128" \ "i128" \ "f16" \ "f128"
TYPE = SCALARTYPE
```

(Note: Even if the implementation does not define the types 16-bit integer types, half or quad precision floating-point type, 
 the grammar still parses them as types, the implementation makes the distinction in semantic parsing).

### §4.2 Qualified Types

All types can be additionally qualified as being const, volatile, atomic, an array or vector type, or having additional alignment requirements. 

These qualifiers take the form of an underscore character, followed by a capital letter designating the qualifier, 
 then (in the case of array, vector, or aligned types) possibly an integer number. 
 
Atomic and Vector types are described specially, in their own sections. 

Types qualified as const and/or volatile have the same size and alignment requirements as the unqualified type. 

Types with additional alignment requirements have those alignment requirements, rather than that of the unqualified type, but have the same size. 

The order of the `_C`, `_K`, and `_L` qualifiers is meaningless. 
The order of `_C` and `_K` relative to `_Q` `_A`, and `_V` qualifiers is meaningless. 

```
TYPEQUALIFIER = "_C" / "_K" / "_Q" / "_A" <DECINTEGER> / "_V" <DECINTEGER> / "_L" <POWEROFTWO>
TYPE = <TYPEQUALIFIER> <TYPE> / "_A" <TYPE>
```

#### §4.2.1 Const Qualifier

Types written as `_C<type>` are const qualified types.
 
 A memory region which stores a value of a const-qualified scalar type cannot be written too. 
 If a Write Operation *applies to* a memory region which stores a value of a const-qualified scalar type,
  the behavior is undefined. 
 
 Const-qualification applies to members of structure types (see §4.5) transitively. 
 The type of a member of a const-qualified structure type is const-qualified itself. 
 This also applies to the elements of an array or a vector. 
 
#### §4.2.2 Volatile Qualifier

Types written as `_V<type>` are volatile-qualified type. 

A memory region which stores a value of a volatile-qualified scalar type can only be accessed with a volatile operation. 
If a Memory Operation *applies to* a memory region which stores a value of a volatile-qualified scalar type,
 that Operation must be a Volatile Access, or the behavior is undefined. 
 
 Like const-qualification, volatile-qualification applies to members of a structure type transitively, 
  as well as to members of a volatile-qualified array or vector. 
  
#### §4.2.3 Alignment Qualifier

Types written as `_L<N><type>` are aligned types. An Aligned Type is aligned to *N* bytes.
 N MUST be at least the alignment requirement of *type*. 
 
#### §4.2.4 Array Types

Types written as `_A<N><type>` are array types.
 Array types are capable of storing *N* values of *type* called the element type. 
 The element type of an array MUST be a complete type. 
 
 Array types have the same alignment requirements as *type*. 
 
 *N* shall not be 0. 
 
Types written as `_A<type>` are array types with an unknown bound. 

An array type with an unknown bound is incomplete. 

 
### §4.3 Pointer Types

Types written as `*<type>` are pointer types. 
A Pointer type stores the address of a memory region, or part of a memory region (when examined as an array of u8). 
All pointer types have the same, implementation-defined, size and alignment requirements. 

Additionally, types written as `_R*<type>` are restrict pointer types. 
A restrict pointer type has the same size and alignment requirements as a pointer type. 
While there is a value of a restrict pointer type which points to a memory region,
 the program shall not access that memory region except through that pointer. 
 The behavior of a program that accesses such a memory region in violation of these rules is undefined. 
 
(Note - By these rules, a program can't access a memory region at all, while there are two or more restrict pointers which point to it).


```
TYPE = ["_R"] "*" <TYPE>
```

### §4.4 Vector Types

Types written as `_V<N><type>` are Vector Types. 

Vector types store *N* values of *type*, called the component type. 
`N` shall not be 0. 

Vector types may be used as a single value in computations.
 A vector of length N with scalar components occupies 1 memory region
  (Note - an array of size N with scalar elements occupies N memory regions). 
  
If *type* is not a scalar type, it is implementation defined if a vector type with that component type can be defined. 

The alignment requirements of a vector type are at least as strict as the alignment requirements for their component type. 


### §4.5 User Defined Types

The User can define additional types using structure declarations, union declarations,
 enumeration declarations, or type alias declarations.
 
The name of a User Defined Type is called a typename

```
TYPENAME = IDENT - ( SCALARTYPE / "void" )
```

Identifiers which start with an underscore (`_`) character, followed by any of `A`, `C`, `K`, `L`, `Q`, `T`, `V`, or `X`, 
MUST NOT parse as a typename. 

```
TYPE = TYPENAME
```

A typename which was not introduced by one of the above declarations shall not be used as a type. 

### §4.6 Atomic Types

A type written as `_Q<type>` is an atomic type. 
All memory operations on values of an atomic type are Atomic Operations. 


