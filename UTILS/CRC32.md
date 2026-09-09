# CRC-32 Hash

Unit: `CRC32`

Fast 32-bit hash using table lookup (ISO 3309, polynomial `$EDB88320`). Compatible with PHP `crc32()`, ZIP, PNG, gzip.

## Overview

CRC32 produces a 32-bit (4-byte) hash from arbitrary input data. Used for:
- High score tamper protection (HISCORE.PAS)
- Save game checksums
- File integrity verification

## Basic Usage

### Hash a String

```pascal
uses CRC32;

var
  hash: String;
begin
  hash := CRC32String('hello');
  WriteLn(hash);  { Outputs: 3610a686 }
end;
```

### Hash a File

```pascal
uses CRC32;

var
  CRC: TCRC32;
begin
  if CRC32File('DATA\FONT.PCX', CRC) then
    WriteLn('CRC: ', CRC32ToHex(CRC))
  else
    WriteLn('Error reading file');
end;
```

### Incremental Hashing

For data processed in chunks (used by HISCORE.PAS):

```pascal
uses CRC32;

var
  CRC: TCRC32;
begin
  CRC32Init(CRC);

  { Process data in chunks }
  CRC32Update(CRC, @buffer1, len1);
  CRC32Update(CRC, @buffer2, len2);

  { Finalize and get result }
  CRC32Final(CRC);
  WriteLn(CRC32ToHex(CRC));
end;
```

## API Reference

### Types

**TCRC32**
```pascal
TCRC32 = LongInt;
```
32-bit CRC value.

### Core Functions

**CRC32Init**
```pascal
procedure CRC32Init(var CRC: TCRC32);
```
Initialize CRC state to `$FFFFFFFF`. Call before first `CRC32Update`.

**CRC32Update**
```pascal
procedure CRC32Update(var CRC: TCRC32; Buf: Pointer; Len: Word);
```
Add data to hash. Can be called multiple times.
- `Buf`: Pointer to data buffer
- `Len`: Number of bytes to hash (max 65535)

**CRC32Final**
```pascal
procedure CRC32Final(var CRC: TCRC32);
```
Finalize hash (XOR with `$FFFFFFFF`). Call after all `CRC32Update` calls.

### Convenience Functions

**CRC32String**
```pascal
function CRC32String(const S: String): String;
```
Hash a Pascal string, return 8-character hex result.

**CRC32File**
```pascal
function CRC32File(const Path: String; var CRC: TCRC32): Boolean;
```
Hash entire file. Returns `True` on success, `False` on I/O error.

**CRC32ToHex**
```pascal
function CRC32ToHex(CRC: TCRC32): String;
```
Convert 32-bit CRC to 8-character lowercase hex string.

## PHP Compatibility

Output matches PHP 8's `crc32()` function:

```php
echo dechex(crc32("123456789")); // cbf43926
echo dechex(crc32("hello"));     // 3610a686
```

Same polynomial (`0xEDB88320`), same algorithm (CRC-32B / ISO 3309).

## Test Vectors

Verified against PHP `crc32()`:

```
CRC32("")            = 00000000
CRC32("a")           = e8b7be43
CRC32("abc")         = 352441c2
CRC32("123456789")   = cbf43926
CRC32("hello")       = 3610a686
CRC32("message digest") = 20159d7f
```

## Implementation Notes

- **Lookup table**: 256-entry table (1KB) computed at unit initialization
- **Polynomial**: `$EDB88320` (reflected form of `$04C11DB7`)
- **Algorithm**: One XOR + one table lookup + one shift per byte
- **No assembly**: Pure Pascal, works on any x86 CPU
- **No initialization call needed**: Table built automatically via unit `begin..end` block

## Performance

Fast on all target hardware (286+):

| Input Size | Time (286 @ 12 MHz) |
|------------|---------------------|
| 32 bytes   | < 1ms               |
| 256 bytes  | < 1ms               |
| 4 KB       | ~5ms                |
| 64 KB      | ~80ms               |

## Files

- **UNITS\CRC32.PAS** - Main implementation
- **TESTS\CRCTEST.PAS** - Test vectors (PHP-compatible)
- **TESTS\CCRCTEST.BAT** - Compile test

## See Also

- [HISCORE.PAS](../UI/HISCORE.md) - High score management (uses CRC32)
