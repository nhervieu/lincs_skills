---
name: lincs-validate
description: Validate a CIDOC-CRM data model or RDF output against LINCS application profile requirements. Checks for missing authority URIs, incorrect patterns, namespace issues, and interoperability gaps.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch
argument-hint: [file-or-description-to-validate]
---

# LINCS Compatibility Validator

Validate the provided model, file, or description against LINCS requirements.

## Target: $ARGUMENTS

## Validation Procedure

Read or analyze the target, then check against each category below. Report findings as PASS, WARN, or FAIL with specific remediation steps.

### Category 1: Namespace Correctness

Check that:
- Core CRM classes (E21, E53, E55, E7, etc.) use `crm:` prefix (`http://www.cidoc-crm.org/cidoc-crm/`)
- CRMgeo classes (E93, E94) and properties (P166, P164, P161, P132, P134) use `crmgeo:` prefix — NOT `crm:`
- CRMdig classes (D1_Digital_Object) use `crmdig:` prefix
- OA Annotation classes use `oa:` prefix
- No invented property names that aren't in any CRM spec (e.g., `PLACE_LINEAGE`, `SPLIT_FROM`)

**Common error**: Treating CRMgeo properties as core CRM. P166, P164, P161 are CRMgeo.

### Category 2: Authority URIs

Check that:
- Places have GeoNames URIs (`http://sws.geonames.org/{id}/`)
- People have VIAF (`http://viaf.org/viaf/{id}`) or Wikidata URIs
- Types/concepts have Wikidata QIDs or LINCS vocabulary URIs
- Every entity NOT in an external authority has a LINCS-minted URI

**FAIL if**: Only internal IDs (e.g., `PLACE_ON142032`) with no external authority link.

### Category 3: Required Properties

Check that every entity has:
- `rdfs:label` (human-readable label, language-tagged)
- `rdf:type` (explicit class declaration)

Check that every named entity has:
- `crm:P1_is_identified_by` → `crm:E33_E41_Linguistic_Appellation` or `crm:E42_Identifier`
- String value via `crm:P190_has_symbolic_content`

### Category 4: Temporal Modeling

Check that:
- Dates are always on `E52_Time-Span` nodes, not directly on entities
- Time-spans have `P82_at_some_time_within` (human string), `P82a_begin_of_the_begin`, `P82b_end_of_the_end`
- `P82a` and `P82b` use `^^xsd:dateTime` with full ISO 8601 format (e.g., `"1871-01-01T00:00:00"^^xsd:dateTime`)
- Events connect to time-spans via `P4_has_time-span`
- `E4_Period` is used only for named historical periods, not as a general temporal index

**FAIL if**: `P82a` or `P82b` values are typed as `xsd:string` or `xsd:date` instead of `xsd:dateTime`. Example of INVALID: `"1871-01-01"^^xsd:string`. Must be: `"1871-01-01T00:00:00"^^xsd:dateTime`.

**WARN if**: E4_Period used as a census-year grouping node (valid CRM, but not LINCS-compatible)

### Category 5: Spatial Modeling

Check that:
- Places use `P168_place_is_defined_by` with GeoSPARQL WKT literal: `"<http://www.opengis.net/def/crs/EPSG/0/4326> POINT(lon lat)"^^geosparql:wktLiteral`
- Coordinate order is **longitude latitude** (x y), NOT latitude longitude
- Spatial hierarchy uses `P89_falls_within` (static, no temporal qualifier)
- For time-varying hierarchy, route through E93_Presence → P10_falls_within (both subject and object must be `E93_Presence` or `E4_Period`, never bare `E53_Place`)
- P89 does NOT carry properties (no `during_period`, `shared_border_length`)

**FAIL if**: Coordinates typed as `^^xsd:string` instead of `^^geosparql:wktLiteral` (breaks spatial queries in GeoSPARQL-capable triplestores).
**FAIL if**: Relationship properties on P89 or P122 (not valid in RDF).
**WARN if**: `^^xsd:string` used for coordinates in a deployment without GeoSPARQL support (acceptable but not ideal).

### Category 6: Event-Centric Patterns

Check that:
- Occupations are `E7_Activity` typed `event:OccupationEvent` (not direct type on person)
- Birth uses `E67_Birth` with `P98_brought_into_life`
- Death uses `E69_Death` with `P100_was_death_of`
- Marriage uses `E85_Joining` + `E74_Group` (not a direct relationship between people)
- Group membership uses `P107_has_current_or_former_member`
- Gender assigned via `E13_Attribute_Assignment` (not a direct property)

### Category 7: Measurement Pattern (Census/Statistical Data)

Check that:
- Measurements use `E16_Measurement` (not `E13_Attribute_Assignment`)
- Each measurement connects: `E16 → P39_measured → target`, `E16 → P40_observed_dimension → E54`, `E54 → P91_has_unit → E58`
- `E54_Dimension` has `P90_has_value` with a numeric XSD datatype (`^^xsd:integer` or `^^xsd:decimal`)
- `E54_Dimension` has `P91_has_unit` pointing to a valid `E58_Measurement_Unit` node (not a bare string)
- Variable classification via `E16 → P2_has_type → E55_Type`
- Provenance via `E73 → P70_documents → E16`

**FAIL if**: `E54_Dimension` is missing `P90_has_value` (measurement has no numeric result).
**FAIL if**: `E54_Dimension` is missing `P91_has_unit` (measurement has no unit — renders data uninterpretable).
**FAIL if**: `P90_has_value` uses `^^xsd:string` instead of a numeric datatype.
**PASS if**: Measurement chain is complete with provenance and correctly typed values.

### Category 8: Document Modeling

Check that:
- Physical originals use `crm:E73_Information_Object`
- Digital surrogates use `crmdig:D1_Digital_Object`
- Digital-to-physical link uses `P130_shows_features_of` or `P130i_features_are_also_found_on`
- Subject link uses `P129_is_about`
- Mention link uses `P67_refers_to`
- Text annotations use OA Web Annotation model (`oa:Annotation`, `oa:hasTarget`, `oa:hasBody`)

### Category 9: Neo4j ↔ RDF Translation Issues

If validating a Neo4j property graph, flag:
- **Relationship properties**: Any edge with attributes (not expressible in basic RDF)
- **Custom relationship types**: Any type not in CRM/CRMgeo/CRMdig specs
- **Missing namespaces**: Node labels without proper URI prefixes
- **Bare string properties**: Values that should route through E33_E41 or E42 patterns
- **Multiple labels**: Neo4j multi-label nodes need explicit `rdf:type` for each class

### Category 10: Dangling URI Check

If validating generated RDF/Turtle output, check for **referential integrity**:

- Every local URI (using `base:` or project prefix) that appears in **object position** of a triple must also appear as a **subject** with an `rdf:type` declaration somewhere in the output.
- External authority URIs (`geonames:`, `viaf:`, `wikidata:`, `aat:`, `lexvo:`) are exempt — they are defined externally.
- `E52_Time-Span` and `E54_Dimension` nodes created inline with blank node syntax (`[ a crm:E52_Time-Span ; ... ]`) are exempt.

**Example of a dangling URI (FAIL)**:
```turtle
# This measurement references a presence that is never defined
base:MEAS_1 a crm:E16_Measurement ;
    crm:P39_measured base:ON082003_1871 .  # <-- DANGLING: base:ON082003_1871 has no rdf:type triple
```

**Corrected**:
```turtle
base:MEAS_1 a crm:E16_Measurement ;
    crm:P39_measured base:ON082003_1871 .

base:ON082003_1871 a crmgeo:E93_Presence ;   # <-- Now defined
    rdfs:label "Westmeath presence (1871)"@en .
```

**FAIL if**: Any local URI referenced in object position has no `rdf:type` declaration as a subject.
**WARN if**: A local URI is defined with `rdf:type` but is missing `rdfs:label`.

### Output Format

```
## LINCS Validation Report

### Summary
- PASS: N checks
- WARN: N checks
- FAIL: N checks

### Findings

#### [PASS/WARN/FAIL] Category Name
Description of finding.
**Remediation**: Specific steps to fix.
```

Prioritize FAIL items first, then WARN. For each finding, provide a concrete Turtle/Cypher example showing the current state and the corrected version.

**Categories**: 1. Namespace Correctness, 2. Authority URIs, 3. Required Properties, 4. Temporal Modeling, 5. Spatial Modeling, 6. Event-Centric Patterns, 7. Measurement Pattern, 8. Document Modeling, 9. Neo4j ↔ RDF Translation, 10. Dangling URI Check
