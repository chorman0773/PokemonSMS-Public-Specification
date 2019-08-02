# Info and Copyright Notice #

## Copyright ##
PokemonSMS Public Specification Project, Copyright 2018 Connor Horman
Pokemon, the Pokemon Logo, and all Official Pokemon are Copyright Nintendo and Game Freak. This Project is in no way affiliated with Nintendo or Game Freak, and disclaims all relation with the above parties. This project is intended as a Fan Game, or as Parody of Legitimate Pokemon titles, and no Concreate Game produced using this project should be considered legitimate or affiliated to the above companies, unless they provide official consent to the connection. This project, and all games produced using this specification intend no copyright infringement or Intellectual Property theft of any kind.<br/><br/>


The PokemonSMS Save File Specification("This Document"), provided by the PokemonSMS Public Specification Project ("This Project") is Copyright Connor Horman("The Owner"), 2018. 
Using the license specified by the project, you may, with only the restrictions detailed below,
(a)Use this document to produce a complete or partial implementation of PokemonSMS, 
(b)Use this document as reference material to create other related projects or derivative works,
(c)Transmit and/or Distribute Verbatim Copies of this document,
(d)Transmit and/or Distribute Partial copies of this document, which link to this document and include this copyright notice,
(e)Use this document as a guideline to design other specifications for projects, related or not,
(f)Quote parts of this document for works of any kind, that link to this document,
(g)Use part or the whole of this document for any purpose which does not constitute copyright infringement under the intellectual property laws of your jurisdiction.
<br/><br/>
The following restrictions (and only the following restrictions) apply to the above:
You may not, under any circumstances, 
(a)Claim that this document, or any part of it, belongs to you, 
(b)Use this document in any way that would be unlawful in your jurisdiction, or in Canada, 
(c)Use this document in a way that would infringe upon the copyright of any person, organization, or corporation, without prior written consent from that entity or an entity which has been designated by that company as a legal authority on their behalf, including the Owner, or Sentry Game Engine.
<br/><br/>
  This Document, and this project are distributed with the intention that they will be useful and complete. However this document and this project are provided on an AS-IS basis, without any warranties of Any Kind, including the implied warranties of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. As such, any person using this document for any reason does so at their own risk.  By using this document, you explicitly agree to release The Owner, and any person who might have distributed a copy of this document to you from all liability connected with your use of this document
<br/><br/>
### Additional Copyright Information ###

This document references the Named Binary Tag specification, created by Markus Person. The up to date specification can be located at <https://wiki.vg/NBT>. 
In the present version of this document, references to that specification imply Version 19133, and DO NOT use GZip or zlib compression. ShadeNBT only allows for uncompressed NBT Data. The specific Specification used defines TAG_LongArray.

## Info ##
Defines the Specification for the File Format used to store PokemonSMS Savegames, The Specific Structure of a Savegame, and Save File Generalization
# Specification [save] #

## ShadeNBT [save.shade] ##

PokemonSMS is specified to save in ShadeNBT Version 1.3 or any previous version. ShadeNBT is specified [here](https://chorman0773.github.io/BinarySpecifications/ShadeNBT). In version 1.3 or later, Shade_flags 0x20 (Allow NaN) MUST NOT be set. A file with that flag set MUST be rejected. 




## Save file generalization and storage [save.generic] ##
PokemonSMS supports multiple save files. A sqlite database of these save files is stored alongside the actual save files, with the name filename saves.pkmdb. The database shall contain a single table called "Saves" that uses the following Schema. 

```sqlite
CREATE TABLE `Saves` (
	`id`	INTEGER NOT NULL AUTOINCREMENT UNIQUE,
	`path`	TEXT NOT NULL,
	`nameComponent`	TEXT NOT NULL,
	`icon`	TEXT NOT NULL,
	`saveTimeSeconds`	INTEGER NOT NULL,
	`saveTimeNanos`	INTEGER NOT NULL,
	`encrypted`	BOOL NOT NULL,
	PRIMARY Key('id')
);
```

`id` is the save file number to uniquely identify the file, may be any number, but at most 1 Row may exist with a given `id` (hense `UNIQUE`)

`path` is a relative pathname to the save file (relative to the directory that saves.pkmdb is in). The pathname shall be case-insensitive, use `/` path separators if applicable (posix). It MUST NOT start with `/` or contain any navigation components (such as `..` or `.`). 

`nameComponent` is a Text Component representing the name given to the player in the file. 

`icon` is the Image which displays alongside the save file in the save file select menu, as a base64 encoded JPEG file

`saveTimeSeconds` is the total seconds the save file has been open for
`saveTimeNanos` is the nanosecond component of the Duration the save file has been open for in total.
`pokedexSeen` is the number of pokemon that are registered as seen in the pokedex
`pokedexCaught` is the number of pokemon that are registered as caught in the pokedex
encrypted is true if the file is using the CryptoShade Extension, false otherwise.

### Options [save.generic.options]

In the Global Save Table, global options are stored in a table called `Options`, defined with the following schema

```
CREATE TABLE `Options`(
	Option TEXT NOT NULL UNIQUE,
	Value TEXT NOT NULL
);
```

It is unspecified if this table is created if the values are modified from its default. Each Option Key MUST be an Entity Name. 
Implementations MAY add additional keys in a non-reserved domain (in particular, they may not use the `internal` or `impl` domains as this may clash with


### Implementation Stored Information [save.generic.impl] ###

Implementations may store other information in saves.pkmdb, such as global options, in other tables. What information is stored and if information is stored is implementation-defined. 

### Maximum Save Slots [save.generic.maxslots] ###

The maximum number of save slots an implementation supports is implementation defined, but must at least be 3. Implementations must accept save databases which indicate a greater number of save files. 

### Save File Location [save.generic.loc] ###

The Directory which save files are stored in is unspecified, and may not even be a physical directory on a filesystem. Implementations are permitted to store a logical filesystem in an unspecified file that contains 



## Save File Structure [save.struct] ##

The NBT Structure of a Save File is broken into chunks, which describe the various portions of the game. Each Portion is detailed below:

### Serializing Lua [save.struct.lua] ###
Lua Tables (and other values, but not Functions or Userdata) can be serialized. 

Tables must either represent a list (Mapping integer keys starting from 1 to some uniform type of values), or Structures (Mapping String Keys to some values), and may not contain values that cannot be serialized. Tables must also not be nested recursively. 


Tables that represent Structures are serialized as `TAG_Compound`, Lists are serialized as `TAG_List`. The elements of tables that can be serialized are serialized as such. 

Numbers are serialized as `TAG_Double` (if they do not meet the requirements for a Lua Integer defined in Lua 5.3 or do not fall in the range for a `TAG_Int`), or `TAG_Int` (if they do). The value `math.huge` serializes as `TAG_Double` which identifies the IEEE 754 Double value "Positive Infinity". The value `-math.huge` serializes as "Negative Infinity". 

Strings are serialized as `TAG_String`. Boolean values are serialized as the `TAG_Byte` containing 1b if they are true, and the `TAG_Byte` containing 0b if they are false. 

It is unspecified which userdata types can be serialized in NBT Form, except that the Duration type shall be serialized in a table as *name*Seconds as a `TAG_Long` and *name*Nanos as a `TAG_Int`, and shall be serialized in a list as a compound containing a `TAG_Long` Seconds field and `TAG_Int` Nanos field. 




### Pokemon Structure [save.struct.pokemon] ###
Pokemon stored in the Save file take the following format:

* (a pokemon)(compound) (tags common to all pokemon)
    * Species (string): The resource location that names the species of the pokemon. If  the string is not a valid resource location (see ResourceNaming.md), the resource location does not name a pokemon, or the string is `system:pokemon/null` the file is ill-formed. Must Exist, or the compound may not contain any other tags. 
    * CatchInfo (compound): The Information describing how the pokemon was obtained
        * TrainerId (long): The Id of the original trainer of this pokemon
        * TrainerSid (long): The secret Id of the original trainer
        * TrainerName (String): A String representing a text component. This is the name of the original trainer. Note that this is not matched to determine if the pokemon should be disobediant.
        * CatchTimestampSeconds (Long): The Number of seconds since 1970-01-01T00:00:00Z which the pokemon was caught at.
        * CatchTimestampNanos (int): The Number of nanos since the start of the second designated by CatchTimestampSeconds, which this pokemon was caught at.
        * CatchLocation (string): A resource location that names a location. Invalid Catch Locations should be preserved and treated as `pokemon:faraway`. If the location is `system:locations/null`, then the file is ill-formed. Not set if CatchLevel is 0. 
        * CatchFlags (byte): A bit array designating the flags applicable to the capture.  See Below for valid bitflags. 
        * CatchLevel (short): The level then pokemon was at when it was caught. Must be a number in [0,100]. 0 means the Pokemon was "caught" as an egg.  
        * CatchSpecies (string): A resource location identifying the Species that the pokemon was caught as. 
    * Stats (compound): The Stats of the Pokemon
        * Curr (Int Array): An array of 7 integers, designating the Current stats of the Pokemon. Follows Natural Stat Order (0 is attack, 1 is defense, 2 is special attack, 3 is special defense, 4 is speed, 5 is maximum hp). Index 6 designates the difference between the maximum hp and the current HP. 
        * EffortValues (Byte Array): An array of 6 bytes, designating the Effort Values in each Stat. Follows Natural Stat ordering. 
        * IndividualValues (Byte Array): An array of 6 bytes, designating the Individual Values in each Stat. Follows Natural Stat Ordering. 
        * Level (Short): The level the pokemon is currently at. Must be a number in [0,100]. If Level is 0, then CatchInfo.CatchLevel must also be 0 or the file is ill-formed. 
    * Individuality (compound): The Structure which defines the Pokemon Individuality Information
        * BlockId (long): The First of the 2 Pokemon Individuality Ids (Block Identifier). 
        * SlotId (long): The Second of the 2 Pokemon Individuality Ids (Slot Identifier)
        * PokemonFlags (byte): The flags associated with the Pokemon. See below
        * CryModifier (compound): The Invididuality Cry Modifiers
            * Pitch (byte): (Unsigned) The pitch modifier that applies to the pokemon
            * Volume (byte): (Unsigned) The volume modifier that applies to the pokemon.
    * Options (compound): A Compound containing implementation specific Information about the Pokemon
    * Items (compound): The held item and equipment item. Must exist but may be empty
        * Held (compound): The compound describing the held item. See Item Structure. May not exist or be empty. 
        * Equipment (compound): The compound describing the equipment (secondary) item. See Item Structure. May not exist.
    * Extra (compound): The Serialization of the Extra Table. Species Dependent. May not Exist
    * Moves (list): Specifies the moves the pokemon knows. May have at most 4 tags
        * (a move) (compound):
            * Move (string): The name of the move. Must be a valid resource location that names a move or the file is ill-formed. If this names `system:moves/null`, then no other tags may be defined. 
            * RemainingPP (byte): The PP the move has remaining. 
            * NumPPUps (byte): The number of PP Ups used on the move. May be at most 3 or the file is ill-formed. May not exist
    * Ability (string): The name of the ability of the pokemon. Must be a valid resource location that names an ability. If the Pokemon cannot have the ability specified, the result is unspecified. 
    
### Item Structure [save.struct.item] ###

Items are Stored in the following structure:

* (an item) (compound)  (Tags common to all items)
    * Id (string): The resource name that identifies the item. Must be a valid resource location that names an item or the file is ill-formed.
    * Count (short): The number of items in the stack. May not Exist. If undefined, defaults to 1. If it is defined, it must be the value 1 if the compound appears in a Pokemon's Items compound. 
    * Variant (string): The Variant Id of the item. May not exist, defaults to the default variant. The set of valid values and the properties of item variants is depedent on the item. 
    * Extra (compound): The serialization of the Item's Extra Table. Item Dependent. May Not Exist



		

