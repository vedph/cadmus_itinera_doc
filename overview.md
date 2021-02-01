# Itinera Data Architecture Overview

The Cadmus data model for the *Itinera* project includes 5 item types:

- letter
- correspondent
- poetic text
- manuscript
- hand

Of these models, letter, correspondent, and poetic text belong to the epistolography domain; manuscript and hands to the codicology domain.

![items](./images/models-items.png)

## Epistolography

![letter](./images/models-epist.png)

## Codicology

![manuscript](./images/models-ms.png)

The hand is somewhat at the crossing of the epistolographic and codicologic domains, as far as it defines a person, but as related to the handwriting found in a manuscript.

![hand](./images/models-hand.png)

## Items and Parts

Items and parts can be sketched in this picture:

![items and parts](./images/items-n-parts.png)

## Bibliography

Itinera uses an external bibliography, fed by its independent API. This allows for a "top-down" approach, where bibliographic items are entered just once in a standalone database, and then used across all the items and parts of the Cadmus-based system.

Thus, when editing a bibliography part, you are really using two distinct resources:

- the Itinera database, where references to some bibliographic items are stored.
- the bibliographic database, where the full bibliography is stored, to be consumed by the Cadmus system or by any other 3rd party application.

Both these databases are accessed (either for reading or writing) via their own APIs: we thus have 2 different web APIs, one for Cadmus Itinera, and another for bibliography. Both APIs are protected; yet, in this configuration authentication and auditing are shared: so, you just need to login once in the Cadmus app, and use both these APIs.

When editing a bibliography part, you variously browse the bibliographic database to find the desired records to add a reference to, and pick them for that part. For each picked record you can also add an optional tag (used to categorize or group your picks) and an optional note. So, the part just stores each reference (with the bibliographic record ID plus a human-readable label), eventually with a couple of metadata (tag and note); the full bibliographic data is stored elsewhere, in an external resource.

At the same time, when editing the part you can also add new records to this external resource, or edit or delete its existing records. Thus, from the user's point of view the experience is not so different from a bottom-up approach: he just enters bibliographic records as he needs to reference them. Yet, in this way he's not only adding bibliographic references to an item; but also creating a full, standalone bibliographic database by means of progressive additions.

This database has a completely independent modeling, which takes no dependency from the Cadmus architecture; it's a simple bibliographic relational database. Some features of this database are noteworthy:

- the design is focused on the work, having any number of authors and keywords. Each work can also be contained in a container (e.g. a journal, a collection of proceedings, a miscellany book, etc.); the container in turn is just another work, with the only difference that it cannot have a container. This allows to enter bibliographic records representing containers just once, rather than repeating them whenever a contained work needs to be entered.

- the design ensures that every author and work has its own ID, guaranteed to be unique also beyond the scope of the database itself. This is possible because this ID is a GUID, rather than some more scoped value like an arbitrary number. This is very useful for connecting such data to the external world, eventually via LOD.
