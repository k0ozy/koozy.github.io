---
title: "Pe Format"
date: 2026-05-11T22:19:26-04:00
draft: false
tags: []
categories: []
ShowToc: true
TocOpen: false
---

## PE Structure

### DOS Header (IMAGE_DOS_HEADER)
- PE Files are always prefixed with `0x4D` and `0x5A` to make `MZ`
- The DOS header is a data structure
- `e_lfanew` is a 4 byte value that holds an offset to the start of the NT Header 
	- Always at an offset of `0x3C`
```c
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // Offset to the NT header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```
### DOS Stub
- Error message that prints "This program cannot be run in DOS mode"
### NT Header (IMAGE_NT_HEADERS)
- Incorporates FileHeader and OptionalHeader
- Contains a signature to verify it "PE" `0x50` and `0x45`. Since it is a DWORD the signature will be represented as `0x50450000`
- Can be reached using the `e_lfanew` member in the DOS Header
- Structure is different depending on the machine's architecture
```c
typedef struct _IMAGE_NT_HEADERS {
  DWORD                   Signature;
  IMAGE_FILE_HEADER       FileHeader;
  IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```
#### File Header (IMAGE_FILE_HEADER)
- Can be reached from the NT header data structure
- `Characteristics` specifies attributes like if it is a DLL or console application
- Documentation - https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_file_header
```c
typedef struct _IMAGE_FILE_HEADER {
  WORD  Machine;
  WORD  NumberOfSections;
  DWORD TimeDateStamp;
  DWORD PointerToSymbolTable;
  DWORD NumberOfSymbols;
  WORD  SizeOfOptionalHeader;
  WORD  Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```
#### Optional Header (IMAGE_OPTIONAL_HEADER)
- Some common struct members
- `SizeOfCode` - Size of the .text section
- `AddressOfEntryPoint` - Offset to the entry point (Typically main)
- `BaseOfCode` - Offset to the start of the .text section
```c
typedef struct _IMAGE_OPTIONAL_HEADER64 {
  WORD                 Magic;
  BYTE                 MajorLinkerVersion;
  BYTE                 MinorLinkerVersion;
  DWORD                SizeOfCode;
  DWORD                SizeOfInitializedData;
  DWORD                SizeOfUninitializedData;
  DWORD                AddressOfEntryPoint;
  DWORD                BaseOfCode;
  ULONGLONG            ImageBase;
  DWORD                SectionAlignment;
  DWORD                FileAlignment;
  WORD                 MajorOperatingSystemVersion;
  WORD                 MinorOperatingSystemVersion;
  WORD                 MajorImageVersion;
  WORD                 MinorImageVersion;
  WORD                 MajorSubsystemVersion;
  WORD                 MinorSubsystemVersion;
  DWORD                Win32VersionValue;
  DWORD                SizeOfImage;
  DWORD                SizeOfHeaders;
  DWORD                CheckSum;
  WORD                 Subsystem;
  WORD                 DllCharacteristics;
  ULONGLONG            SizeOfStackReserve;
  ULONGLONG            SizeOfStackCommit;
  ULONGLONG            SizeOfHeapReserve;
  ULONGLONG            SizeOfHeapCommit;
  DWORD                LoaderFlags;
  DWORD                NumberOfRvaAndSizes;
  IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
```
##### Data Directory
- Array of data type `IMAGE_DATA_DIRECTORY`
- Of size `IMAGE_NUMBEROF_DIRECTORY_ENTRIES` which has a constant value of 16
```c
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

```c
#define IMAGE_DIRECTORY_ENTRY_EXPORT          0   // Export Directory
#define IMAGE_DIRECTORY_ENTRY_IMPORT          1   // Import Directory
#define IMAGE_DIRECTORY_ENTRY_RESOURCE        2   // Resource Directory
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION       3   // Exception Directory
#define IMAGE_DIRECTORY_ENTRY_SECURITY        4   // Security Directory
#define IMAGE_DIRECTORY_ENTRY_BASERELOC       5   // Base Relocation Table
#define IMAGE_DIRECTORY_ENTRY_DEBUG           6   // Debug Directory
#define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE    7   // Architecture Specific Data
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR       8   // RVA of GP
#define IMAGE_DIRECTORY_ENTRY_TLS             9   // TLS Directory
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG    10   // Load Configuration Directory
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT   11   // Bound Import Directory in headers
#define IMAGE_DIRECTORY_ENTRY_IAT            12   // Import Address Table
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT   13   // Delay Load Import Descriptors
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR 14   // COM Runtime descriptor
```
##### Export Directory
- Contains information about functions and variables that are exported from the executable
- Contains the addresses of exported functions and variables which can be used by other executables to access the functions and data
- Usually found in DLLs that export functions (kernel32.dll exporting CreateFileA)

##### Import Address Table
- Contains information about the addresses of functions imported from other executable files
- Addresses are used to access the functions and data in the other executables (App.exe importing CreateFileA from kernel32.dll)

### PE Sections
- .text - Contains the executable code
- .data - Contains initialized data (variables initialized in the code)
- .rdata - Read only data (Constant variables prefixed with const)
- .idata - Import Tables. This is used by the PE Loader to determine which DLL files to load with the process, as well as the functions used from each DLL
- .reloc - Contains information on how to fix up memory addresses to program can be loaded into memory with no errors
- .rsrc - Used to store resources like icons and bitmaps
- Each PE Section has an IMAGE_SECTION_HEADER data structure. These are saved under the NT headers in a PE file and stacked above each other, with each structure representing a section
### Resources
### Additional References

- PE Overview - https://0xrick.github.io/win-internals/pe2/
- DOS Header, DOS Stub and Rich Header - https://0xrick.github.io/win-internals/pe3/
- NT Headers - https://0xrick.github.io/win-internals/pe4/
- Data Directories, Section Headers and Sections - https://0xrick.github.io/win-internals/pe5/
- PE Imports (Import Directory Table, ILT, IAT) - https://0xrick.github.io/win-internals/pe6/
