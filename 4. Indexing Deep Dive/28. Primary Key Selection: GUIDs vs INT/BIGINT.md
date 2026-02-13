# BIGINT

Canonical documentation for [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md). This document defines concepts, terminology, and standard usage.

## Purpose
The [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) (Large Integer) data type exists to provide a standardized mechanism for storing and manipulating integer values that exceed the capacity of standard 32-bit integer types. It addresses the requirement for high-precision counting, unique identification in massive datasets, and the representation of discrete temporal units (such as nanoseconds) without the precision loss inherent in floating-point arithmetic.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the 64-bit integer standard common across modern computing environments.

## Scope
This documentation covers the theoretical and practical application of 64-bit signed integers.

> [!IMPORTANT]
> **In scope:**
> * Core functionality of 64-bit signed integer storage.
> * Theoretical boundaries and mathematical ranges.
> * Data integrity and precision standards.

> [!WARNING]
> **Out of scope:**
> * Specific vendor implementations (e.g., MySQL vs. PostgreSQL vs. SQL Server specific syntax).
> * Arbitrary-precision integers (BigNum/BigInteger libraries) that grow dynamically beyond 64 bits.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Bit | The most basic unit of information in computing, representing a 0 or 1. |
| Byte | A group of 8 bits. |
| Two's Complement | The mathematical operation used to represent signed binary numbers where the most significant bit indicates the sign. |
| Overflow | A condition occurring when a calculation produces a result exceeding the maximum capacity of the data type. |
| Underflow | A condition occurring when a calculation produces a result lower than the minimum capacity of the data type. |
| Precision | The ability to represent a value exactly without rounding or approximation. |

## Core Concepts
The fundamental concept of [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) is the utilization of 8 bytes (64 bits) of storage to represent a whole number. Unlike floating-point types (like DOUBLE or FLOAT), [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) is exact. Every increment of 1 is distinct and preserved.

The range of a signed [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) is determined by the formula:
$-2^{n-1}$ to $2^{n-1} - 1$, where $n=64$.

This results in a range of:
**-9,223,372,036,854,775,808** to **9,223,372,036,854,775,807**.

> [!TIP]
> Think of [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) as a high-capacity container. While a standard INT can hold the population of a single country, a [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) can assign a unique ID to every grain of sand on Earth several times over.

## Standard Model
The standard model for [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) is the **64-bit signed two's complement integer**. 

1.  **Storage:** Fixed 8-byte allocation.
2.  **Alignment:** In many hardware architectures, [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) values are most efficient when aligned to 8-byte boundaries in memory.
3.  **Comparison:** Operations follow standard algebraic rules.
4.  **Sorting:** Values are sorted numerically, not lexicographically (as strings would be).

## Common Patterns
*   **Surrogate Primary Keys:** Used in distributed systems or large-scale databases where the number of records is expected to exceed 2.1 billion (the limit of a 32-bit INT).
*   **High-Resolution Timestamps:** Storing time as the number of microseconds or nanoseconds since a specific epoch (e.g., Unix Epoch).
*   **Financial Cents:** Storing currency values as integers (e.g., $1.00 is stored as 100) to avoid the rounding errors associated with decimals.
*   **Bitmasking:** Using the 64 available bits as individual boolean flags for complex permission systems.

## Anti-Patterns
*   **Over-Engineering:** Using [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) for fields with known small ranges (e.g., "Age" or "Day of Week"). This wastes 4-7 bytes per row, which can significantly impact cache performance and storage costs at scale.
*   **Floating-Point Conversion:** Casting a [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) to a FLOAT or DOUBLE for calculation. This often leads to "Precision Loss," where the last few digits of a large number are rounded.
*   **String Storage:** Storing large numbers as strings (VARCHAR) instead of [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md). This prevents mathematical optimization and increases storage overhead.

> [!CAUTION]
> Avoid using [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) for values that require fractional components unless you are using a fixed-point scaling pattern (like storing cents).

## Edge Cases
*   **The JavaScript Problem:** Many environments (like standard JavaScript) treat numbers as 64-bit floats (IEEE 754). These can only safely represent integers up to $2^{53} - 1$. When a [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) larger than this is passed to a web browser, it may be corrupted unless handled as a specialized "BigInt" type or a string.
*   **Sign Flip on Overflow:** In systems without strict overflow checking, adding 1 to the maximum positive [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) value will "wrap around" to the minimum negative value.
*   **Serialization:** When converting [BIGINT](4. Indexing Deep Dive/28. Primary Key Selection: GUIDs vs INT/BIGINT.md) to formats like JSON, the value may be truncated by the receiving parser if it does not support 64-bit integers natively.

## Related Topics
*   **INT (32-bit Integer):** The standard integer type for smaller ranges.
*   **DECIMAL/NUMERIC:** For values requiring both high precision and fractional components.
*   **UUID/GUID:** An alternative for unique identification that does not rely on sequential integers.
*   **Two's Complement:** The underlying binary representation logic.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-02-13 | Initial AI-generated canonical documentation |