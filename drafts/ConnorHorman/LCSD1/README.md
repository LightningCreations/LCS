# Connor Horman LCSD1 - Binary IO Format

Draft Date: 2019-11-10

This document is released under the terms of the GNU Free Documentation License. See §4 "GNU Free Documentation License" for details. 

## §1 Purpose of this document

This document specifies the method in which Binary Files should be encoded,
 as referenced by other LCS Documents. 
This document does not specify the method in which the files are stored, transported, or interpreted, only the encoding. 
Additionally, this document does not specify the mechanism for which the encoding is produced or consumed. 

### §1.1 Further Encoding

This document defines the binary representation of files using this encoding. 
 However, this need not correspond precisely to the physical representation on disk or in transport. 
 
 In particular, files may be further encoded in a way that preserves the representation,
  but alters it in some reversible manner, such as encryption, lossless compression, or textual encoding (such as base64). 

It SHOULD be possible to recover the content of the file from the further encoding, independent of the implementation which created it.
It is not necessary that this recovery be possible without details which are not known or part of the file (for example, encryption keys).


## §2 Terms

### §2.1 Requirement Terms

The key words and phrases "MUST", "SHOULD", "MAY", "MUST NOT", "SHOULD NOT", "REQUIRED", "RECOMMENDED", 
 and "OPTIONAL", in all caps, shall be interpreted according to [[RFC 2119]](https://tools.ietf.org/html/rfc2119). 
 
### §2.2 Reject a File

This, or a specification which references this specification,
 may require or allow an implementation "reject" a file. 

If an implementation rejects a file, that file MUST NOT be successfully read,
 and its contents discarded by the implementation. 
 Other specifications may impose additional requirements on implementations that reject a file (for example, to require a diagnostic message be generated and reported). 
 
An Implementation MUST NOT produce a file it would reject,
 either by a requirement of this specification or another specification,
  or an optional term which the implementation chooses to follow. 
  
For brevity, the requirement Terms "MUST" and "SHOULD", when describing the contents of a file, means that
 an implementation MUST or MAY, respectively, reject a file that violates that term. 

### §2.3 Encoding notation

This specification uses a sequence of hexadecimal octets enclosed in `[]` to indicate the byte encoding
 of various parts of a file (primarily in examples). 
 This defines the binary representation of the content of the file,
  in the precise order defined in the notation. 
 
 The octets contained in this notation are always, unambiguously, 2 hexadecimal digits. 
 They MUST NOT be treated as decimal, octal, binary, base32, or base64 digits, even when the digit,
  or even the entire octet may be valid to interpret as either. 
  
Uppercase hexadecimal digits will be used within this document.
 However other specifications that use this notation may use lowercase hexadecimal digits. 
 There is no difference between the two (IE. the byte representations `[ab]`, `[AB]`, `[aB]`, and `[Ab]` are all the same). 

### §2.4 Byte

A byte is a single unit within the binary representation of the file, constructed of 8-bits. 
A byte can store any byte representation with exactly octet (such as `[00]`, `[ff]`)

### §2.5 Byte Order Mode

The content of files produced under this specification can be described as being in one of two "Byte Order Modes",
 "Little Endian", and "Big Endian". 
 
The Byte Order Mode refers to the order which multibyte scalar types are written. 
In Little Endian Byte Order Mode, the least significant byte is written at the lowest byte position in the file, 
 in Big Endian Byte Order Mode, the most significant byte is written at the lowest byte position in the file. 
 
 For example, the `u32` value `0x12345678` is encoded as `[78 56 34 12]` in Little Endian Byte Order Mode,
  and `[12 34 56 78]` in Big Endian Byte Order Mode. 
  
An implementation MUST support at least one of the above Byte Order Modes, and SHOULD support both. 
Implementations MUST document which Byte Order Modes are supported. 

If an implementation is provided in the form of an Application Programming Interface, 
 it is RECOMMENDED that a method be provided to select the initial Byte Order Mode of the implementation,
 and additionally provide a method to change the Byte Order Mode. 
 
This specification does not define the mechanism under which the Byte Order Mode of a file is communicated,
 if such a mechanism is provided. 

### §2.6 End of File

The end of binary files is some point where the representation meaningfully terminates,
 and no further bytes can be obtained from the file. 

The End of File MUST NOT abruptly terminate the byte representation of any scalar, structure, or array type. 


## §3 Binary Types

### §3.1 Scalar Types

Binary Files can be broken into indivisible parts, known as scalar values. 
The representation of these values is defined by the type of the value, called a scalar type. 

With three exceptions, the representation of a scalar value depends on Byte Order Mode. 
These types are considered multibyte scalar type. The exceptions are called single-byte scalar types. 

The scalar types are as follows:
* u8, a 1 byte long, single-byte scalar type which represents an unsigned integer value in the range `[0,255]`. 
* i8, a 1 byte long, single-byte scalar type which represents a signed integer value in the range `[-128,127]`.
* u16, a 2 byte long, multibyte scalar type which represents an unsigned integer value in the range `[0,65535]`
* i16, a 2 byte long, multibyte scalar type which represents a signed integer value in the range `[-32768,32767]`
* u32, a 4 byte long, multibyte scalar type which represents an unsigned integer value in the range `[0,4294967295]`
* i32, a 4 byte long, multibyte scalar type which represents a signed integer value in the range `[-2147483648,2147483647]`
* u64, an 8 byte long, multibyte scalar type which represents an unsigned integer value in the range `[0,18446744073709551616]`
* i64, an 8 byte long, multibyte scalar type which represents a signed integer value in the range `[-9223372036854775808,9223372036854775807]`
* bool, a 1 byte long, single-byte scalar type which can either represent true or false.
* f32, a 4 byte long, multibyte scalar type which represents an IEEE 754 single-precision, binary floating-point number
* f64, an 8 byte long, multibyte scalar type which represents an IEEE 754 double-precision, binary floating-point number

Other specifications may define additional scalar types and the properties of such scalar types. 

### §3.1.1 Unsigned Integer Types

The scalar types `u8`, `u16`, `u32`, and `u64` are called unsigned integer types. 

The representation of values these types (with the special exception of `u8` which is exactly represented in its byte representation),
 is defined as follows:
 * The least significant byte is the value, modulo 256
 * The value is divided by 256 and its fractional part discard
 * The above process is repeated for the next least significant byte, until all bytes of the value are represented. 
 * The representation is then ordered according to the Byte Order Mode,
  where in Little Endian, the bytes of the representation are in the order they are obtained, and in Big Endian, the bytes are in the reverse of this order.

The length of the representation depends solely on the type, not the value. 
(For example, the 0x1F (in Little Endian Byte Order Mode) is `[1F]` as a `u8` value, 
`[1F 00]` as a `u16` values, `[1F 00 00 00]` as a `u32` value, 
and `[1F 00 00 00 00 00 00 00]` as a `u64` value.)

### §3.1.2 Signed Integer Types

The scalar types `i8`, `i16`, `i32`, `i64` are called signed integer types.

The representation of each type is similar to the representation of the same-sized unsigned integer type. 
If the value is exactly representable as a value of the respective unsigned type, then the representation is the same. 
If the value is not (IE. the value is negative), the absolute value (as the unsigned type) is taken, 
 followed by the bitwise negation of that value, then 1 is added, and the representation of the result is the representation of the signed value. 
 
### §3.1.3 Boolean Type

The scalar type `bool` is the boolean type. It is a single byte type, and can stored one of two values,
 `true` or `false`.
 
The byte representation of `true` is `[01]`, and the byte representation of `false` is `[00]`,
 a boolean value MUST NOT have any other representation. 
 
### §3.1.4 Floating Point Types

The scalar types `f32` and `f64` are floating-point types. 

The representation of these types are defined by IEEE 754/IEC 559.
`f32` is a single-precision binary floating point type (24 significand bits, 7 expontent bits), 
and `f64` is a double-precision binary floating point type (52 significand bits, 11 expontent bits). 

## §3.2 Array Types

Types may be defined as an array. Arrays contain 0 or more values of a single type,
 where the length is either defined, or read from a different part of the file. 
 
Array types are formed as `<type> [<length>]`. *length* MUST NOT be negative. 

In a special case, an array type can be formed as `<type> []`, when the `length` is not known,
 however it is possible to otherwise determine when the array ends.
 
As members in structure types, the member name is written after *type*, but before the extent (`[<length>]`). 

The binary representation of an element of an array immediately follows the
 binary representation of the previous element, if such an element exists. 
 There is no padding between elements of arrays. 

## §3.3 Structure Types

Types may be defined as a structure composed of multiple values of various types. 
The types of these values are called Structure types. 

A structure type is defined as
```
<name>{
    <member1>;
    <member2>;
    ...
};
```

and is referenced as *name*. 

Where each member is either `<type> <name>` or `<type> <name>[<length>]`. 

The name of a member with an integer type can be used as the length of any array member. 
The length MUST NOT be a negative value if signed. 

The binary representation of each member directly follows the binary representation
 of the previous member, if such a member exists. There is no padding between members of structures. 
 
Additionally, no padding may be added before the first member of a structure, or after the last member of a structure. 

The byte order mode does not affect the order of the members of a structure,
 however it does affect the layout of any multibyte scalar members.

### §3.4 Union Types

In addition to structure types, union types may be defined. 
The definition is similar to structure types, though the key word `union` directly preceeds the definition. 

Unlike with structure types, only the binary representation of one member, called the active variant, 
 appears in the file. How the active variant is chosen or determined is not defined by this specification. 
 
### §3.5 Predefined Structure types

Various types are defined by this specification, for use in other specifications. 

#### §3.5.1 Duration and Instant types

```
duration{
    i64 seconds;
    u32 nanos;
};
instant{
    i64 seconds;
    u32 nanos;
};
```

duration represents a duration of time as a period of seconds, and nanos-of-second. 
*nanos* MUST be less than 1000000000.

instant represents an instant in time, encoded as a duration from 1970-01-01T00:00:00.000000000Z (the unix epoch). 

#### §3.5.2 String type

```
string{
    u16 length;
    u8 chars[length];
}
```

A string encodes a UTF-8 String with *length* given in bytes.

String values have various rules about the representation, in particular:
* all bytes in *chars* MUST NOT have the value `0`
* A continuation byte may not appear except as part of a multibyte character
* a multibyte character MUST be completed with the appropriate number of bytes before the end of the string.
* No character may be longer than 3 bytes. 
* If a character introduces a surrogate pair, there must be a following character which validly completes the surrogate pair
* If a character completes a surrogate pair, there must be a proceeding character which introduces a surrogate pair.

#### §3.5.3 Version and UUID type

```
version{
    u8 major;
    u8 minor;
};

uuid{
    u64 most;
    u64 least;
};
```
 
A version value encodes the version *major*+1.*minor*. 

A uuid value encodes the uuid given by *most* and *least*. 
Regardless of the byte order mode, *most* is always the 64 most-significant bits, and *least* is always the 64 least-significant bits. 
(For example, in Little Endian Byte-order mode, the UUID 00112233-4455-6677-8899-AABBCCDDEEFF, 
 is encoded as `[77 66 55 44 33 22 11 00 FF EE DD CC BB AA 99 88]`, 
 and as `[00 11 22 33 44 55 66 77 88 99 AA BB CC DD EE FF]` in Big Endian Byte Order mode).
 
(Note, the decision to not reverse the order of *most* and *least*
 in Little Endian Byte Order mode is a legacy choice. It is kept intact in this specification
 due to the existance of legacy implementations and files from legacy implementations.
 In retrospect, this was a poor choice. The same applies to version with *major* and *minor*.
 In both cases, it prohibits reading the structure value as a single scalar value,
 u16 for version, and an extension type u128 for uuid, as the value would be incorrect in Little Endian Byte Order Mode).

### §4 GNU Free Documentation License

Copyright (C)  2019  Connor Horamn.
Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
A copy of the license can be obtained at <https://www.gnu.org/licenses/fdl-1.3.en.html>