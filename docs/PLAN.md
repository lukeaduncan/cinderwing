# Cinderwing plan

## Goal

A Flutter app for creating and editing FHIR resources. Questionnaires drive
data-entry workflows; FHIR Mapping Language (StructureMap) transforms on
matchbox turn QuestionnaireResponses into resources. A generic editor covers
quick edits to any resource.

## MVP scope

- FHIR R4 only.
- List the Questionnaires on the data server, fill one out, and produce a
  QuestionnaireResponse.
- Send the QuestionnaireResponse to matchbox `StructureMap/$transform`; the map
  is the one named by the Questionnaire's `sdc-questionnaire-targetStructureMap`
  extension.
- POST the resulting Bundle to HAPI FHIR as a transaction.
- Browse and view the resources on HAPI.
- Generic quick edit of any resource (see Key decisions).
- Works with any resource type. The sample content is iHRIS-style health
  workforce data (Practitioner, PractitionerRole, Organization, Location).

## Out of scope (for now)

On the roadmap, later:

- Questionnaire and StructureMap authoring in the app. These are authored
  outside the app for now (FSH or FML files).
- Editing through Questionnaires (resource → QuestionnaireResponse → transform
  again).
- SMART on FHIR auth.
- R5/R6.

Not planned:

- SDC `$extract` (we use `$transform` only).
- Offline use.

## Key decisions

- **License:** Apache 2.0, the norm in the FHIR ecosystem (HAPI FHIR, matchbox).
- **Platforms:** web and Linux first. Android or iOS can be added later.
- **Servers:** two, run with `docker-compose`.
  - **HAPI FHIR (JPA):** stores all data, including the Questionnaires.
  - **matchbox:** transforms only. It holds the StructureMaps (and any logical
    models they use), because `$transform` needs them loaded. Nothing else is
    stored there.
- **Models:** `fhir_r4` typed models. The generic editor works on the
  `toJson()` map and parses it back, because it walks elements by path from the
  StructureDefinition.
- **Page layout (MVP):** taken from the StructureDefinition alone, in snapshot
  element order.
- **FHIR version:** R4 only. We don't build a version abstraction until a
  second version is needed, but we prefer libraries that have an R5/R6 path.
- **View/edit pages:** generated from StructureDefinition snapshots, either
  core R4 or custom profiles (the iHRIS approach). All StructureDefinitions,
  core and custom, are read from HAPI; the core R4 package is loaded into HAPI. The snapshot supplies
  cardinality, types, choice `[x]` elements, backbone elements, and extensions.
  Each FHIR data type gets one view widget and one edit widget.
- **Data types:** the goal is all R4 data types. They're delivered in tiers
  (see Data type support); any type without a widget yet falls back to a raw
  JSON editor for that element, so every resource stays editable.
- **Questionnaire rendering:** to be decided in the spike (increment 5). Either
  reuse a maintained library (`fhir_questionnaire_r4`,
  `fhir_renderer_questionnaire`, `faiadashu`), or build our own renderer on the
  same data-type widgets as the generic editor. The second shares code but is
  more work.

## Data type support

| Tier | Types | When |
|---|---|---|
| 1 | Primitives: boolean, integer, decimal, string, markdown, code, id, uri, url, canonical, oid, uuid, date, dateTime, instant, time, unsignedInt, positiveInt, base64Binary | MVP |
| 2 | Identifier, HumanName, ContactPoint, Address, Period, Coding, CodeableConcept, Reference, Extension (simple values) | MVP |
| 3 | Quantity (and Age, Count, Distance, Duration, Money/SimpleQuantity), Money, Range, Ratio, Attachment, Annotation, Meta, Narrative (view only) | Next |
| 4 | Timing, Dosage, SampledData, Signature, ContactDetail, UsageContext, RelatedArtifact, Expression, DataRequirement, ParameterDefinition, TriggerDefinition, Contributor | Later |
| 5 | Rare R4 types: MarketingStatus, Population, ProdCharacteristic, ProductShelfLife, SubstanceAmount, ElementDefinition | As needed (JSON fallback until then) |

Cross-cutting features that cut across all the types: ValueSet-bound codes
(`$expand`), Reference search, complex extensions, contained resources.

## Open questions

- Custom page layout, post-MVP (field order, sections, labels, related
  resources on one page). Options:
  - **Questionnaire as layout:** items point at resource elements through
    `item.definition`. Standard, and the same renderer could serve forms and
    pages. It's a stretch for view-only pages.
  - **iHRIS-style page resource** (Basic plus extensions): proven, but custom.
  - **GraphDefinition:** standard way to say which related resources belong on
    a page (Practitioner → PractitionerRole → Location). It doesn't cover field
    layout.
  - A likely combination: GraphDefinition for page composition, plus a
    Questionnaire or the profile for each resource's fields.

## Increments

1. [x] Scaffold: Flutter app, license, README, plan.
2. [x] Kickoff: scope and increments.
3. [ ] Servers: `docker-compose` with HAPI and matchbox, plus a sample
   Practitioner Questionnaire and StructureMap. Load `hl7.fhir.r4.core` into
   HAPI so `StructureDefinition?url=…` returns the core definitions. Prove
   `$transform` works with curl before any Dart.
4. [ ] App lists the Questionnaires on HAPI (adds `fhir_r4` and an HTTP client).
5. [ ] Render one Questionnaire into a QuestionnaireResponse (spike: library
   vs. own renderer).
6. [ ] Send the QuestionnaireResponse to `$transform` and show the resulting
   Bundle.
7. [ ] POST the Bundle to HAPI as a transaction; browse and view resources.
8. [ ] View/edit pages generated from a StructureDefinition: tier 1 widgets
   plus the JSON fallback.
9. [ ] Tier 2 widgets.
10. [ ] Custom profiles: generate pages from an iHRIS-style profile on HAPI.

Later: data type tiers 3–5 and the cross-cutting features, editing through
Questionnaires, SMART on FHIR, in-app authoring, R5/R6.
