# Cinderwing

Questionnaire-driven FHIR data entry and editing, built with Flutter.

Cinderwing is a FHIR CRUD editor that uses FHIR Questionnaires to manage
workflows such as initial data import and editing. Completed
QuestionnaireResponses are converted into FHIR resources by FHIR Mapping
Language (StructureMap) transforms hosted on
[matchbox](https://github.com/ahdis/matchbox), a HAPI FHIR–based server.
It's similar in purpose to [iHRIS](https://www.ihris.org/), with forms and
mappings as the main focus.

**Status:** early development. Nothing usable yet.

## How it works

1. A user fills out a Questionnaire.
2. The QuestionnaireResponse is sent to matchbox's `$transform` operation
   with the StructureMap the Questionnaire names.
3. The resulting resources are saved to a
   [HAPI FHIR](https://hapifhir.io/) server as a transaction.
4. Resources can be viewed and quickly edited on pages generated from their
   StructureDefinitions, core R4 or custom profiles.

HAPI FHIR stores all data, Questionnaires, and StructureDefinitions.
Matchbox is used only for transforms. FHIR R4 only, for now.

## Running

Requires Flutter 3.47 or later.

```sh
flutter pub get
flutter run -d chrome   # or: -d linux
flutter test
```

## Roadmap

See [docs/PLAN.md](docs/PLAN.md).

## License

[Apache License 2.0](LICENSE). Copyright 2026 Luke Duncan.
