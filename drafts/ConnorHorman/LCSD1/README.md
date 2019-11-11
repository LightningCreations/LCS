# Binary IO Format

Draft Date: 2019-11-10

This document is released under the terms of the GNU Free Documentation License. See §5 "GNU Free Documentation License" for details. 

## §1 Purpose of this document

This document specifies the method in which Binary Files should be encoded, as referenced by other LCS Documents. 
This document does not specify the method in which the files are stored, transported, or interpreted, only the encoding. 
Additionally, this document does not specify the mechanism for which the encoding is produced or consumed. 

### §1.1 Further Encoding

This document defines the binary representation of files using this encoding. 
 However, this need not correspond precisely to the physical representation on disk or in transport. 
 
 In particular, files may be further encoded in a way that preserves the representation,
  but alters it in some reversible manner, such as encryption, lossless compression, or textual encoding (such as base64). 

It SHOULD be possible to recover the content of the file from the further encoding, independant of the implementation which created it.
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


### §2.4 Byte Order Mode

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

## §3 Binary Types

### §3.1 Atomic Types



