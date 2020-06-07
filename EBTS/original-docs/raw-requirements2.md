## EBTS CONSTRUCTS

The following sections present the EBTS constructs, the risks associated with each one, as well as recommendations to help mitigate each risk. The initial constructs are focused on the file structure and common issues that are found in several EBTS record types.

Recommendations in constructs that describe a removal or replacement option may require a complete build of a new file or the new record. Any change to an EBTS record or field will most likely require realignment of the file, which means adjusting length values and and possibly changing the Type-1 or Type-2 record (from Table 3-1) to reflect the new data. Type-1 and Type-2 records contain structural and descriptive information about the remainder of content in the EBTS file. Replacement or removal of a field may also involve zeroing out (0x30) or injecting spaces (0x20) into the fields. This can be done while preserving the original length of the field and will not require restructuring the entire file. This option can be taken to simplify the filtering process while still replacing or removing content. It is the filter developer’s responsibility to understand that manipulation of records or fields may require restructuring of the entire EBTS data.

## Record/Field Numbers and Data Types

### OVERVIEW:

This section focuses on generic field records and various different field types including numeric, alphabetic, alphanumeric, and binary. All EBTS files start out with a single Type-1 record (covered in construct 4.10). This construct covers the formatting of the record and field numbers for all fields in EBTS. For tagged-field logical records (Type-1, 2, 9, 10-17, and 99), the field number is defined in the form of “TT.xxxxxxxxx”. The first number is the logical record type number and can be up to two digits long. For example, a Type-99 record will start with “99.x” to designate that this field belongs to a Type-99 record. In the case of a Type-2 record, the field will start with “2.x”. The second number is the field number and can be variable in length, but is a maximum of 9 digits long. It is valid for a number to contain preceding zeros before the non-zero field number. This number is an unsigned integer and should be previously defined in a specification. Following the field number is a colon which defines that the next portion is the field value itself (which could be in a variety of formatted data).

## RISKS AND RECOMMENDATIONS:

**Data Hiding:** Unknown, deprecated, or unused record/field numbers could be ignored by applications. Fields with invalid character types may be an attempt to hide data within EBTS data.
1. Validate: Check that each record and field number for any record is valid from one of the EBTS specifications.
2. Remove: Remove any deprecated fields in the EBTS file.
3. Validate: Check that the remainder of the field contents aligns with a character type as denoted by the appropriate standard.
4. Validate: If the field content has a type or format (such as a date), check that the field adheres to the correct format.
5. Validate: If the field specifies a minimum and/or maximum value, verify that the value meets that criteria.

