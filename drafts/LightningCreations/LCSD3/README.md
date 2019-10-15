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



