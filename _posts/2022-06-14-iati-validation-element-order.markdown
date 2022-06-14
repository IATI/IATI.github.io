---
layout: post
title:  "Why does IATI validation enforce element order?"
author: alex_miller
date:   2022-06-14
categories: blog
---

If you've ever had to write software that imports or exports IATI data, you may have noticed that the order of the XML elements is considered a critical part of the schema validation. Why was this change made historically, and why does it persist today?

## History of the decision

Starting with IATI version 2.01, the order of elements was enforced within the schema. But oddly enough, if you look at the [change-log from version 1.05 to 2.01](https://iatistandard.org/en/iati-standard/upgrades/upgrade-changelogs/integer-upgrade-to-2-01/2-01-changes/), you won't find any reference to this change. How was such a decision made, and why?

My investigation into this decision began with the code. Namely, the [single-source of truth repository](https://github.com/IATI/IATI-Standard-SSOT). From there, it's possible to find links to the other repositories which define the technical IATI standard itself. Within the IATI Schemas repository, a search for "order" yielded an issue titled ['Implement 2.01 "Enforcing Order on the Activity Schema"'](https://github.com/IATI/IATI-Schemas/issues/138), which further linked to a support ticket. The original links within the issue are broken, but it appears that they were migrated from entries to articles some time in 2020. Thankfully, a Google search turned up the article, titled ["Enforcing Order on the Activity Schema"](https://support.iatistandard.org/hc/en-us/articles/214393986-Enforcing-Order-on-the-Activity-Schema) authored by Bill Anderson. The original date is lost, but given the Github issue is dated July 2014, it must have been posted some time prior to that. In this ticket, Bill writes:

> It is not currently possible to use the schema to validate for mandatory fields. This type of validation (cardinality) can only be enforced if the order in which elements appear is enforced. Enforcing order was specifically ruled out when the standard was drafted as this was felt to place an unnecessary burden on publishers.
> 
> We now believe that strengthening the core of the standard by ensuring that we can validate elements and attributes (that are either mandatory in all cases, or are conditionally mandatory depending on usage) far outweighs the extra burden placed on publishers to report elements in a particular order. This also makes it much easier for a publisher to check their own data using simple schema-validation tests.

## How was this decision technically implemented in the standard?

In version 1.05, this is how the `iati-activity` element is defined:

```xml
  <xsd:element name="iati-activity">
    <xsd:annotation>
      <xsd:documentation xml:lang="en">
        Top-level element for a single IATI activity report.
      </xsd:documentation>
    </xsd:annotation>
    <xsd:complexType>
      <xsd:choice minOccurs="1" maxOccurs="unbounded">
        <xsd:element ref="reporting-org"/>
        <xsd:element ref="iati-identifier"/>
        <xsd:element ref="other-identifier"/>
        <xsd:element ref="title"/>
        ...
        <xsd:any namespace="##other" processContents="lax"/>
      </xsd:choice>
    </xsd:complexType>
  </xsd:element>
```

And in version 2.01, this is how `iati-activity` is constructed:

```xml
  <xsd:element name="iati-activity">
    <xsd:annotation>
      <xsd:documentation xml:lang="en">
        Top-level element for a single IATI activity report.
      </xsd:documentation>
    </xsd:annotation>
    <xsd:complexType>
      <xsd:sequence>
        <xsd:element ref="iati-identifier" minOccurs="1" maxOccurs="1"/>
        <xsd:element ref="reporting-org" minOccurs="1" maxOccurs="1"/>
        <xsd:element name="title" type="textRequiredType" minOccurs="1" maxOccurs="1">
          <xsd:annotation>
            <xsd:documentation xml:lang="en">
              A short, human-readable title that contains a meaningful
              summary of the activity. May be repeated for different
              languages.
            </xsd:documentation>
          </xsd:annotation>
        </xsd:element>
        ...
        <xsd:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/>
      </xsd:sequence>
    </xsd:complexType>
  </xsd:element>
```

If you compare the activity schema for [version 1.05](https://github.com/IATI/IATI-Schemas/blob/version-1.05/iati-activities-schema.xsd) to [version 2.01](https://github.com/IATI/IATI-Schemas/blob/version-2.01/iati-activities-schema.xsd), you can see that these fields were made mandatory by changing element groupings from `xsd:choice` to `xsd:sequence.` Along with `xsd:all`, these three are the only groupings available to specify child elements for `xsd:complexType` (how elements like `iati-activity` are constructed).

`xsd:choice` is a grouping that, as it sounds, allows a choice of child elements. When `minOccurs` and `maxOccurs` are attached to `xsd:choice`, those parameters refer to the number of unique choices, not the number of indvidual child elements. `maxOccurs="1"` would let you choose any one child element, and then the `minOccurs` and `maxOccurs` attributes of that element define how many times it can or must occur. Therefore, child elements cannot be made strictly mandatory under `xsd:choice` because any child element may be substituted for another.

`xsd:all` is another grouping option that, under XSD version 1.0, greatly restricts how complex types may be defined. Any child element under `xsd:all` may occur zero or one time, without room for additional configuration. Additionally, `xsd:any` is forbidden as a child of `xsd:all`. Therefore, `xsd:all` could not be used for the `iati-activity` element since some children need to occur more than once, and `xsd:any` is needed to allow publishers to use arbitrary custom elements.

Lastly, `xsd:sequence` is the simplest of the three groupings, in that it simply enforces that child elements occur in the order in which they're defined, and according to their respective `minOccurs` and `maxOccurs` attributes. Given the restrictions placed upon `xsd:all` under XSD version 1.0, `xsd:sequence` was the only grouping available to IATI that would allow for validation to catch when mandatory elements were missing.

## Why is element order still enforced?

Now that we know order was originally enforced as a mechanism to allow for mandatory fields, and the mandatory ordering inherent in `xsd:sequence` was a byproduct of validating mandatory fields, the question arises: is it still necessary? Technically, no; but in practice, unfortunately, yes.

In April 2012, the World Wide Web Consortium (W3C), the standards agency responsible for XML, released an upgraded specification for XSD version 1.1. Within this upgraded specification, `xsd:all` was given the ability to have `xsd:any` as a child element, and the fixed `maxOccurs="1"` requirement was lifted. Technically speaking, if IATI were able to use XSD 1.1, we would be able to freely exchange our `xsd:sequence` groupings for `xsd:all`, and do away with element order as a mandatory part of validation.

However, in the 10 years since the launch of XSD 1.1, few XML libraries have implemented it. For example, the `libxmljs2` library that we currently use for the IATI Validator does not support XSD 1.1, and the author of the dependent library `libxmljs2-xsd` has explicitly written in their documentation:

> As of now, XSD 1.1 is not supported, and the author does not actively work on it. Feel free to submit a PR if you want to.

In conclusion, unless IATI dedicates the resources to writing XML parsing libraries that support the XSD 1.1 standard or moves away from XML entirely, we must continue to enforce IATI element order as a part of validating mandatory elements in the IATI Validator.