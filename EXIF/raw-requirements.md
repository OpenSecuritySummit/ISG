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

## RISKS AND RECOMMENDATIONS

**Data Hiding and Data Attack** – Files with unknown tags or known tags with incorrect values for type/number of values can contain information that may not be presented to an end user. PHP code has been known to be placed in Exif tags and cause denial of service or execution of arbitrary code10. Careful whitelisting of the tag values is necessary to prevent Exif from carrying PHP attacks. Several well published attacks have used PHP scripts embedded in Exif tags.

1. Validate – Validate that the Exif Tag ID, bytes 1-2, match a known tag defined in the baseline specification for Exif version 2.3.
2. Validate – Validate that the Exif tag is on a whitelist of known good tags.
3. Validate –Validate that the field type, bytes 3-4, is a known, acceptable value for the tag.
4. Validate – Validate that the number of values, bytes 5-8 or bytes 5-12, do not exceed a prescribed value in the specifications.
5. Validate – Validate that the tag data or offset to the data, bytes 9-12 contains well-formed data for the tag according to the specifications (e.g., a data type of RATIONAL contains two LONG values).
6. External Filtering Required – External filtering is required for tags with a field type, bytes 3-4, of ASCII because they can contain large volumes of text that may be unrelated to the image.
7. Remove- Remove all tags and metadata that are not required by Baseline Exif renders to be processed. This may compromise image quality in certain images.
8. Reject – Reject files that contain unknown tags.
9. Reject – Reject files that contain malicious code in tags.

**Exif Tags**

The following constructs address specific Exif tags that can contain data risks. Note that the structure of the tags should be compliant with Section 4.1.2. Note that these constructs will not address structural validation.

**MakerNote Tag**

## OVERVIEW

The MakerNote tag, with tag id 0x92 7c, contains proprietary information that is not always disclosed to the general public11. This tag is application specific, and it cannot be reliably parsed or extracted since the format is unknown. The structure and contents of the tag are undefined. The maximum number of MakerNote tags allowed to exist in Exif is also undefined. This could lead to a large amount of undefined data residing in this location.

## RISKS AND RECOMMENDATIONS

**Data Hiding, Data Disclosure and Data Attack** - MakerNote tags that are not validated can pose a risk to application parsers and may contain sensitive data. Because the MakerNote data format is proprietary, the tag data may contain obfuscated information about the image (placed intentionally or unintentionally), which cannot be reliably verified or inspected. The content of the tag value may be an active content script12 which could allow an attacker to gain privileges on a remote machine when an application attempts to process the MakerNote tag.

1. Remove- Remove all MakerNote tags in the Exif IFD. Note this will cause loss of information when using proprietary applications that understand the MakerNote tag.

**UserComment Tag**

## OVERVIEW

The UserComment tag, tag id 0x92 86, allows for additional comments to be added, in a defined character code, to the image file. The current set of character codes include: ASCII, JIS, Unicode, and Undefined.

## RISKS AND RECOMMENDATIONS

**Data Hiding and Data Attack** - Tag values that are not validated can pose a risk to application parsers and may contain sensitive data. The content of the tag value may be an active content script which could allow an attacker to gain privileges on a remote machine13.

1. External Filtering Required – Pass the contents of the UserComment tag to a text filter that is capable of dirty word filtering and other text filtering techniques to ensure that comment is approved textual data.

**GPS Tag Sensitivity**

## OVERVIEW
GPS data often contains information that may be sensitive in nature. Sensitive data includes position (latitude, longitude, and altitude) about the captured image.

## RISKS AND RECOMMENDATIONS

**Data Disclosure** – GPS tags may contain data that is sensitive or data that should not be disclosed.

1. Validate – Validate that the GPS tag is on a whitelist of acceptable GPS tags and the values are within approved ranges.
2. Remove – Remove any tags that are not on the accepted whitelist or are within a approved ranges.
3. Reject – Reject files that contain GPS tags that are not on the whitelist.

## Audio

#### OVERVIEW

Exif audio files detail methods for the writing of audio files with specific components. The Exif audio components include:
 Audio file format
 Detailed structure of the audio data
 Chunks within the file
 File naming conventions

The audio data recorded in an Exif audio file is written in RIFF tagged file structure. RIFF files are written in blocks of data and are referred to as chunks. The most commonly recorded attributes are written in an INFO list. The Exif attributes are written to the Exif chunks. Chunk data typically contains three components: chunk ID, chunk size, and chunk data. The only valid Exif audio chunks are listed in Table 3-4.

## RISKS AND RECOMMENDATIONS

**Data Hiding** – Data hiding may be accomplished by inserting data into the Exif audio chunks that are not validated as part of the standard. The Specification guidance recommends that applications skip information chunks that they cannot process or are unknown.

1. Validate – Validate that only the known Exif Audio Chunks are present in the audio file.
2. Validate – Validate that the mandatory sub-chunk data “ever” is present. See Table 3-4 for a description of the “ever” sub-chunk.
4-21
3. External Filtering Required – Validate the sub-chunk data by using an external ASCII filter and that he contents adhere to acceptable values.
4. Remove – Remove any Exif audio chunks that are not defined in the specification.
5. Remove – Remove the undefined “emnt” chunk, since any manufacturer defined data can reside this this chunk.
6. Reject – Reject files that contain Chunks not contained in the specification.