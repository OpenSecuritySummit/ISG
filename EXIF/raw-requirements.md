## Raw requirements extracted from PDF

Extracted from the [ISG pdf on EXIF](original-docs/CTR-U-OO-108403-18.pdf)

### Actions Recommendation 


| RecommendationAction |  Comments |
|----------------------|-----------|
| Validate | Verify the data structure’s integrity, which may include integrity checks onother components in the metadata. (This should almost always be arecommended action.)
| Replace | Replace the data structure or one or more of its elements with values that alleviate the risk (e.g., replacing a username with a non-identifying, harmless value or substituting a common name for all authors). |
| Remove | Remove the data structure or one or more of its elements and any other affected parts |
| External Filtering Required | Note the data type and pass the data to an external action for handling that data type (e.g., extract text and pass it to a dirty word search). |
| Review | Present the data structure or its constructs for a human to review. (This should almost always be recommended if the object being inspected can be revised by a human.) |
| Reject | Reject the message. |


## Raw requirements

#### 4.1 Image File Directory (IFD)

**Data Attack** – Files with invalid offsets, unordered tags, or unknown tags may indicate that the file was specially crafted to exploit a machine. By having offsets that create recursive loops among the offsets one could trigger a stack buffer overflow7. Tags that are not recognized may contain malicious code instead of data pertinent to the file. Because Exif applications are instructed by the specifications to ignore tags they do not
know, these fields are often not checked

1. Validate – Validate the offset to the next IFD evaluates to 0x0 or points to a valid
location. An example of a valid location is a location that has yet to be
referenced.
2. Validate – Validate that the IFD chain ends with an offset that points to 0x0.
3. Validate – Validate that the tags adhere to a whitelist of acceptable values. Tags
are documented in the Exif specifications and other application specific
documentation.
4. Reject – Reject files that contain malformed IFDs.

**Data Hiding and Data Disclosure** – Files with multiple IFDs will have content that cannot be viewed in some image rendering applications. This information may be sensitive in nature.

1. Validate – Validate the required set of tags are present in the IFD.
2. Validate – Validate the IFD chain ends with an offset that points to 0x0.
3. Validate – Validate the tags adhere to a whitelist of acceptable values. Acceptable
values are tags defined by the specification or an authoritative source8.
4. Replace – Replace files with multiple IFDs with a single flattened version.
5. Reject – Reject files that contain multiple IFDs.