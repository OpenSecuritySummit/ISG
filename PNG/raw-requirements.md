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

### 4.1 PNG Signature

The PNG Signature constitutes the first eight bytes of every PNG file. The first byte is
0x89 in order to prevent confusion with a text file. The next three bytes are 0x50, 0x43,
and 0x47, which evaluates to ‘PNG’ in ASCII. The next two bytes are 0x0d and 0x0a; this
combination is referred to as a CR-LF sequence. The seventh byte is 0x1a or control-Z
character. The final byte is 0x0a (a line feed).

#### RISKS AND RECOMMENDATIONS

Data Attack – Files that do not conform to the specification indicate that the file may
have been modified or created with an invalid application.

1. Validate – The first eight bytes are equivalent to: 0x89, 0x50 (‘P’), 0x43 (‘N’), 0x47
(‘G’), 0x0d, 0x0a, 0x1a, and 0x0a.
2. Reject – Files that do not conform to the specification requirements.



#### 4.2.1 Chunk Format
OVERVIEW
PNG Chunks should follow the PNG file signature and be formatted according to the
specifications. Chunks consist of at most four parts: a four-byte length field, a four-byte
18 chunk type field, a variable length chunk data field, and a four-byte cyclic redundancy
code (CRC). Note that if the length field is zero, then there is no chunk data field.

#### RISKS AND RECOMMENDATIONS

Data Hiding – Unrecognized chunks may contain information that applications will not
present to end users.
Data Attack – Malformed chunks may contain malicious code or trigger parsing
problems for applications.
1. Validate – The chunk type is defined by the specification or in a whitelist.
2. Validate – The length, chunk type, and chunk data match the CRC.
3. Remove – Chunks with unrecognized types.
4. Remove – Non-critical Chunks with length zero.
5. Reject – Files that contain data unrecognized chunks.
6. Reject – Files that contain invalid CRCs.



PRODUCT
PNG (Portable Network Graphics) Specification, Version 1.2
LOCATION
Chunks are found immediately after the PNG file signature.
4.2.2 Chunk Order
OVERVIEW
PNG Chunks should follow the PNG file signature and be formatted according to the
specifications. Chunks can appear in any order with the exception of two chunks: IHDR
and IEND. IHDR should be the first chunk and IEND should be the last chunk.
The format of a PNG file can be modified to include information before the IHDR. This
information may not be processed by image processing applications. Figure 4-1, below,
demonstrates a custom chunk, FAKE, that exists before the IHDR chunk and contains
26 bytes of hidden data.
Figure 4-1 Invalid Chunk before IHDR
19
Furthermore, data can be appended to end of the IEND chunk; this does not alter how
image processing applications render a PNG. Figure 4-2Error! Reference source not
found., below, shows the hexadecimal and ASCII representation of a file modified to
include information after the IEND chunk. The IEND chunk is underlined in blue and
the data that will not be processed by some image processing applications is underlined
in red.
Figure 4-2 Invalid Data after IEND
Finally, all chunks have a designated order as required by the specification. Table 4-1,
below, enumerates the ordering constraints on critical and ancillary chunks; it also
describes the whether there should be more than one of a chunk. If there are multiple
IDAT chunks, the order for uncompressing the data is derived from the order the
chunks are encountered, i.e. the first chunk will be uncompressed first and the last
chunk last.
20
Table 4-1 – Chunk Order Constraints [2]
RISKS AND RECOMMENDATIONS
Data Hiding – PNG Chunks before the IHDR or after the IEND chunk may not be
processed by PNG decoders and may not be presented to end users. Chunks that
appear more often than allowed, in the wrong order, occur without the presence of a
required chunk, or that should not occur because of the presence of another chunk may
be ignored by some image processing applications and serve as a vector for hidden
data.
Data Attack – PNG Chunks found out of order may contain malicious code or trigger
parsing problems for applications. Because this data should be ignored by applications
it may not contain an exploit, but may contain a payload because arbitrary data can be
stored here.
1. Validate – There is no data before the IHDR chunk.
2. Validate – There is no data after the IEND chunk.
3. Validate – Chunks that should not appear multiple times do not.
4. Validate – Chunks are found in acceptable locations according to the
specification.
5. Validate – sRGB and iCCP chunks are not both present.
21
6. Replace – The file with a version that contains chunks in the order described by
the specification.
7. Remove – Non-chunk data after the PNG file signature and before the IHDR
chunk.
8. Remove – Data after the IEND chunk.
9. Reject – Files that contain data in between the PNG file signature and IHDR
chunk or after the IEND chunk.
PRODUCT
PNG (Portable Network Graphics) Specification, Version 1.2
LOCATION
The IHDR chunk is found immediately after the PNG file signature, and the IEND
chunk is the last chunk in a PNG file.
4.2.3 Critical Chunks
Critical chunks are required for processing a PNG data stream. The specification defines
four such critical chunks: IHDR (Image Header), IEND (Image Trailer), IDAT (Image
Data), and PLTE (Palette).
IHDR Image Header
OVERVIEW
The IHDR chunk is a fixed length and fixed format chunk. The data field will always
have a length of 0x0d (13). The type field value should be ‘IHDR.’ The data field is 13
bytes are organized as follows:
 Bytes 1-4 is the image width; this must be an unsigned integer and non-zero.
 Bytes 5-8 is the image height; this must be an unsigned integer and non-zero.
 Byte 9 is the bit depth; this must be a single byte integer that denotes the number
of bits per sample or palette index. The values can only be 1, 2, 4, 8, or 16 and
vary with the type of image (e.g., grayscale, palette based, etc.). Note that this is
not the same as color depth, which indicates the number of bits per pixel.
 Byte 10 is the color type; this must be a single byte integer that denotes the PNG
image type (e.g., true color, indexed color, etc.). The valid values are 0, 2, 3, 4,
and 6.
 Byte 11 is the compression method; this must be a single byte integer that
indicates the compression method. The only valid value is 0, which means that
the inflate/deflate compression method is used.
 Byte 12 is the filter method; this must be a single-byte integer that indicates how
to preprocess the data before compression. The only valid value is 0, which
22
means adaptive filtering will be used and any one of the five filtering methods in
the specification will be used, e.g. Sub.
 Byte 13 is the interlace method; this must be a single byte integer that indicates
whether or not the image data is interlaced. The valid values are 0 (not
interlaced) or 1 (Adam7 interlaced).
Error! Reference source not found., below, deconstructs an IHDR chunk in a PNG file.
The data on the left is a hexadecimal representation of a PNG file and the data on the
right is an ASCII representation. The blue line indicates the chunk data length, which is
0x0d (13). The green line is the chunk field type, which evaluates to ‘IHDR’. The next
thirteen bytes is the actual data for the IHDR chunk. The red line indicates the image
width and the orange line indicates the image height. Each byte above the yellow line is
a different field; the first byte is the bit depth, the second byte is the color type, the third
byte is the compression method, the fourth byte is the filter method, and the fifth byte is
the interlace method. The final, purple line is the CRC for the chunk. Note that it was
observed that the upper bits of these fields could be modified without affecting how a
file was processed by an application. These fields could contain small amounts of data
that would not be observable by an end user.
The IHDR chunk can be modified and still be processed by some image processing
applications; for example, the length can be changed to include extraneous information
(in addition to valid information) that will be disregarded. Figure 4-4, below, shows the
length is doubled from 0x0d to 0x01a, which allows thirteen more bytes of data. The
CRC wasn’t recalculated in this example; however, some image processing applications
do not verify the chunk using the CRC If the CRC is not verified, then the file could
have been modified before reaching its destination and could be corrupt or malicious.
Figure 4-4 Modified IHDR Length Field
Modifying the image height or width in the IHDR chunk may cause image data to not
be rendered by image processing applications. Figure 4-5, below, depicts a PNG image
and resultant image after the height field in the IHDR chunk is modified. The height is
Figure 4-3 Deconstructed IHDR Chunk
23
changed from four pixels to two pixels. This can allow for image data to exist in the file
and not be rendered to an end user.
The valid bit depth settings with respect to color type can be found below. Table 4-2,
below, shows allowed bit depths per color types.
Table 4-2 – Supported Bit Depths per Color Type
PNG Image Type IHDR Color Type Allowed Bit Depths
Greyscale 0 1, 2, 4, 8, 16
Truecolor 2 8, 16
Indexed Color 3 1, 2, 4, 8
Greyscale + Alpha 4 8, 16
Truecolor + Alpha 6 8, 16
RISKS AND RECOMMENDATIONS
Data Hiding – The IHDR chunk may be modified to include information that an image
processing application may not present to end users.
Data Attack – If the IHDR chunk does not adhere to the specification it may contain
malicious code or trigger parsing problems for applications8.
1. Validate – The IHDR chunk contains a field type that evaluates to ‘IHDR’ in
ASCII.

8
 https://www.snort.org/rule_docs/1-3133
Figure 4-5 Modified IHDR Height Field
24
2. Validate – The image width and height is a non-zero positive number and is
equal to the width and height values found in the IHDR chunk.
3. Validate – The bit depth is 1, 2, 4, 8, or 16.
4. Validate – The color type is 0, 2, 3, 4, or 6.
5. Validate – The bit depth is supported by the color type.
6. Validate – The compression type is 0.
7. Validate – The filter method is 0.
8. Validate – The interlace method is 0 or 1.
9. Validate – The CRC of the IHDR chunk is valid.
10. Reject – Files that contain malformed or an invalid IHDR chunk.
PRODUCT
PNG (Portable Network Graphics) Specification, Version 1.2
LOCATION
The IHDR chunk is found immediately after the PNG file signature.