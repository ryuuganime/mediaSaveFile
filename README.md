<!-- markdownlint-disable MD033 -->
<!-- cSpell:words headered headerless HJSON ryuuganime mediahistory savefile Anony substat substatus kitsu -->
<!-- omit in toc -->
# Ryuuganime Media SaveFile Format

Standardization attempt and Proof of Concept of cross tracking site services save file/exported list for media entries library in JSON, YAML, HJSON, PowerShell `psd1`, and CSON format.

> **WARNING**
>
> This documentation is still in beta. The schema and the format may change in the future.

<!-- omit in toc -->
## Table of Contents

* [What is this?](#what-is-this)
* [Why do we need this?](#why-do-we-need-this)
* [How does it work?](#how-does-it-work)
* [Which media type is supported?](#which-media-type-is-supported)
  * [Why is the schema not optimized for other media types?](#why-is-the-schema-not-optimized-for-other-media-types)
* [How do I use it?](#how-do-i-use-it)
* [Is there a way to store progress history for each record?](#is-there-a-way-to-store-progress-history-for-each-record)
* [Ryuuganime Media SaveFile Format Documentation](#ryuuganime-media-savefile-format-documentation)
  * [Code formatting, styling, and file name](#code-formatting-styling-and-file-name)
  * [SaveFile Format](#savefile-format)
    * [Headered](#headered)
    * [Headerless](#headerless)
  * [Headered Metadata (`/metadata`)](#headered-metadata-metadata)
    * [`version`](#version)
    * [`name`](#name)
    * [`mediaType`](#mediatype)
    * [`description`](#description)
    * [`service`](#service)
      * [`service/name`](#servicename)
      * [`service/uri`](#serviceuri)
      * [`service/version`](#serviceversion)
      * [`service/*`](#service-1)
    * [`exported`](#exported)
      * [`exported/date`](#exporteddate)
      * [`exported/*`](#exported-1)
    * [`user`](#user)
      * [`user/id`](#userid)
      * [`user/name`](#username)
      * [`user/uri`](#useruri)
      * [`user/*`](#user-1)
  * [Entries (`/entries[]`)](#entries-entries)
  * [Definitions](#definitions)
    * [Enum: MediaType](#enum-mediatype)
    * [Int: Progress](#int-progress)
* [Usage Example](#usage-example)
  * [JSON Headerless](#json-headerless)
  * [PSD1 Headered](#psd1-headered)
  * [YAML Headered](#yaml-headered)

## What is this?

This is an attempt to standardize the save file format for media entries in JSON and YAML. This is a proof of concept and is not yet finalized. This is also not a standardization of the API, but rather the save file format.

The repo also acts as a schema holder for the SaveFile format.

## Why do we need this?

There are many sites that offer a way to track media entries. However, there are no standardized save file formats for these entries. This makes it difficult to transfer your list from one site to another.

We also believe that user data must be readable and editable by the user, and not being locked by undocumented properties and formats.

## How does it work?

Media entries are saved either in JSON/C, YAML, [HJSON](https://hjson.github.io/), PowerShell `psd1`, and/or CSON format. Usage of these formats are not mutually exclusive, and the user can choose which format to use, but we recommend using YAML for readability.

However, the usage of XML is not recommended, as it is not human-readable, and is not supported by the schema by default.

The file itself can be saved as an object ([`headered`](#headered)) or as an array ([`headerless`](#headerless)).

## Which media type is supported?

SaveFile format is virtually media type agnostic/independent.

However, the schema is mostly optimized for anime and manga progress system, where the media entry is divided into episodes/chapters and volumes, and progress is tracked by the number of episodes/chapters and volumes consumed, not by which episode/chapter and volume watched/read.

For example:

```yaml
- id: 123456
  mediaType: animation
  current:
    chapter: 4
  upstream:
    chapter: 12
```

In this example, the user is already watching the 4th episode of the anime with ID `123456`, and the upstream service has 12 episodes.

For other media types, the schema can be modified to fit the media type.

### Why is the schema not optimized for other media types?

Each media type has its own progress system, and it is not possible to create a schema that can fit all media types, as seen in the example above.

However, the schema allows for additional allowed properties, so it is possible to add properties that are specific to the media type.

For example, TV and animation media type can use `//chapter` as episodes marker and `//volume` as seasons marker, while other media that can not determined by episodes and seasons (such `game`) can use `//progress` as percentile marker.

## How do I use it?

SaveFile format must follow the schema defined in the `schema` folder, and the documentation of usage will be explained in this README.

## Is there a way to store progress history for each record?

No, SaveFile format is not designed to store progress history. However, it is possible to store progress history in a separate file as `<service>.mediahistory.log` and readable as a TSV file with `\t` as delimiter.

Below is a format example of the history log:

<!-- markdownlint-disable MD010 -->
<!-- cSpell:disable -->
```log
# DATE FORMAT				STATUS	KIND		ID		TITLE	EVENTTYPE	SUBSTAT	AFTER MSG	BEFORE		MESSAGE
[2022-12-20T17:50:00+07:00]	EVENT	ANIMATION	123456	"HELLO"	PROGEPISODE	CURRENT	CH:2 VL:8	CH:1 VL:8	null
```
<!-- cSpell:enable -->
<!-- markdownlint-enable MD010 -->

<!-- omit in toc -->
### Date format

The date format is based on [RFC 3339](https://tools.ietf.org/html/rfc3339) and [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) with the following format in Unix:

```txt
%Y-%M-%DT%h:%m:%s%Z:%z
```

<!-- omit in toc -->
### Status

The status of the event. Can be either `EVENT`, `INFO`, `ERROR`, or `WARN`.

<!-- omit in toc -->
### Kind

The kind of the logged information. Can be either `USER`, `SERVICE`, `MEDIA`, `ANIMATION`, `COMIC`, `GAME`, `BOOK`, `TV`, `MOVIE`, `PODCAST`, `MUSIC`, `OTHER`, or `MESSAGE`.

<!-- omit in toc -->
### ID

The ID of the entry. Can be either `0` or `null` if the entry is not yet registered.

<!-- omit in toc -->
### Title

The title of entry.

<!-- omit in toc -->
### Event Type

Denoting what event has happened. Can be either:

<!-- cSpell:disable -->

<details><summary>Click to expand</summary>

* `PROGCH` - Progress chapter
* `PROGPRCT` - Progress percentile
* `PROGREP` - Progress repeat
* `PROGVOL` - Progress volume
* `SWCHPRIV` - Switch privacy
* `SWCHREP` - Switch repeat
* `SWCHSTAT` - Switch status
* `UPDEND` - Update end date
* `UPDNOTE` - Update note
* `UPDPRIO` - Update priority
* `UPDSCR` - Update score
* `UPDSTRT` - Update start date

</details>

<!-- cSpell:enable -->

<!-- omit in toc -->
### Substat

The substatus of the event. Can be either `CURRENT`, `PLANNED`, `PAUSED`, `STOPPED`, `COMPLETED`, or `null`

<!-- omit in toc -->
### After

Details of the event after the event has happened. Can be either `null` or a string.

<!-- omit in toc -->
### Before

Details of the event before the event has happened. Can be either `null` or a string.

<!-- omit in toc -->
### Message

The message of the event. Can be either `null` or a string.

<hr>

## Ryuuganime Media SaveFile Format Documentation

> **NOTE**
>
> JSON Path/Pointer in this documentation is based on [JSON Pointer](https://tools.ietf.org/html/rfc6901), for example. `/` is the root of JSON object (`$json`), and `/metadata` is the `$json.metadata` object.

### Code formatting, styling, and file name

Properties must follow `camelCase` format. Code indentation preferably 2 spaces.

File encoding must be UTF-8, and line ending must be LF (`\n`).

However, UTF-8 with BOM is also allowed.

Generally, the file name should append `.savefile` or `.sf` in the end of the file name, before the file extension.

For example:

```text
ryuuganime_anime.savefile.json
```

If the file is a headerless format, the file name should append `.headerless` or `.hl` in the end of the file name, before the file extension.

For example:

```text
ryuuganime_anime.hl.savefile.json
```

### SaveFile Format

SaveFile format is a JSON, YAML, HJSON, PSD1, or CSON file that contains media entries. The file itself can be saved as an object ([`headered`](#headered)) or as an array ([`headerless`](#headerless)).

#### Headered

A headered format is an object-format file that contains an information header (`/metadata`) object and an array of media entries (`/entries`).

Optionally, the headered format can also contains `$schema` property, which is a pointer to the schema file, by default it is `https://ryuuganime.github.io/mediaSaveFile/schema/savefile_v1.0.0.schema.json`.

#### Headerless

A headerless format only contains [an array of media entries](#entries-entries).

### Headered Metadata (`/metadata`)

* Type: Object
* Is Required: Yes
* Additional Properties: No

The headered metadata object contains information about the SaveFile format, such as the version of the format, the name of the SaveFile, and the description of the SaveFile.

The metadata also may contains information regarding:

* Service name, URI, and version
* Date of export
* Media type
* User information

Usage

```yaml
metadata:
  version: '1.0.0'
  name: MyService
  mediaType: animation
  description: MyService exported list
  service:
    name: MyService
    uri: https://myservice.com
    version: '1.0.0'
  exported:
    date: 2021-01-01T00:00:00.000Z
  user:
    id: 123456
    name: AnonyMouse
    uri: https://myservice.com/users/123456
```

To add more properties outside of the schema, you can add them to `/metadata/service`, `/metadata/user`, `/metadata/exported`, or `/metadata/other`.

#### `version`

* Type: String
* Is Required: Yes
* Format: [Semantic Versioning](https://semver.org/)

The version of the SaveFile format.

#### `name`

* Type: String
* Is Required: No

The name of the SaveFile.

#### `mediaType`

* Type: String
* Is Required: Yes
* Enumeration of: [MediaType[]](#enum-mediatype)

The media type of the SaveFile.

#### `description`

* Type: String
* Is Required: No

Description of the SaveFile.

#### `service`

* Type: Object
* Is Required: No
* Additional Properties: Allowed

Information regarding the service that exported the SaveFile.

##### `service/name`

* Type: String
* Is Required: Yes

The name of the service that exported the SaveFile.

##### `service/uri`

* Type: String
* Is Required: Yes
* Format: [URI](https://tools.ietf.org/html/rfc3986)

The URI of the service that exported the SaveFile.

##### `service/version`

* Type: String
* Is Required: No

The version of the service that exported the SaveFile.

##### `service/*`

* Type: Any
* Is Required: No

Additional properties that are specific to the service.

#### `exported`

* Type: Object
* Is Required: Yes
* Additional Properties: Allowed

Information regarding export.

##### `exported/date`

* Type: String
* Is Required: Yes
* Format: [RFC 3339 § 5.6: `Full-Date`](https://tools.ietf.org/html/rfc3339#section-5.6)

The date of export.

##### `exported/*`

* Type: Any
* Is Required: No

Additional properties that are specific to the export.

#### `user`

* Type: Object
* Is Required: No
* Additional Properties: Allowed

Information regarding the user.

##### `user/id`

* Type: String | Integer
* Is Required: Yes
* Format: Any

The ID of the user.

##### `user/name`

* Type: String
* Is Required: No

The name of the user.

##### `user/uri`

* Type: String
* Is Required: No
* Format: [URI](https://tools.ietf.org/html/rfc3986)

The URI of the user.

##### `user/*`

* Type: Any
* Is Required: No

Additional properties that are specific to the user.

### Entries (`/entries[]`)

The entries array contains media entries.

> WIP.

### Definitions

#### Enum: MediaType

* `animation`
* `comic`
* `game`
* `tv`
* `movie`
* `book`
* `podcast`
* `other`

> **NOTE**
>
> This is not a complete list of media types, and is subject to change.

#### Int: Progress

* Type: Integer
* Minimum: 0
* Maximum: *unset*

The progress of the media entry.

If the media is not yet started, the progress is `0`, replacing the `null` value.

## Usage Example

### JSON Headerless

```json
[
  {
    "id": 45619,
    "title": "SPY x FAMILY Part 2",
    "status": "current",
    "current": {
      "episode": 12,
      "isRepeating": false
    },
    "date": {
      "start": "2022-10-02",
      "finish": null
    },
    "notes": "",
    "repeatCount": 0,
    "rating": 0,
    "isPrivate": false,
    "metadata": {
      "slug": "spy-x-family-part-2",
      "date": {
        "season": "fall",
        "start": "2022-10-01",
        "finish": "2022-12-24"
      },
      "origin": {
        "language": "ja",
        "country": "JP"
      },
      "isNsfw": false
    }
  }
]
```

### HJSON Headerless

```hjson
[
  {
    id: 45619
    title: 'SPY x FAMILY Part 2'
    status: 'current'
    // Kitsu JSON:API does not support upstream episode datas
    current: {
      episode: 12
      isRepeating: false
    }
    date: {
      start: '2022-10-02'
      finish: null
    }
    notes: ''
    repeatCount: 0
    rating: 0
    isPrivate: false
    metadata: {
      slug: 'spy-x-family-part-2'
      date: {
        season: 'fall'
        start: '2022-10-01'
        finish: '2022-12-24'
      }
      origin: {
        language: 'ja'
        country: 'JP'
      }
      isNsfw: false
    }
  }
]
```

### PSD1 Headered

<!-- cSpell:words Ayumu Sensei Hikaru -->

```ps1
@{
  metadata = @{
    version = '1.0.0'
    # https://github.com/nattadasu/animeManga-autoBackup
    name = 'animeManga-autoBackup (MangaDex)'
    mediaType = 'comic'
    description = 'MangaDex exported list, by animeManga-autoBackup'
    service = @{
      name = 'MangaDex'
      uri = 'https://mangadex.org'
    }
    exported = @{
      date = '2021-01-01T00:00:00Z'
    }
    user = @{
      id = 'cd6eccb3-5bc0-42bc-b4a7-970036e9b544'
      name = 'AnonyMouse'
    }
  }
  entries = @(
    @{
      id = 'df1e3421-1bf4-4be1-b750-2cbcab901469'
      title = 'Ayumu Sensei to Hikaru-kun'
      status = 'planned'
      upstream = @{
        chapter = 4
        volume = 0
      }
      current = @{
        chapter = 0
        volume = 0
      }
      date = @{
        start = $Null
        finish = $Null
      }
      rating = 0
      isPrivate = $False
      metadata = @{
        mappings = @{
          myAnimeList = $Null
          aniList = $Null
          kitsu = $Null
          animePlanet = $Null
          mangaUpdates = '03g50ni'
          novelUpdates = $Null
        }
        status = 'completed'
        contentRating = 'safe'
        origin = @{
          language = 'ja'
          country = 'JP'
        }
      }
    }
  )
}
```

### YAML Headered

```yaml
---
metadata:
  version: '1.0.0'
  name: Trakt
  mediaType: movie
  description: Trakt exported list
  service:
    name: Trakt
    uri: https://trakt.tv
  exported:
    date: 2021-01-01T00:00:00Z
  # Below is not a real user (。_。)
  user:
    id: 123456
    name: AnonyMouse
entries:
  - id: 464
    title: Monsters, Inc.
    status: completed
    upstream:
      episode: 1
    current:
      episode: 1
      isRepeating: false
    date:
      finish: 2018-12-22
    notes: ''
    repeatCount: 0
    rating: 7
    metadata:
      slug: monsters-inc-2001
      mappings:
        imdb: tt0198781
        tmdb: 585
      year: 2001
...
```
