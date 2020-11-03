# Cadmus Itinera Models

In what follows I adopt these conventions:

- each type name is followed by its type in parentheses; if the type is drawn from a taxonomy, the word "thesaurus" is added (this is the internal name of a taxonomy in Cadmus).
- the type name marked by asterisk means that it requires a value.
- data types followed by `[]` mean an array, i.e. a list of such data. For instance, `string[]` = a list of `string`'s. For strings, `MD` means Markdown, i.e. a lightweight markup system providing essential formatting; this practically means that you can add formatting to the text, like e.g. italic or bold.
- custom types are defined with a name (e.g. `HistoricalDate`), and described only the first time they are introduced in the document; if they are types already present in the Cadmus "stock" models, there is no description but a link to the documentation.

In general, this architecture is targeted to modular content creation. The usual flow is creating highly structured and modular data using Cadmus, and then automatically build a relational DB from them, optimized for the search types we're going to support, plus LOD "extractions" for connecting the relevant parts of our data to the semantic web.

There are 5 items in the Cadmus DB:

- letters: data about letters (not their text);
- correspondents of the letters;
- poetic texts related to the letters;
- manuscripts;
- manuscript hands.

About 30 part types are variously assigned to those items.

## Common Models

Some commonly reused (sub-part) models are listed here:

- `DocReference`: a literary citation:

  - `tag` (`string`; thesaurus)
  - `author`\* (`string`; thesaurus of authors/works like that illustrated in the general purpose [quotations fragment](https://github.com/vedph/cadmus_core/wiki/Philology-Parts#quotations)
  - `work`\* (`string`; thesaurus as above)
  - `location` (`string`; e.g. "2.34")
  - `note` (`string`, 500)

- `PersonName`: a personal name:

  - `language`\* (`string` = code from [ISO 639-3](https://en.wikipedia.org/wiki/ISO_639-3), thesaurus)
  - `tag` (`string`, optionally from thesaurus): optional tag used to group names, typically used when a person has several names.
  - `parts`\* (`PersonNamePart[]`):
    - `type`\* (`string`, thesaurus): e.g. first name, last name, etc. Types are not unique in a name: for instance, you might have a person with 2 first names.
    - `value`\* (`string`)

- `CitedPerson`: a person cited in a literary source:

  - `name` (`PersonName`)
  - `ids` (`DecoratedId[]`)

- `MsLocation`: a location in a manuscript:

  - `n`\* (`number`): sheet number
  - `v` (`bool`): recto (false)/verso (true)
  - `l` (`number`): line number

- `DecoratedCount`:

  - `id`
  - `value`
  - `note`

- `DecoratedId`:

  - `id`
  - `rank`: a numeric rank to sort identifications in their order of probability. For a single identification, just leave the rank value equal (to 0). Otherwise, use 1=highest probability, 2=lower than 1, and so on.
  - `tag`
  - `sources` (`DocReference[]`)

- `Chronotope`:

  - `tag`\* (`string`)
  - `place`\* (`string`)
  - `date`\* (`HistoricalDate`)
  - `textDate` (`string`): the attested text form of the date. It might also be different from the reconstructed one.
  - `sources` (`DocReference[]`)

Note that here and in other parts the _place just corresponds to a conventional name_ (e.g. `Napoli`). This is not a full-fledged geographic model, but just a name; including data like coordinates here would not be an option, because this would imply duplicating (i.e. re-entering and re-storing) them wherever the same name occurs. Rather, coordinates will be later automatically deduced from the name using a geolocation service like Google. We must thus ensure that the place name can be located without issues (eventually by enriching our geolocation request with restrictions, e.g. "in Italy" or the like).

- `EpistAttachment`:

  - `type`\* (`string`; thesaurus: manuscript, work)
  - `name`\* (`string`; if work, use thesaurus of works): not specified if unindentified.
  - `portion` (`string`): the specification of the portion of the attachment. For instance, if the attachment is a work, this specifies the work's location (e.g. Aen. 1.13-2.26).
  - `note`: you can use this when the object is not identified to provide an informal description.

Existing sub-models:

- `PhysicalDimension`:

  - `tag` (`string`)
  - `value`\* (`number`, decimal)
  - `unit`\* (`string`, thesaurus)

- `PhysicalSize`:
  - `tag` (`string`)
  - `w` (`PhysicalDimension`)
  - `h` (`PhysicalDimension`)
  - `d` (`PhysicalDimension`)
  - `note` (`string`)

At least 1 among `w`/`h`/`d` should be specified.

## Letter Item

The letter item has 7 part types (2 generic):

- `LetterInfoPart`: essential metadata about a letter (`it.vedph.itinera.letter-info`).
- `ChronotopicsPart`: date and place (`it.vedph.itinera.chronotopics`).
- `DocReferencesPart`: references to documents (`it.vedph.itinera.doc-references`).
- `AttachmentsPart`: letter's attachments (`it.vedph.itinera.attachments`).
- `CitedPersonsPart`: persons cited in the letter (`it.vedph.itinera.cited-persons`).
- `BibliographyPart`: bibliography (`it.vedph.bibliography`).
- `NotePart`: general purpose note (`it.vedph.note`).

## Correspondent Item

The correspondent item has 8 part types (2 generic):

- `PersonPart`: biographic data (`it.vedph.itinera.person`).
- `PersonEventsPart`: events in a person's life (`it.vedph.itinera.person-events`); biographic events and meetings are separated into two such parts, with different roles (`ebio` and `emet`): TODO: or just one part??
- `CorrDedicationsPart`: dedications involving the correspondent, i.e. by the correspondent to the reference author, or vice-versa (`it.vedph.itinera.corr-dedications`).
- `CorrPseudonymsPart`: pseudonyms for the correspondent used by the reference author, or vice-versa (`it.vedph.itinera.corr-pseudonyms`).
- `CorrExchangesPart`: exchanges of works, manuscripts or other objects involving the correspondent (`it.vedph.itinera.corr-exchanges`).
- `DocReferencesPart`: references made by author; references made by correspondent. A role distinguishes the two usages (`it.vedph.itinera.doc-references` with roles `auth` and `corr`).
- `BibliographyPart`: bibliography (`it.vedph.bibliography`).
- `NotePart`: general purpose note (`it.vedph.note`).

Note about _correspondents of correspondents_: given that their model is equal to or a subset of the author's correspondents, it makes more sense to just treat them as author's correspondents. That is, instead of having a _nested_ list of correspondents of correspondents inside each correspondent (thus building a recursive nesting), we would just keep a plain, _flat_ list of correspondents. Some of them would be correspondents of the reference author (here Petrarch); others would be correspondents of the correspondent X.

We can then just use these models, adding a _group ID_ to the correspondent item, with a value equal to their reference correspondent ID.

Each item in Cadmus can have a group ID for general purpose grouping, e.g. grouping a set of texts belonging to the same work (fragmented to be more manageable); a set of lemmata under the same letter in a dictionary; etc. In this case, we would e.g. have the correspondent with `id`=`barbato`, and all his correspondents would be stored just like any other correspondent. The only difference is that their group ID would be `barbato`. We can thus keep a much simpler "flat" list of correspondents, each with a reference correspondent, arbitrarily chosen according to the project's purposes. In this case, the default correspondent is Petrarch, with no group ID; correspondents of correspondents have the group ID of their reference correspondent. The advantages are:

- avoid repeating parts of the correspondents model inside itself, with a redundant and potentially recursive structure.
- keep the data architecture flat, as a single list of correspondents, thus easing indexing and searching.
- allow the full data model of correspondents also for correspondents of correspondents, even though usually they will just use a subset of them. This allows for future or occasional expansions where required, without having to change the nested model.

## Poetic Text Item

The poetic text item has 6 parts (2 generic):

- `PoeticTextInfoPart`: essential metadata about a text.
- `ChronotopicsPart`: date and place (`it.vedph.itinera.chronotopics`).
- `AttachmentsPart`: letter's attachments (`it.vedph.itinera.attachments`).
- `CitedPersonsPart`: persons cited in the text (`it.vedph.itinera.cited-persons`).
- `BibliographyPart`: bibliography (`it.vedph.bibliography`).
- `NotePart`: general purpose note (`it.vedph.note`).

## Manuscript Item

The manuscript item has 17 parts (3 generic):

- `MsSignaturesPart`: signature(s) (`it.vedph.itinera.ms-signatures`).
- `MsCompositionPart`: composition (`it.vedph.itinera.ms-composition`).
- `MsPlacePart`: place of origin (`it.vedph.itinera.ms-place`).
- `MsMaterialDscPart`: material description (`it.vedph.itinera.ms-material-dsc`).
- `MsDimensionsPart`: dimensions (`it.vedph.itinera.ms-dimensions`).
- `MsWatermarksPart`: watermarks (`it.vedph.itinera.ms-watermarks`).
- `MsNumberingsPart`: numberings (`it.vedph.itinera.ms-numberings`).
- `MsQuiresPart`: quires (`it.vedph.itinera.ms-quires`).
- `MsCatchwordsPart`: catchwords (`it.vedph.itinera.ms-catchwords`).
- `MsHandsPart`: hands (`it.vedph.itinera.ms-hands`).
- `MsDecorationsPart`: decorations (`it.vedph.itinera.ms-decorations`).
- `MsBindingPart`: binding (`it.vedph.itinera.ms-binding`).
- `MsHistoryPart`: history of the MS (`it.vedph.itinera.ms-history`).
- `MsPoemRangesPart`: poem's ranges in a manuscript (`it.vedph.itinera.ms-poem-ranges`).
- `BibliographyPart`: bibliography (`it.vedph.bibliography`).
- `NotePart`: general purpose note (`it.vedph.note`).
- `HistoricalDatePart`: datatation (`it.vedph.historical-date`).

## Hand Item

The person's hand part has 4 parts (2 generic):

- `PersonPart`: biographic data (`it.vedph.itinera.person`).
- `PersonHand`: hand's description (`it.vedph.itinera.person-hand`).
- `BibliographyPart`: bibliography (`it.vedph.bibliography`).
- `NotePart`: general purpose note (`it.vedph.note`).

## Parts

### PersonPart

This contains the ID and biographic data of any person (the part role ID makes the distinction among different groups explicit). It can also represent any other generic actor, e.g. a group of persons ("Celestini").

- `personId`\* (`string`, eventually from thesaurus): this is an internal human-readable ID for the correspondent, e.g. `barbato`, `flavio-2`, etc. It can used as a facility for referring to the correspondent across parts, without recurring to the machine-related item ID. Thus, it must be unique in the database.
- `externalIds` (`string[]`): optional IDs in external resources which can be mapped to this correspondent.
- `names`\* (`PersonName[]`): at least 1 name.
- `sex` (`string`: `m`, `f`, or null if unknown)
- `birthDate` (`HistoricalDate`)
- `birthPlace` (`string`)
- `deathDate` (`HistoricalDate`)
- `deathPlace` (`string`)
- `bio` (`string`, MD, 6000)

### PersonEventsPart

Any event relevant for the biography of a person, including his works. This part is used, with 2 different roles, for both the correspondent's biography and his meetings with the reference author.

- `events` (`BioEvent[]`):
  - `type`\* (`string`; thesaurus)
  - `date` (`HistoricalDate`)
  - `places` (`string[]`)
  - `description` (`string`, MD, 1000)
  - `sources`\* (`DocReference[]`)
  - `participants` (`DecoratedId[]`)
  - `work` (`string`)
  - `rank` (`number`)
  - `isWorkLost` (`boolean`)
  - `externalIds` (`string[]`)

### CorrDedicationsPart

Dedications made by the author to the correspondent, or vice-versa.

- `dedications` (`LitDedication[]`):
  - `title`\* (`string`)
  - `date` (`HistoricalDate`)
  - `dateSent` (`HistoricalDate`)
  - `isByAuthor` (`boolean`): the dedication is by the author to the correspondent, or vice-versa. Given that the focus is on the author, the dedication by the author to the correspondent can be the marked case of the binary opposition.
  - `sources` (`DocReference[]`)

### CorrPseudonymsPart

Pseudonyms used by the reference author to address the correspondent, or vice-versa.

- `pseudonyms` (`CorrPseudonym[]`):
  - `language`\* (`string`, ISO639-3)
  - `value`\* (`string`)
  - `isAuthor` (`boolean`): true if the alias targets the reference author; false if it targets the correspondent.
  - `sources` (`DocReference[]`)

### CorrExchangesPart

Exchanges from the reference author to the correspondent, or vice-versa.

- `exchanges` (`CorrExchange[]`) of works/manuscripts with author:
  - `isDubious` (`boolean`)
  - `isIndirect` (`boolean`)
  - `isFromParticipant` (`boolean`): the "direction" of the exchange: true when the attachment is coming from 1 or more of the participants; false when it is coming from the correspondent (represented by the container item) to the participants.
  - `from`\* (`Chronotope`)
  - `to`\* (`Chronotope`)
  - `participants` (`DecoratedId[]`):
    - `id` (`string`): the participant ID. This should refer to a person part.
    - `tag` (`string`, thesaurus): the role of the participant (e.g. destinatario, latore...).
  - `sources` (`DocReference[]`)
  - `attachments` (`EpistAttachment[]`)

### DocReferencesPart

A list of document references (literary citations, short bibliographic references, etc.).

- `references` (`DocReference[]`):
  - `tag`: any classification tag meaningful in the data context.
  - `author`: the author ID (e.g. `Hom.`). For archive documents, it can be a constant reserved ID. For bibliographic references, it's the modern author ID.
  - `work`: the work ID (e.g. `Il.`). For archive documents, it can be the archive name. For bibliographic references, it's the modern work's title ID.
  - `location`: the work's location (e.g. `12.34`). For archive documents, it can be their location in the archive (e.g. a signature). For bibliographic references, it's usually a page number or other means of locating some passage.
  - `note`: a generic annotation.

### ChronotopicsPart

Spatial and temporal coordinates for a letter or a related poetic text.

- chronotopes (`Chronotope[]`)

### LetterInfoPart

- `language`\* (`string` = code from [ISO 639-3](https://en.wikipedia.org/wiki/ISO_639-3), thesaurus)
- `subject`\* (`string`, MD, 500)
- `heading` (`string`): the heading found in the manuscript text.
- `note` (`string`, MD, max 1000): a general-purpose free text note to hold "varia".

### AttachmentsPart

Attachments linked to a letter or poetic text.

- `attachments` (`EpistAttachment[]`)

### CitedPersonsPart

- `persons` (`CitedPerson[]`):
  - `ids` (`DecoratedId[]`)
  - `sources` (`DocReference[]`)
  - `name` (`PersonName`)

### PoeticTextInfoPart

- `language`\* (`string` = code from [ISO 639-3](https://en.wikipedia.org/wiki/ISO_639-3), thesaurus)
- `subject`\* (`string`, MD, 500)
- `metre` (`string`, thesaurus)
- `authors` (`CitedPerson[]`)
- `related` (`DocReference[]`): related text passages.

Note: the item's title is the poetic text's title, so there is no need to duplicate it in the info part.

### PersonHandPart

- `type` (`string`, thesaurus)
- `job` (`string`, thesaurus)
- `description` (`string`, MD, 2000)
- `initials` (`string`)
- `corrections` (`string`)
- `punctuation` (`string`)
- `abbreviations` (`string`, MD, 1000)
- `imageIds` (`string[]`)
- `signs` (`MsHandSign`[]): description of any relevant graphical sign, whether it's a letter or not (the type is specified in `type`):
  - `id`\* (`string`): this must be a unique, arbitrarily chosen string for this sign
  - `type`\* (`string`, thesaurus)
  - `description` (`string`, MD, 1000)
  - `imageId` (`string`): this is an ID representing the prefix for all the images representing that sign; e.g. if it is `ae`, we would expect any number of image resources named after it plus a conventional numbering, like `ae00001`, `ae00002`, etc.

### MsSignaturesPart

- signatures (`MsSignature[]`):
  - `tag` (string)
  - `city`\* (`string`)
  - `library`\* (`string`)
  - `fund` (`string`)
  - `location`\* (`string`)

At least 1 must be present. The signature with empty tag is the default (current) signature. Other signatures may be added for historical reasons, and should have a tag.

### MsCompositionPart

- `sheetCount`\* (`number`)
- `guardSheetCount` (`number`)
- `guardSheets` (`MsGuardSheet[]`):
  - `isBack` (`boolean`)
  - `material` (`string`, thesaurus)
  - `location` (`MsLocation`)
  - `date` (`HistoricalDate`)
  - `note` (`string`)
- `sections` (`MsSection[]`):
  - `tag` (`string`): any arbitrarily chosen ID used to group sections together.
  - `label`\* (`string`)
  - `start`\* (`MsLocation`)
  - `end`\* (`MsLocation`)
  - `date`\* (`HistoricalDate`)

### MsPlacePart

Place of origin (provenance is in `MsHistoryPart`).

- `area`\* (`string`, `thesaurus`)
- `address` (`string`): this contains any number of components separated by comma, like in geocoding addresses, whereas the area is the top level component.
- `city` (`string`)
- `site` (`string`, `thesaurus`): e.g. "St. Paul monastry".
- `subscriber` (`string`): subscriber ID (human-readable, arbitrary string)
- `subscriptionLoc` (`MsLocation`)
- `sources` (`DocReference[]`): modern bibliography references.

### MsMaterialDscPart

- `material`\* (`string`, thesaurus)
- `format`\* (`string`, thesaurus)
- `state`\* (`string`, thesaurus)
- `stateNote` (`string`, 500)
- `palimpsests` (`MsPalimpsest[]`):
  - `location`\* (`MsLocation`)
  - `date` (`HistoricalDate`)
  - `note` (`string`)

### MsDimensionsPart

- `sample`\* (`MsLocation`): the sheet used as the sample for taking measurements.
- `dimensions` (`PhysicalDimension[]`): any number of measurements taken for any kind of measurable thing in the manuscript.
- `counts` (`DecoratedCount[]`)

The `counts` property allows entering any number of counts with different levels of precision. For instance, you might have `rowMinCount`, `rowMaxCount`, `lineCount`, `approxLineCount`, `lineMinCount`, `lineMaxCount`, `prickCount`, etc. It also allows descriptions for properties like columns, direction, blanks, ruling, execution, etc., eventually with a count (which might represent an average, or the most frequent value, etc.).

### MsWatermarksPart

- `watermarks` (`MsWatermark[]`):
  - `subject`\* (`string`, thesaurus)
  - `similarityRank` (`number`): a rank for the similarity of the watermark described here to the paradigmatic watermark model; the higher the number, the less similar is the watermark.
  - `description` (`string`): a text description for this watermark instance.
  - `place` (`string`)
  - `date` (`HistoricalDate`)
  - `externalIds` (`string[]`)

### MsNumberingsPart

- `numberings` (`MsNumbering[]`):
  - `isMain` (`boolean`)
  - `era`\* (`string`, thesaurus: one from "coeva", "antica", "moderna", "recente")
  - `system`\* (`string`, thesaurus)
  - `technique`\* (`string`, thesaurus)
  - `century`\* (`number`)
  - `position` (`string`, thesaurus)
  - `issues` (`string`)

### MsQuiresPart

- `quires` (`MsQuire[]`):
  - `isMain` (`boolean`)
  - `startNr`\* (`number`)
  - `endNr`\* (`number`)
  - `sheetCount`\* (`number`)
  - `sheetDelta` (`number`, negative or positive)
  - `note` (`string`)

Note: allow users to enter collations using formulas like `1-3^4-1`. In this syntax we have any number of tokens separated by whitespace. Each token has the syntax `N-N^N±N {note}` where `-N` and `±N` are optional:

- `N` or `N-N` is the gatherings range (=`startNr` and `endNr`; when only `N` is specified, both are assumed to be equal);
- `^` introduces the sheets count (`sheetCount`);
- `±N` adds an exceptionally missing or additional sheet count (`sheetDelta`).
- `note` is an optional short note. It should not include braces.

### MsCatchwordsPart

- `catchwords` (`MsCatchword[]`):
  - `position`\* (`string`, thesaurus)
  - `isVertical` (`boolean`)
  - `decoration` (`string`)
  - `register` (`string`): numerazioni/segnature a registro
  - `note` (`string`)

### MsHandsPart

- `hands` (`MsHandInstance[]`):
  - `id`\* (`string`)
  - `idReason`\* (`string`, thesaurus)
  - `start` (`MsLocation`)
  - `end` (`MsLocation`)
  - `extentNote` (`string`)
  - `rubrications` (`MsRubrication[]`):
    - `location`\* (`MsLocation`)
    - `type`\* (`string`, thesaurus)
    - `description` (`string`)
    - `issues` (`string`)
  - `subscription` (`MsSubscription`):
    - `location`\* (`MsLocation`)
    - `language`\* (`string` = code from [ISO 639-3](https://en.wikipedia.org/wiki/ISO_639-3), thesaurus)
    - `text` (`string`)

### MsDecorationsPart

- `decorations` (`MsDecoration[]`):
  - `type`\* (`string`, thesaurus)
  - `subject`\* (`string`)
  - `colors`\* (`string[]`; 0-N picks from a thesaurus)
  - `layout`\* (`string`, thesaurus)
  - `tool`\* (`string`, thesaurus)
  - `start` (`MsLocation`)
  - `end` (`MsLocation`)
  - `position` (`string`, thesaurus)
  - `size` (`PhysicalSize`)
  - `description` (`string`, MD, 1000)
  - `textRelation` (`string`, 1000)
  - `guideLetters` (`MsGuideLetter[]`):
    - `position`\* (`string`, thesaurus)
    - `morphology` (`string`)
  - `imageId` (`string`): this is an ID representing the prefix for all the images representing the decoration; e.g. if it is `ae`, we would expect any number of image resources named after it plus a conventional numbering, like `ae00001`, `ae00002`, etc.
  - `artist` (`MsDecorationArtist`):
    - `type`\* (`string`, thesaurus)
    - `id`\* (`string`): inner ID
    - `name`\* (`string`)
    - `note` (`string`)
    - `sources` (`DocReference[]`)

### MsBindingPart

- `century`\* (`number`)
- `description`\* (`string`, MD, 5000)
- `coverMaterial`\* (`string`, thesaurus)
- `supportMaterial`\* (`string`, thesaurus)
- `size` (`PhysicalSize`)

### MsContentsPart

- `contents` (`MsContent[]`):
  - `author` (`string`)
  - `claimedAuthor` (`string`)
  - `work`\* (`string`)
  - `start` (`MsLocation`)
  - `end` (`MsLocation`)
  - `state` (`string`, thesaurus)
  - `note` (`string`)
  - `units` (`MsContentUnit[]`):
    - `label`\* (string)
    - `incipit` (string)
    - `explicit` (string)

### MsContentLociPart

Loci critici.

- `loci` (`MsContentLocus[]`)
  - `citation`\* (`string`)
  - `text`\* (`string`)

### MsHistoryPart

- `provenances`\* (GeoAddress[], at least 1; in chronological order):
  - area\* (`string`)
  - address (`string`)
- `history`\* (`string`, MD, 5000)
- `persons` (`MsHistoryPerson[]`):
  - `id` (`string`)
  - `role` (`string`, thesaurus)
  - `name`\* (`PersonName`)
  - `date` (`HistoricalDate`)
  - `note` (`string`, 1000)
  - `externalIds` (`string[]`)
  - `sources` (`DocReference[]`)
- `annotations` (`MsAnnotation[]`):
  - `language`\* (`string` = code from [ISO 639-3](https://en.wikipedia.org/wiki/ISO_639-3), thesaurus)
  - `type`\* (`string`, thesaurus)
  - `text`\* (`string`, 1000)
  - `start` (`MsLocation`)
  - `end` (`MsLocation`)
  - `personId` (`string`)
  - `sources` (`DocReference[]`)
- `restorations` (`MsRestoration[]`):
  - `type`\* (`string`, thesaurus)
  - `place` (`string`)
  - `date` (`HistoricalDate`)
  - `note` (`string`)
  - `personId` (`string`)
  - `sources` (`DocReference[]`)

### MsPoemRangesPart

This part defines the sequence of poems or other types of numbered compositions in a manuscript. This is used to compare the sequence of poems in different mss of Petrarch's RVF.

- `tag` (`string`)
- `ranges`\* (`AlnumRange[]`):
  - `a`\* (`string`): the start "number".
  - `b` (`string`): the end number unless equal to the start "number".
- `note` (`string`)

In ranges, values for `a` and `b` are strings representing "alphanumerics", i.e. either numbers (a string with only digits), or at least 1 digit followed by any combination of digit or non-digit characters. These alphanumerics are sorted according to the following criteria:

- an alphanumeric with only digits (e.g. `12`) is sorted by its numeric value.
- an alphanumeric with non-digit characters (e.g. `12a`, `12b1`, etc.) is sorted first by its numeric value, and then alphabetically by its string suffix.

Ranges can be expressed only for alphanumerics with digits only, as they would be ambiguous in other cases: e.g. 12-15 means 12, 13, 14, 15; but 12-15a could be interpreted in a number of different ways, as we have no predefined convention for the non-digits part. For instance, a valid range could be `1-13, 15-19, 20a, 21-36, 38, 50-67`.
