# Dagor Engine BitStream (Stripped)

a stripped-down version of the `BitStream` class implementation from the Dagor Engine. It includes only the essential headers required for the `BitStream` functionality to compile and work independently.

## Usage for Parsing Data

### 1. Initialization

To parse existing data, initialize the `BitStream` with a pointer to your data buffer and its size. Set `copy` to `false` to avoid unnecessary memory duplication.

```cpp
#include "bitstream.h"

const uint8_t* myDataBuffer = /* ... pointer to data ... */;
size_t myDataSizeInBytes = /* ... size of data ... */;

danet::BitStream stream(myDataBuffer, myDataSizeInBytes, false);
```

### 2. Reading Data

The `BitStream` provides methods to read various data types. Most `Read` methods return `false` if there isn't enough data left in the stream.

-   **Basic Types:** Use the templated `Read(T& value)` method.
    ```cpp
    int32_t intValue;
    float floatValue;
    bool boolValue;

    if (!stream.Read(intValue)) {};
    if (!stream.Read(floatValue)) {};
    if (!stream.Read(boolValue)) {};
    ```

-   **Compressed Integers:** Use `ReadCompressed` for variable-length encoded integers
    ```cpp
    uint32_t compressedUint;
    int16_t compressedInt;

    if (!stream.ReadCompressed(compressedUint)) {}
    if (!stream.ReadCompressed(compressedInt)) {}
    ```

-   **Strings:** Reads a `uint16_t` length prefix, then the string characters. Works with `EASTL`-compatible string types.
    ```cpp
    #include <EASTL/string.h> 

    eastl::string myString;
    if (!stream.Read(myString)) {}
    ```

-   **Containers:** Reads a `uint32_t` size prefix, then reads each element. Works with `EASTL`-compatible containers like `eastl::vector`.
    ```cpp
    #include <EASTL/vector.h> 

    eastl::vector<int> myVector;

    // Reads size, resizes, then reads elements
    if (!stream.Read(myVector)) {}
    ```

-   **Raw Bits/Bytes:** Use `ReadBits` or `Read` for raw data chunks.
    ```cpp
    uint8_t bitBuffer[10];

    if (!stream.ReadBits(bitBuffer, 20)) {}

    char byteBuffer[50];
    if (!stream.Read(byteBuffer, 50)) {}
    ```

-   **Aligned Bytes:** Use `ReadAlignedBytes` to read data starting from a byte boundary.
    ```cpp
    stream.AlignReadToByteBoundary();
    uint8_t alignedBuffer[100];
    if (!stream.ReadAlignedBytes(alignedBuffer, 100)) {}
    ```

### 3. Stream Navigation and State

-   **Check Remaining Data:**
    ```cpp
    uint32_t unreadBits = stream.GetNumberOfUnreadBits();
    if (unreadBits == 0) {
      // end of stream
    }
    ```

-   **Get/Set Position:** Use `GetReadOffset()` (returns bits) and `SetReadOffset(bitPosition)` to manage the current reading position.
    ```cpp
    uint32_t currentOffset = stream.GetReadOffset();

    // Move back 8 bits (1 byte)
    stream.SetReadOffset(currentOffset - 8);
    ```

-   **Skip Data:** Use `IgnoreBits(bitsToSkip)` or `IgnoreBytes(bytesToSkip)`.
    ```cpp
    // Skip 4 bytes
    stream.IgnoreBytes(4);
    ```

-   **Alignment:** Use `AlignReadToByteBoundary()` to advance the read offset to the start of the next byte. This is often needed before reading byte aligned structures or blocks of data.

### Example

Refer to [main.cpp](src/main.cpp)