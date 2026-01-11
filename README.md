# Merkle Tree Extension Specification

- **Title:** Merkle Tree
- **Identifier:** <https://stacchain.github.io/merkle-tree/v1.0.0/schema.json>
- **Field Name Prefix:** `merkle`
- **Scope:** Item, Collection, Catalog
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Dependencies:**
  - [File Info Extension](https://github.com/stac-extensions/file) (Recommended for Deep Integrity)
- **Owner**: @jonhealy1

This extension specifies a way to ensure metadata integrity for STAC Items, Collections, and Catalogs by encoding them in a Merkle tree via hashing. Each STAC object (Item, Collection, or Catalog) is hashed using a hash function (e.g., SHA-256), and this hash is stored in the object's properties under the `merkle:object_hash` field. Details concerning the methods used for hashing are stored in a separate object called `merkle:hash_method`.

To produce the Merkle root identifier for a Collection or Catalog, the hashes from its child objects are aggregated. This structure allows for the generation of **Merkle Proofs**, which enable efficient verification of any individual STAC object against the Collection Root without requiring access to the entire dataset.

To achieve **Deep Integrity** (verifying not just the metadata, but the data files themselves), this extension utilizes fields defined in the **File Info Extension**. By binding the `merkle:object_hash` to the `file:checksum` of the assets, the cryptographic proof extends to the binary files themselves.

- **Examples:**
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item.
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection.
  - [Catalog example](examples/catalog.json): Shows the basic usage of the extension in a STAC Catalog.
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Table of Contents

- [Fields](#fields)
  - [Additional Field Information](#additional-field-information)
  - [Hash Method Object](#hash-method-object)
- [Deep Integrity & File Hashing](#deep-integrity--file-hashing)
- [Merkle Proofs](#merkle-proofs)
  - [Linking to a Proof](#linking-to-a-proof)
  - [Proof Object Schema](#proof-object-schema)
  - [Example Proof](#example-proof)
- [Computing Hashes and Merkle Roots](#computing-hashes-and-merkle-roots)
  - [Computing `merkle:object_hash`](#computing-merkleobject_hash)
  - [Computing `merkle:root`](#computing-merkleroot)
  - [Computing the Merkle Proof](#computing-the-merkle-proof)
- [Examples](#examples)
- [Relation types](#relation-types)
- [Contributing](#contributing)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (via File Info Extension recommendation)
- [ ] Links

| Field Name           | Type                                      | Description                                                                                                                                                                                                                                                                                                                       |
| -------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `merkle:object_hash` | string                                    | **REQUIRED** in Items, Collections, and Catalogs. A cryptographic hash of the object's metadata, used to verify its integrity.                                                                                                                                                                                                    |
| `merkle:hash_method` | [Hash Method Object](#hash-method-object) | **REQUIRED** in Collections and Catalogs. An object describing the method used to compute `merkle:object_hash` and `merkle:root`, including the hash function used, fields included, ordering, and any special considerations. Items inherit this from their parent Collection but may include it if they use a different method. |
| `merkle:root`        | string                                    | **REQUIRED** in Collections and Catalogs. The Merkle root hash representing the Collection or Catalog, used to verify the integrity of all underlying objects.                                                                                                                                                                    |

### Additional Field Information

#### `merkle:object_hash`

- **Type:** string
- **Description:** A cryptographic hash of the object's metadata (Item, Collection, or Catalog), computed according to the method specified in
  the `merkle:hash_method`. This hash allows users to verify that the object's metadata has not been altered.

#### `merkle:hash_method`

- **Type:** [Hash Method Object](#hash-method-object)
- **Description:**
  - An object that specifies how `merkle:object_hash` and `merkle:root` were computed, including:
  - The hash function used.
    - The fields included in the hash computation.
    - The ordering of hashes when building the Merkle tree.
    - Any special considerations (e.g., serialization format).
  - This provides transparency and allows users to accurately verify the hashes.
- **Usage:**
  - **Collections and Catalogs:** This object is **REQUIRED**.
  - **Items:** Items inherit the `merkle:hash_method` from their parent Collection by default. An Item can optionally include its own
    `merkle:hash_method` if it uses a different hash method than the Collection.

#### `merkle:root`

- **Type:** string
- **Description:** The Merkle root hash representing the Collection or Catalog. It is computed by building a Merkle tree from the `merkle:object_hash`
  values of its child objects and, optionally, its own `merkle:object_hash`. This root hash provides a single value that represents the integrity of
  all underlying objects.

### Hash Method Object

The `merkle:hash_method` object provides details about the hash computation method used for `merkle:object_hash` and `merkle:root`.

| Field Name    | Type       | Description                                                                                                                                                                                                                                       |
| ------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `function`    | string     | **REQUIRED**. The cryptographic hash function used (e.g., `sha256`, `sha3-256`).                                                                                                                                                                  |
| `fields`      | \[string\] | **REQUIRED** (for all objects). An array of fields included in the hash computation. Use `"*"` or `"all"` to indicate that all fields are included. For nested fields, dot notation should be used (e.g., `properties.datetime`, `assets.image`). |
| `ordering`    | string     | **REQUIRED** (for Collections and Catalogs). Describes how the hashes are ordered when building the Merkle tree (e.g., "ascending by hash value").                                                                                                |
| `description` | string     | Optional. Additional details or notes about the hash computation method, such as serialization format or any special considerations.                                                                                                              |

## Deep Integrity & File Hashing

To ensure the integrity of the underlying data files (e.g., Cloud Optimized GeoTIFFs), implementations are **STRONGLY RECOMMENDED** to use the
[File Info Extension](https://github.com/stac-extensions/file) in conjunction with this extension.

1. **Implement File Info Extension:** The Item MUST include the File Info Extension URI
   (`https://stac-extensions.github.io/file/v2.1.0/schema.json`) in its `stac_extensions` list.
2. **Calculate File Hash:** Compute the checksum of the physical asset file.
   - *Note:* The File Info Extension requires checksums to be **Multihash** compliant (hexadecimal string).
3. **Store in Asset:** Store this value in the Asset definition using the `file:checksum` field.
4. **Include in Merkle Hash:** Ensure the `assets` object (specifically the `file:checksum` fields) is included in the `merkle:object_hash`
   calculation.

This binds the physical file to the metadata. If the file on S3 is swapped or corrupted, its checksum changes, invalidating the Asset metadata,
which invalidates the Item hash, which invalidates the Collection Root.

## Merkle Proofs

While `merkle:hash_method` allows a user to verify an individual object's hash, verifying that an object belongs to a specific Collection Root would typically require access to the entire collection (to reconstruct the full tree). To enable efficient verification (O(log n)) without downloading the full dataset, implementations SHOULD provide a **Merkle Proof** (also known as an audit path or witness).

### Linking to a Proof
STAC objects SHOULD provide a link to their proof using the `merkle-proof` relation. This link may point to a static file (e.g., on S3) or a dynamic API endpoint.

```json
"links": [
  {
    "rel": "merkle-proof",
    "href": "https://example.com/collections/c1/items/i1/proof.json",
    "type": "application/json",
    "title": "Merkle Inclusion Proof"
  }
]
```

### Proof Object Schema

The resource at the `merkle-proof` link MUST return a JSON object with the following structure:

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `target_hash` | string | **REQUIRED.** The `merkle:object_hash` of the Item being proven. |
| `root` | string | **REQUIRED.** The `merkle:root` of the Collection/Catalog this Item belongs to. |
| `path` | [Proof Node] | **REQUIRED.** An ordered list of sibling hashes required to reconstruct the root. |

### Example Proof

This JSON allows a client to take the Item's hash, combine it with the hashes in path, and confirm the result matches root.

```json
{
  "target_hash": "3a7bd3e2360a8e7d9f5b1c2d4e6f7890abcdef1234567890abcdef1234567890",
  "root": "abc123def4567890abcdef1234567890abcdef1234567890abcdef1234567890",
  "path": [
    {
      "position": "right",
      "hash": "8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4"
    },
    {
      "position": "left",
      "hash": "2b26b46c6b3c9657f86494796342f7c00e62059c4033757f446574676528d223"
    }
  ]
}
```

## Computing Hashes and Merkle Roots

### Computing `merkle:object_hash`

1. **Prepare Metadata:**
   - Include all fields specified in `merkle:hash_method.fields`.
   - **Recommendation:** Explicitly include `assets` to capture `file:checksum` values.
   - If `fields` is `"*"` or `"all"`, include all fields of the object.
2. **Serialize Metadata:**
   - Use canonical JSON serialization (RFC 8785) with sorted keys to ensure reproducibility.
3. **Compute Hash:**
   - Apply the specified cryptographic hash function (e.g., SHA-256) to the serialized metadata.

### Computing `merkle:root`

1. **Collect Hashes:**
   - Gather `merkle:object_hash` values from all child objects (Items, Collections, or Catalogs).
   - Include the `merkle:object_hash` of the Collection or Catalog itself.
2. **Order Hashes:**
   - Order the hashes according to the method specified in `merkle:hash_method.ordering`.
3. **Build Merkle Tree:**
   - Pairwise hash the ordered hashes, proceeding up the tree until a single hash remains—the `merkle:root`.

### Computing the Merkle Proof

To generate a proof for a specific child object (Target) within a tree:

1. **Identify Target:** Locate the `merkle:object_hash` of the Target object in the sorted list of all children hashes.
2. **Initialize Path:** Create an empty list for the `path`.
3. **Traverse Tree:**
- Pair the Target hash with its **Sibling** (the adjacent hash in the sorted list).
- **Record Sibling:** Add the Sibling's `hash` and relative `position` ("left" or "right") to the `path`.
- **Hash Upwards:** Compute the parent hash by concatenating and hashing the pair.
- **Repeat:** Treat this new parent hash as the current Target and repeat the process until the **Root** is reached.
4. **Finalize:** The resulting ordered list is the `path`.

## Examples

### Item Example (With File Integrity)

```jsonc
{
  "type": "Feature",
  "stac_version": "1.1.0",
  "stac_extensions": [
    "https://stac-extensions.github.io/file/v2.1.0/schema.json",
    "https://stacchain.github.io/merkle-tree/v1.0.0/schema.json"
  ],
  "id": "item-001",
  "properties": {
    "datetime": "2024-10-15T12:00:00Z",
    "merkle:object_hash": "3a7bd3e2360a8e7d9f5b1c2d4e6f7890abcdef1234567890abcdef1234567890"
  },
  "assets": {
    "visual": {
      "href": "https://storage.example.com/item-001.tif",
      "type": "image/tiff; application=geotiff; profile=cloud-optimized",
      "file:checksum": "12204a1b..." // Multihash (SHA-256)
    }
  },
  "links": [
    {
      "rel": "merkle-proof",
      "href": "https://storage.example.com/proofs/item-001.json",
      "type": "application/json"
    }
  ]
}
```

### Collection Example

```jsonc
{
  "type": "Collection",
  "stac_version": "1.1.0",
  "id": "collection-123",
  "description": "Sample Collection with Merkle Root",
  "merkle:object_hash": "7890abcdef1234567890abcdef1234567890abcdef1234567890abcdef123456",
  "merkle:root": "abc123def4567890abcdef1234567890abcdef1234567890abcdef1234567890",
  "merkle:hash_method": {
    "function": "sha256",
    "fields": ["properties", "assets", "geometry"],
    "ordering": "ascending",
    "description": "Includes assets to ensure file integrity via file:checksum."
  },
  "stac_extensions": [
    "https://stac-extensions.github.io/file/v2.1.0/schema.json",
    "https://stacchain.github.io/merkle-tree/v1.0.0/schema.json"
  ],
  "links": [
    {
       "rel": "root",
       "href": "https://example.com/catalog-001.json"
    },
    {
       "rel": "merkle-proof", 
       "href": "https://example.com/proofs/collection-123.json",
       "type": "application/json",
       "title": "Proof of membership in Master Catalog"
    }
  ]
}
```

### Catalog Example

```jsonc
{
  "type": "Catalog",
  "stac_version": "1.1.0",
  "id": "catalog-001",
  "description": "Sample Catalog with Merkle Root",
  "merkle:object_hash": "abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890",
  "merkle:root": "f1e2d3c4b5a67890abcdef1234567890abcdef1234567890abcdef1234567890",
  "merkle:hash_method": {
    "function": "sha256",
    "fields": ["*"],
    "ordering": "ascending",
    "description": "Computed by including merkle:object_hash values of child objects in ascending order and building the Merkle tree."
  },
  "links": [
    {
      "rel": "child",
      "href": "collection-123.json"
    },
    {
      "rel": "child",
      "href": "collection-456.json"
    }
  ],
  "stac_extensions": ["https://stacchain.github.io/merkle/v1.0.0/schema.json"]
}
```

## Relation types

This extension does not introduce any new relation types. The standard STAC relation types should be used as applicable in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md). Instructions
for running tests are copied here for convenience.

**Note:** This extension is currently a proposal and is open for feedback from the STAC community. Your input is valuable to
refine and adopt the Merkle Root Extension.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```
