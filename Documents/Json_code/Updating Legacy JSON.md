# Updating Legacy JSON

Use the `home` key to get to the top.

- [Introduction](#introduction)
  + [Tools Required](#tools-required)
  + [Regex](#regex)
  + [What is JSON?](#what-is-json)
  + [Terminology in this Document](#terminology-in-this-document)
- [abstract, ident, and id](#abstract-ident-and-id)
- [Ammo Type](#ammo-type)
- [Artifacts](#artifacts)
- [barrel_length](#barrel_length)
- [Bleeding](#bleeding)
- [blob and slime](#blob-and-slime)
- [blueprint](#blueprint)
- [bullet_resist](#bullet_resist)
- [Color](#color)
- [copy-from and looks_like](#copy-from-and-looks_like)
- [Linting](#linting)
- [Materials](#materials)
- [Name](#name)
- [picklock](#picklock)
- [Pocket Data](#pocket-data)
  + [Gun Pocket data](#gun-Pocket-data)
  + [Magazine Pocket data](#magazine-Pocket-data)
- [`type: CONTAINER`](#type-container)
- [Volume](#volume)
- [Weight](#weight)
- [Effect](#effect)
- [Shape](#shape)

# Introduction
Welcome to Updating Legacy JSON.md. This document aims to guide you through the process of replacing obsolete code with modern JSON.

Before you go any further, I highly recommend you read the [Manual of Style](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/MANUAL_OF_STYLE.md), the [Guide to adding new content to CDDA for first time contributors](https://github.com/CleverRaven/Cataclysm-DDA/wiki/Guide-to-adding-new-content-to-CDDA-for-first-time-contributors), and the [JSON Style Guide](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_STYLE.md). These three documents provide necessary knowledge to understand CDDA's code.

## Tools Required
A lot of obsolete and legacy JSON requires the usage of find-and-replace to bring it up to date. To this end, I recommend that you have a text editor with regex capabilities such as [Sublime Text 3](https://www.sublimetext.com/3), [Notepad++](https://notepad-plus-plus.org/), or [atom](https://atom.io/). I personally find Sublime Text 3 the easiest to work with, but TheGoatGod advocates atom, and many others prefer Notepad++, so find what works best for you.

It can also be useful to have a file searcher on hand if you're editing large quantities of files, or modifying file names. [Grepwin](https://tools.stefankueng.com/grepWin.html) is what I use, and TheGoatGod recommends [Ultrasearch](https://www.jam-software.com/ultrasearch). Again, and I cannot emphasise this enough, use whatever works best for you.
-Goats Comment- I use both tools now, im going to end up with grepwin being my defualt.

## Regex
Regex, short for regular expressions, is a syntax language most commonly used in find-and-replace to find patterns and use complicated replaces. As you'll see later on, even simple regex can massively streamline the process of removing obsolete code. In each section below, I include the specific regex necessary for the task, and how to use it, so don't worry if you don't know what regex is. (For those of you interested in learning more, [Regex Quickstart Cheatsheet](https://www.rexegg.com/regex-quickstart.html) has the necessary information.).

## What is JSON?
JSON is short for Javascript Object Notation. It is a text format designed to be lightweight and easy for both humans and machines to read and write. JSON is made up of 5 key components: An `array`, an `object`, a `value`, a `number`, and a `string`.

An `array` is started with `[` and ended with `]`. It is composed of `value` separated by `,`. An `object` is started with `{` and ended with `}`. It is, like an array, composed of `value` separated by `,`. A `value` is any other JSON component or `true`, `false`, `null`. A `string` is enclosed in `""` and is a sequence of characters (text). A `number` is simply a number. See [Introducing JSON](https://www.json.org/json-en.html) to learn more.

All of JSON is made up of `key: value` pairs. A `key` is a `string`, while a `value` is a `value`. `array` and `object` are used to format `key: value` pairs, and to provide multiple `value` to a single `key`. Here are some examples of `key: value` pairs used in CDDA code:
```JSON
"flags": [ "SEES", "HEARS", "SMELLS", "POISON", "NO_BREATHE", "REVIVES", "BONES" ]  //A single key contains an array, which then contains multiple values.
"weight": "1000 g"  //A simple key: value pair.
"damage": { "damage_type": "bullet", "amount": 100, "armor_penetration": 4 }  //A single key contains an object, which in turn contains 3 key: value pairs.
"extend": { "effects": [ "FRAG", "EXPLOSIVE_SMALL" ] }  //A key, extend, contains an object value, which in turn contains a key, that contains an array.
"components": [ [ [ "iotvucp", 1 ] ], [ [ "ceramic_plate", 4 ] ] ]  //A key contains an array, which contains an array, which contains an array, which contains 2 value.
```
Note that in the above examples, the single `key` does not contain multiple `value` - the `array` or `object` contains the extra `value`, the `key` `value` is an `array` or `object`.

## Terminology in this Document
Throughout this document, I refer to the previous JSON terminology. Any specific `key` will look like `this`, while a key value pair will look like `key: value`. The 5 previously mentioned JSON components will always looke like this: `array`, `value`, `string`, `number`, `object`. Any large block of code (such as the one above), will look like this,
```JSON
key: value,
key: [ value ], //note that these are highlighted in red because plain text inside an array or object must be a string, and thus enclosed in "".
key: { key: value, key: value, key: value }
```

When I provide regex it will always be in the format find entry, empty line, replace entry. If there is no replace entry, **do not replace with nothing**. Here is an example of regex from later in this document:
```regex
"material": "([a-z]+)"

"material": [ "$1" ]
```

# abstract, ident, and id
`abstract` and `id`, are a specific type of `key` that tells the game the unique identifier of the item. Almost every top-level JSON `object` contains an an identifier and a `type`. `type` is an incredibly important `key` that tells the game how to handle the specific `key: value` pairs in the `object`.


`abstract` can only be used on `type: TOOL`, `type: GENERIC`, `type: GUN`, `type: COMESTIBLE`, `type: BOOK`, `type: AMMO`, `type: PET_ARMOR`, `type: vehicle_part`, `type: BIONIC_ITEM`, `type: ARMOR`, `type: TOOLMOD`, `type: ENGINE`, `type: MONSTER`, `type: uncraft`, and `type: overmap_terrain`.

`abstract` creates a pseudo-item that only exists to be copied from and is discarded after JSON loading is complete. If you see it used outside of these fields, replace with `ident` or `id` as appropriate. For debugging purposes, it is preferable to not use `abstract` on types other than `type: overmap_terrain`. It can cause cascading errors that are impossible to find due to lack of item id and presence in the game. See [JSON Inheritance](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INHERITANCE.md) for more information on how to use abstract.

`id` can (and perhaps should) be used anywhere `abstract` can be. It is used almost anywhere and doesn't need updating. Check out [JSON Info](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INFO.md) for more information on where to use `id`.


# Ammo Type
It is possible to specify the `key: value` pair `ammo: string` under both `type: GUN`, and `type: AMMO`. However, under `type: GUN`, the `key: value` pair should actually be `ammo: [ string ]`.
For reference, here is what it should look like under `type: GUN`:
```JSON
"ammo": [ "300blk" ]
```
And what it should look like under `type: AMMO`:
```JSON
"ammo_type": "300blk"
```

It is practically impossible to replace all at once, due to the similarities between `type: GUN` and `type: AMMO`. However, if you exclude all `type: AMMO` from the search (manually or otherwise), this will work:
```regex
"ammo": "([^\s]+)"

"ammo": [ "$1" ]
```


# Artifacts
Artifacts are currently undergoing a massive change. Nothing too important has changed yet, so this is simply a placeholder section.


# barrel_length
Very recently, the `barrel_length` `key` for `type: GUN` has been replaced by `barrel_volume`. Fortunately this is a rather easy fix:
```regex
"barrel_length":

"barrel_volume":
```

Less recently, at some point, `barrel_length` switched `value` from a `number` to a `string`. The modern JSON looks like this:
```JSON
"barrel_volume": "750 ml"
```
While the obsolete JSON looks like this:
```JSON
"barrel_length": 1
```
The conversion from `number` to `string` is simple - just multiply by 250 and add ml.
```JSON
"barrel_length": 1 = "barrel_length": "250 ml"
"barrel_length": 5 = "barrel_length": "1250 ml"
```
As the result is distinctly different, you will have to use manual find and replace for this part as specified in [volume](#volume).

# Bleeding
I have no idea where to start with this. If you have any information, please feel free to comment.


# blob and slime
It turns out that THE BLOB is not the same as the blobs you see around all the time. Check out [blobs are slimes](https://github.com/CleverRaven/Cataclysm-DDA/pull/42287) for more info. Mentions to blob may have to be updated to slime.


# blueprint
If you've been directed here from the linting section, it is because you have the parameter "blueprint": "", when it should be "blueprint": [ " " ], - A blueprint must always be enclosed in an array. Since this doesn't actually effect the game in any way (blueprint is exclusively used in the code), adding it exclusively for the purpose of linting should be enough.


# bullet_resist
`bullet_resist` is a new mandatory `key: value` pair for `type: material`. If it does not contain `bullet_resist`, it will cause the game to be unable to run. The only way to fix this is to manually add:
```JSON
"bullet_resist": number,
```

# Color
Color has gone through updates and there are many variants of ...


# copy-from and looks_like
These are both simple type errors. They should look like:
```JSON
"copy-from": "example_item"
"looks_like": "example_item"
```
But can also be typed:
```JSON
"copy_from": "example_item"
"looks-like": "example_item"
```
Running a basic find and replace for each will clean the code of any errors caused by these.


# Linting
Linting is a coding term for formatting to a certain style, and is a very important part of bringing JSON up to date. The simplest way to lint JSON is to paste it into the [JSON formatter](http://dev.narc.ro/cataclysm/format.html), click 'Lint', and then paste the resulting code back into the original file. If it doesn't work, use the debug steps at [JSONLint](https://jsonlint.com/), to check for errors in the code. If it comes up with the error 'Linter currently unavailable', see the [blueprint](#blueprint) section of this document.

It is also possible to use the JSON formatter that comes with CDDA, see the [JSON Style Guide](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_STYLE.md) for information on how to use it.


# Materials
Many items specify a `material: value` pair, which should have an `array` first, like this:
```JSON
"material": [ "plastic" ],
"material": [ "plastic", "steel" ]
```
However, this did not used to be required, so it was specified instead as:
```JSON
"material": "plastic"
```
This is easily fixed with a regex search:
```regex
"material": "([a-z]+)"

"material": [ "$1" ]
```


# Name
Almost everything that can be seen by characters has a `name: string` `key: value` pair. However, a small subset of these should be specified as:
```JSON
"name": { "str": "pair of socks", "str_pl": "pairs of socks" }
"name": { "str_sp": "irradiated celery" } //use this if the item should not pluralise at all.
```
A good guide as to whether it should be the above code instead of the code below is if it includes a `nam_pl` `key`:
```JSON
"name": "pair of socks",
"name_pl": "pairs of socks"
```
Otherwise, check out [JSON Info](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INFO.md) for more details on which one is correct in any given scenario.

If you are sure that the files you are replacing in need to be in the new format, you can use:
```regex
"name": "([ a -z ]+)",

"name": { "str": "$1" },
```
Then,
```regex
"name_plural": "([ a-z]+)",

"str_plural": "$1",
```
Afterwards, manually join the fields together. Trust me, it'll still save time.

# picklock
It is very possible that you will see the `use_action` `picklock`. This has been rendered obsolete by the addition of the `quality` `LOCKPICK`. You may see:
```JSON
"use_action": { "type": "picklock", "pick_quality": 5 }
```
Which should be:
```JSON
"qualities": [ [ "LOCKPICK", 5 ] ]
```

This has to be done manually due to the possible presence of other text in the `use_action` and a pre-existing `qualities` key.

# Pocket Data
In [JSON Tools](https://github.com/CleverRaven/Cataclysm-DDA/tree/master/tools/json_tools), there is pocket_mags.py that should be able to handle some of the work.


## Gun Pocket Data
legacy code just delete These **needs updating**
```JSON
"magazines": [ [ "Example_ammo", [ "Example_magazine1", "Example_magazine2" ] ] ]
"magazine_well": 1
```
new code for guns with magazines
```JSON
"pocket_data": [
  {
    "magazine_well": "250 ml",
    "pocket_type": "MAGAZINE_WELL",
    "holster": true,
    "max_contains_volume": "20 L",
    "max_contains_weight": "20 kg",
    "item_restriction": [ "Example_magazine1", "Example_magazine2" ]
  }
]
```
new code for guns without magazines
```JSON
"pocket_data": [ { "pocket_type": "MAGAZINE", "rigid": true, "ammo_restriction": { "Ammo_example": 100 } } ]
```

## Magazine Pocket Data
add these to "type": `Magazine` **needs updating**
```JSON
"pocket_data": [ { "pocket_type": "MAGAZINE", "rigid": true, "ammo_restriction": { "Ammo_example": 100 } } ]
```


## Container Pocket data
This is more complicated **needs updating**
```JSON
"pocket_data": [
  {
    "pocket_type": "CONTAINER",
    "rigid": true,
    "max_contains_volume": "4 L",
    "max_contains_weight": "30 kg",
    "moves": 200
  }
],
```


# type: CONTAINER
`type: CONTAINER` has been obsolete for a while now, and having it in JSON causes error messages. The following should easily remove any problems with `type: CONTAINER`:
```regex
"type": "CONTAINER"

"type": "GENERIC"
```

# Volume
The current JSON standards for the `key` `volume` look like this:
```JSON
"volume": "250 ml"
```
With obsolete JSON looking like this:
```JSON
"volume": 1
```
The conversion from `number` to `string` is:
```JSON
"volume": 1  =  "volume": "250 ml"
"volume": 20  =  "volume": "5000 ml"
"volume": "10000 ml"  =  "volume: 10 L"
```

Unfortunately, updating volume is not as simple as replacing all volume values with their modern version, as volume is also sometimes used to determine the loudness of sounds. However, this is relatively uncommon, so you can often find those specific files and remove them from this search.
```regex
"volume": 1

"volume": "250 ml"
```
And repeat for every individual volume value.

Note: There is, in [JSON tools](https://github.com/CleverRaven/Cataclysm-DDA/tree/master/tools/json_tools), a specific file called CDDAUpdateJsonVolume.js that may be relevant to fixing this.


# Weight
The current JSON standards for `key` `weight` look like this:
```JSON
"weight": "100 g"
```
With obsolete JSON looking like this:
```JSON
"weight": 100
```
The conversion from `number` to `string` is:
```JSON
"weight": 1  =  "weight": "1 g"
"weight": 20  =  "weight": "20 g"
```

Unfortunately, updating weight is not as simple as replacing all weight values with their modern version, as weight is quite frequently used to determine the probability of a specific piece of terrain spawning in mapgen. Once you have determined that there are no mapgen files in the area you wish to change, you can use this:
```regex
"weight": ([0-9]+),

"weight": "$1 g",
```


# Effect
The current JSON standards for `key` `effect` look like this: **needs updating**
```JSON
"effect": "attack"
```
With the obsolete json looking like this
```JSON
"effect": "target_attack"
```
with the fix being simple
```JSON
"effect": "target_attack"
into
"effect": "attack"
```

if you want to do this quickly with regex use the following example
```regex
"effect": "([a -zA -Z]+)",

"effect": "attack",
```

# Shape
The current JSON standards for `key` `shape` look like this: **needs updating**
```JSON
"shape": "blast"
```
all shapes -
```JSON
"line"
"cone"
"blast"
```
