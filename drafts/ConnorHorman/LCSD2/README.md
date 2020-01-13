# Connor Horman LCS Draft 2 - PokemonSMS Public Specification

Draft Date - 2020-01-12
Copyright (C) 2018-2020 Connor Horman, See License and Copyright Notice in §1

## §1 Copyright Notice

The specification documents provided here are written and provided by the PokemonSMS Public Specification Project. 
The Project, all contained documentation and all associated "exemplar" code are (c) 2018-2020 Connor Horman.

Neither The PokemonSMS Public Specification Project,
 nor the PokemonSMS Core Libraries (collectively The "PokemonSMS Core and Specification")
 are developed by, associated with, or affiliated with Nintendo or Game Freak,
 or any other company associated with official Pokemon Games. 
 In addition, the PokemonSMS Core and Specification do not claim to be an official Pokemon Game, 
 or claim to provide the capability to create an official Pokemon Game.

Unless otherwise specified on an implementation defined basis,
 no implementions of PokemonSMS are official Pokemon Games of any kind and are to be considered a Parody
  of such official titles, and hold no affiliation with Nintendo or Game Freak.
   An implementation may be treated as such if and only if the copyright holder of the implementation
    has obtained prior written permission from Nintendo and/or Game Freak.

Any person or persons may use this document, in part or in whole, to do any of the following,
 for any purpose, including but not limited to, private purposes and commercial purposes:
* Provide Full or Partial Implementations of PokemonSMS or similar projects
* To develop custom Core Libraries, which are deployed on Implementations of PokemonSMS or similar projects
* To develop a derivative specification, based on, but not copied from, part or all of this specification
* To provide reference material for any form of written material
* To distribute verbaitum, or partial copies, keeping the appropriate copyright notice intact,
  and/or linking to the original document, except that copies may not be offered for a fee

Additionally, exemplar code contained within the specification are distributed under the MIT Free Software License.
 The Core Libraries are distributed under the GNU Lesser General Public License.

All of the documentation and exemplar code provided by this specification are provided on an AS IS basis, without any warranties of any kind, including the implied warranties of MERCHENTABILITY, FITNESS FOR A PARTICULAR PURPOSE, or NON INFRINGMENT.


## §2 References

Requirement Terms: The meaning of the terms "MUST", "MAY", "SHOULD", "SHALL", "MUST NOT", "SHOULD NOT", "SHALL NOT", 
 "REQUIRED", "RECOMMENDED", and "OPTIONAL" in all caps are the ones defined by [[RFC 2119]](https://tools.ietf.org/html/rfc2119).
 If a SHOULD, SHOULD NOT, or REQUIRED clause is not fufilled, the implementation must document this occurence.
 It is Implementation-defined if any MAY or OPTIONAL clause is fufilled. 

Binary IO: This document refers to, both normatively and informatively, [[LCS 4 - Binary IO]](https://LightningCreations.github.io/LCS/publications/LCS4).

ShadeNBT: This document refers to the ShadeNBT Specification, and the CryptoShade variant of the ShadeNBT Specification.
 See <https://chorman0773.github.io/BinarySpecifications/ShadeNBT>.
 
PkmCom: This document defines a Concrete Protocol for the PkmCom specification,
 documented [here](https://chorman0773.github.io/PkmCom-APL-Library).

## §3 Terms and Notation

### §3.1 PokemonSMS

Refers to the Complete work consisting of the PokemonSMS Public Specification,
 and the PokemonSMS Core.
 
### §3.2 PokemonSMS Core

PokemonSMS Core refers to a set of lua scripts, images, json language files, audio, and other data files for various purpose,
 which give specific meaning to parts of this Specification and of an Implementation.
 
#### §3.2.1 PokemonSMS Core Libraries

The PokemonSMS Core Libraries refer to the lua scripts contained within the PokemonSMS Core.

### §3.3 Implementation

An Implementation is some Program which operates according to this specification,
 and which loads and executes the PokemonSMS Core Libraries. 
 
### §3.4 Extension

An Extension is some code outside of the PokemonSMS Core Libraries or an Implementation, which is loaded in some
 implementation-defined way if supported.
 
### §3.5 Resource

A Resource is any part of the PokemonSMS Core that is not part of the PokemonSMS Core Libraries,
 or some non-code portion of an Extension.
 
### §3.6 Illegal Action

An illegal action is an action performed by the PokemonSMS Core Libraries or an Extension,
 which is forbidden by this specification. Code which performs an illegal action shall cease execution.
 It is implementation defined if the remainder of the program continues, or ceases execution entirely (Crash).

### §3.7 Undefined Behavior

Undefined Behavior is an error in the PokemonSMS Core Libraries or an Extension,
 for which this specification does not provide any meaningful requirements on.
There are no restrictions on the result of undefined behavior,
 except that if execution continues,
 the implementation must be left in a meaningful (but potentially erroneous state).

### §3.8 Implementation-defined Behavior

Implementation-defined behavior is behavior which is loosely constrained by this specification. 
An implementation is REQUIRED to document the result of all implementation-defined behavior,
 and must be consistent and deterministic in such results.
 
### §3.9 Unspecified Behavior

Unspecified behavior is similar to implementation-defined behavior, but the implementation is not required 
 to document the results, and may not be required to operate consistently. 
 
### §3.10 Conditionally-Supported Behavior

Conditionally-supported behavior, when supported by an implementation, shall operate according to its requirements,
 otherwise it is either an illegal action or undefined behavior.
 It is implementation-defined if conditionally-supported behavior is supported. 
 
### §3.11 Implementation Imposed Limit

An Implementation Imposed Limit is some upper limit where breaking such a limit is an illegal action.
 Anything which can be limited by the implementation will have some minimum limit.
 It is unspecified if any such limit is greater than this minimum.
 
### §3.12 This Specification

This Specification refers to the PokemonSMS Public Specification.

### §3.13 Range Notation

This Specification may refer to ranges of numbers using range notation, 
 which consists of a pair of numbers, separated by a comma inside of parenthesis and/or square brackets.
The first number indicates the lower bound, and the second number indicates the upper bound.
 If a bound has a square bracket (`[` or `]`), that bound includes the number, and is closed.
 If a bound has a parenthesis (`(` or `)`), that bound excludes that number, and is open.

If an open bound has no indicated number, then it is unbounded. 
A closed bound cannot be unbounded.

The range denoted as `(,)` is completely unbounded, and includes all numbers.

The context indicates whether or not the range contains Real Numbers, or only Integers.
 This can also be indicated explicitly by prefixing the range with `Z` (for integers), or `R` (for real numbers).

## §4 Object Naming

Various Objects in PokemonSMS are given a name, called an Object Name.
 This Object Name consists of 2 parts: a domain, and a path.
 The domain is an single identifier, consisting of lowercase latin letters without accents,
 latin digits, and the underscore character, starting with either a letter or the underscore character.
 The path is made up of one or more of such identifiers, called components, separated by a forward slash `/`.
 No domain or component of a path may consist of only underscores and digits.
 The domain and path are joined with a `:`. That is, the name of an object in the `foo` domain, called `bar`, 
 is `foo:bar`.

### §4.1 Registering Objects

When loaded, objects are registered with their name.
 The object names must be unique; it is illegal to register two objects with the same name.

Objects may be loaded in any of the following ways:
* By Defining in from a Core Library Descriptor script (Core Entries)
* By the Implementation, manually (Implementation-defined Entries)
* By an Extension (Injected Entries). If an Extension registers an entry in a domain, that extension becomes the owner of a domain. 
 If multiple extensions register entries in the same domain, the behavior is undefined. 

It is implementation defined if multiple entries with the same name of different types may be registered. 

### §4.2 Reserved Domains

Various domains are reserved by this specification for various reasons.
 Reserved Domains may not have Objects registered in the domain, except under specific rules.
Implementations MUST NOT violate these rules. 
The behavior of a Core Library Script or Extension that registers an object in a reserved domain
 in violation of these rules is undefined.

The reserved domains are owned by this specification or (in the case of the `impl` and `internal` domains) by the implementation.

#### §4.2.1 The `pokemon` Domain

The `pokemon` domain is owned by this specification for the PokemonSMS Core Library to register entries in.

