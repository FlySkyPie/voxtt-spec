# VoxTT 0.1.0 (Draft)

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Abstract

VoxTT is a specification for storing voxel data in [SQLite](http://sqlite.org/) databases for immediate usage and for transfer.
VoxTT files, MUST implement the specification below to ensure compatibility with devices.

## Terms

* Voxel: A value on a regular grid in three-dimensional space, the concept is similar to pixel but in 3D.
* Global coordinate: The absolute coordinate of a voxel location in the three-dimensional space. Variables known as `x`, `y`, and `z`.
* Sub-coordinate: The relative  coordinate of a voxel location in the chunk. Variables known as `x'`, `y'`, and `z'`.
* Chunk coordinate:  The absolute coordinate of a chunk. Variables known as `Cx`, `Cy`, and `Cz`.

## Coordinates

* Coordinate MUST following 3 dimensional Cartesian coordinate system.
* A voxel located at `(x, y, z)`, then:
  * The global coordinate of voxel MUST be `(x, y, z)`
  * The cub-coordinate of voxel MUST be `(x MOD size, y MOD size, z MOD size)`, `size` is the length, width height of chunk.
  * Chunk coordinate MUST be MUST be `(⌊x/size⌋, ⌊y/size⌋, ⌊z/size⌋)`.

## Database Specifications

VoxTT files SHALL be valid SQLite databases of
[version 3.0.0](http://sqlite.org/formatchng.html) or higher.
Only core SQLite features are permitted; VoxTT files SHALL NOT require extensions.

VoxTT files databases MAY use [the officially assigned magic number](http://www.sqlite.org/src/artifact?ci=trunk&filename=magic.txt)
to be easily identified as VoxTT files.

## Database

Note: the schemas outlined are meant to be followed as interfaces.
SQLite views that produce compatible results MAY be used instead.
For convenience, this specification refers to tables and virtual tables (views) as tables.

## Charset

All text in `text` columns of tables in an VoxTT files MUST be encoded as UTF-8.

### Metadata

#### Schema

The database MUST contain a table or view named `metadata`.

This table or view MUST yield exactly two columns of type `text`, named `name` and `value`. A typical create statement for the `metadata` table:

```sql
CREATE TABLE metadata (name text, value text);
```

#### Content

The `metadata` table is used as a key/value store for settings. It MUST contain these two rows:

* `name` (string): The human-readable name of the VoxTT file.
* `format` (string): The file format of the chunk data: `FLAT_PNG_V1`, or `CUSTOM`; If the format didn't specified in this document.
* `size` (number): The size of one cubic chunk, which pixels of length, width or height.
* `version` (string): The version of VoxTT specification.

The `metadata` table SHOULD contain these five rows:

* `timestamp` (string): The timestamp that file created, format SHOULD follow ISO 8601.
* `license` (string): Declare the license that the VoxTT file, SHOULD using [SPDX license ID](https://spdx.org/licenses/), also MAY using full text of the license.
* `credit` (string): If the license require  appropriate credit, SHOULD leave the credit in this field.
* `author` (string): If the license require  appropriate author, SHOULD leave the author's name in this field.
* `contributors` (string): If the license require  appropriate contributors, SHOULD leave the contributor's names in this field.

The `metadata` table MAY contain this row:

* `description` (string):  A description of the VoxTT file's content.

The `metadata` table MAY contain additional rows for VoxTT file for other purposes.

### Chunks

#### Schema

The database MUST contain a table named `chunks`.

The table MUST contain three columns of type `integer`, named `x`, `y`, `z`, and one of type `blob`, named `chunk_data`.
A typical `create` statement for the `tilchunkses` table:

```sql
CREATE TABLE chunks (x integer, y integer, z integer, chunk_data blob);
```

The database MAY contain an index for efficient access to this table:

```sql
CREATE UNIQUE INDEX id on chunks (x integer, y integer, z integer, chunk_data blob);
```

#### Content

The chunks table contains voxels and the values used to locate them.
The `x`, `y`, and `z` columns MUST store chunk coordinate.

The `tile_data` column MUST contain the raw binary image or other's file for the associated tile as a `blob`.

## Format

### FLAT_PNG_V1

- When the VoxTT file specified as `FLAT_PNG_V1` format at `metadata` table. The file in the `chunk_data` MUST follow this specification.
- The width of png MUST be `size*size`, and height MUST be `size`, `size` is specified at `metadata` table.
- The voxel is stored as pixel in the png, and MUST following:
  - Coordinate of pixel is known as `Px` and `Py`.
  - `x' = Px MOD size`
  - `y' = Py`
  - `z' = ⌊Px/size⌋`