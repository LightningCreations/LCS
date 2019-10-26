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
The following sections define the Base and Compound Syntax of an LCIR File, 

### §2.1 Base Syntax

The following rules are used throughout this specification to
describe basic parsing constructs. The US-ASCII coded character set
is defined by ANSI X3.4-1986.

```
OCTET          = <any 8-bit sequence of data>
CHAR           = <any valid character in the source charset, except for a Control Character, or the forward slash character "\">
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
LS             = *CR LF
NUL            = <US-ASCII Nul Character (0)>
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
If the source character set is not US-ASCII, then the appropriate corresponding character MUST be used in place of the US-ASCII equivalent used above. 

There is a different charset, called the execution charset, which is only used for generation of string literals. 
The execution charset shall contain all characters the Source Charset contains, and an Implementation-Defined Nul character. 


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
```

When Parsing Numerical Escapes, the character shall be interpreted according to the execution charset, except that 
the escape sequence `\0` shall always parse as the Nul Character. 

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
The term Volatile Access may also be used in such a say, and any Memory Operation that is also a Volatile Operation is a Volatile Access. 

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
* A Pointer Type, possibly restrict qualified (See §4.4 Pointer Types)
* The value part of an Atomic Type (See §4.6 Atomic Types)
* A vector type (See §4.3 Vector Types), where the component type of the vector is one of the above, or
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

## §4 Types

### §4.1 Scalar Types

Scalar Types are the fundamental building blocks of values in LCIR. 
 
