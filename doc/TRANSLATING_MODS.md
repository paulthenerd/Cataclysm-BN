# Translating mods for Cataclysm: BN

- [Intro](#intro)
- [A short glossary](#a-short-glossary)
- [Workflow overview](#workflow-overview)
- [Setting up environment for string extraction](#setting-up-environment-for-string-extraction)
- [Extracting strings](#extracting-strings)
- [Creating new PO](#creating-new-po)
- [Updating existing PO](#updating-existing-po)
- [Compiling PO into MO](#compiling-po-into-mo)
- [Adding MO file to the mod](#adding-mo-file-to-the-mod)
- [Miscellaneous notes](#miscellaneous-notes)

## Intro
This document aims to give a brief explanation on how to set up and operate
mod translation workflow for Cataclysm: Bright Nights.

For mod localization the game uses custom localization system that is similar to
[GNU gettext](https://www.gnu.org/software/gettext/) and is compatible with GNU gettext MO files.
The system is currently in experimental state and must be enabled in Debug settings tab
(`Modular translations testing` option). Please report any bugs or suggestions
on our discord or by submitting an issue on our GitHub page.

While it's possible to use Transifex or any other platform or software that supports gettext,
this document only gives examples on how to work with [Poedit](https://poedit.net/) and 
command-line [GNU gettext utilities](https://www.gnu.org/software/gettext/).

If you desire an in-depth explanation on PO/POT/MO files or how to work with them using GNU gettext utilities,
see [GNU gettext manual](https://www.gnu.org/software/gettext/manual/gettext.html).

To get some generic tips on translating strings for Cataclysm: Bright Nights and its mods,
see [TRANSLATING.md](TRANSLATING.md).

## A short glossary
### POT file
Portable Object Template file (`.pot`).

This is a text file that contains original (English) strings extracted from mod's JSON files.
The POT file is a template used for creating empty or updating existing PO files of any language.

### PO file
Portable Object file (`.po`).

This is a text file that contains translated strings for one language.
The PO files are what translators work with, and what will be compiled into a MO file.

### MO file
Machine Object file (`.mo`).

This is a binary file that contains translated strings for one language.
The MO files are what the game loads, and where it gets translated strings from.

## Workflow overview
The first translation workflow is as follows:

1. Extract strings from mod JSON files into a POT file
2. Create PO file for target language from this POT
3. Fill the PO file with translated strings
4. Compile PO into MO
5. Put the MO into your mod files

As the mod changes with time, so may change its strings.
Updating existing translations is done as follows:

1. Extract strings from mod JSON files into a new POT file
2. Update existing PO file from the new POT file
3. Add new or edit existing translated strings in the PO file
4. Compile PO into MO
5. Replace old MO in the mod files with new version

Step 1 in both workflows requires you to set up environment for string extraction (see below).

Steps 2-4 can be done using translation software either by the mod author/maintainer, or by the translator.

## Setting up environment for string extraction
You'll need Python 3 with `polib` library installed (available via `pip`).

Scripts for string extraction can be found in the `lang` subdirectory of the repository:
* `extract_json_strings.py` - main string extraction routines
* `dedup_pot_file.py` - fixes errors in POT file produces by the 1st script

## Extracting strings
To extract strings, you'll need to run these python scripts in the following order:
```bash
python extract_json_strings.py -i path/to/mod/ -o path/to/resulting/index.pot --project YourModName
python dedup_pot_file.py path/to/resulting/index.pot
```

Replace `YourModName` with the mod's name.
This is what translation software would use as project name for PO/POT,
so it doesn't have to match actual mod name or id.

Replace `path/to/resulting/index.pot` with output POT file name.

Replace `path/to/mod/` with path to the mod's folder. 
Resulting POT will remember which file each string comes from (including `path/to/mod/` part),
and as such it's best to keep this path as short as possible
(or even use `./` and run the scripts from the mod's directory)
to keep the resulting references short, clean and easy for translators to understand.

If the mod is under development and you plan on keeping translations up-to-date in the future,
it may be worthwhile to create _another_ script (batch or bash, depending on your OS)
that would take care of string extraction process for you.

Extraction script itself (`extract_json_strings.py`) has a number of extra features
not documented here; call it with `-h` flag (or see the code) for more info.

## Creating new PO
Before creating PO file, you need to choose language id.

Open `data/raw/languages.json` to see the list of languages supported by the game.

In this list, each entry has its own id in form of `ln_LN`,
where `ln` determines language and `LN` - dialect.
You can either use full `ln_LN` for exact language+dialect match,
or `ln` if you want the game to use your MO regardless of dialect.

### Poedit
1. Open the POT file with Poedit
2. Press "Create new translation" button (should show up near the bottom)
3. In language selection dialog, enter language id you chose
4. Save the file as `path/to/mod/lang/LANG.po` where `LANG` is the same language id

### msginit
```bash
msginit -i lang/index.pot -o lang/LANG.po -l LANG.UTF-8 --no-translator
```
Where `LANG` is the language id you chose

## Updating existing PO
### Poedit
1. Open the PO file with Poedit
2. Choose `Catalog->Update from POT file...` and select the new POT file
3. Save the file

### msgmerge
```bash
msgmerge lang/LANG.po lang/index.pot
```

## Compiling PO into MO
### Poedit
1. Open the PO file with Poedit
2. Make sure MO file will be encoded using UTF-8 (it should do so by default,
   you can double check in `Catalog->Properties->"Translation properties" tab->Charset`).
3. By default, each time PO file is saved Poedit automatically compiles it into MO,
   but the same can also be done explicitly via `File->Compile to MO...`

### msgfmt
```
msgfmt -o lang/LANG.mo lang/LANG.po
```

## Adding MO file to the mod
Create `lang` directory in the mod files directory and put MOs there:

```
mods/
    YourMod/
        modinfo.json
        lang/
            es.mo
            pt_BR.mo
            zh_CN.mo
```

**Note:**  Storing your optional extraction script and POT/PO files
in the same `lang` subdirectory may make it easier to keep track of them.
The game ignores these files, and your mod folder structure will look like this:

```
mods/
    YourMod/
        modinfo.json
        lang/
            extract.bat  (extract.sh on MacOS/Linux)
            index.pot
            es.po
            es.mo
            pt_BR.po
            pt_BR.mo
            zh_CN.po
            zh_CN.mo
```

## Miscellaneous notes
### Is it possible to use arbitrary location or names for MO files, like with JSONs?
No. The game looks for MO files with specific names that are located in the
`lang` subdirectory of the mod's `path` directory specified in `modinfo.json`
(if not specified, `path` matches the mod's directory).

However, any mod will automatically try to use any other mod's translation
files to translate its strings. This makes it possible to create mods that are
purely "translation packs" for other mods (or mod collections).

### Reloading translations in a running game
Open debug menu and select `Info...->Reload translations`,
and the game will reload all MO files from disk.

This makes it easy to see how translated string looks in game,
provided the translator has a way to compile MO files.

Example workflow with Poedit:
1. Translate a string
2. Hit Ctrl+S
3. Alt+Tab into the game
4. Reload translation files via debug menu
5. The game now displays translated string

### Dialects and MO load order
When loading MO files, the game first looks for the file with
exact language and dialect match.
If such file is absent, then it looks for a file with no dialect.

For example, when using `Español (España)` the load order is
1. `es_ES.mo`
2. `es.mo`

And when using `Español (Argentina)` the load order is
1. `es_AR.mo`
2. `es.mo`

Thus, `es.mo` would be loaded for either dialect of Spanish
if the exact translation files are not present.

### What if 2 or more mods provide different translations for same string?
Then the game uses translation from the first such mod in the mod loading order.

The in-repo mods (including the core content "mod") are an exception:
all of them use single MO file, which is loaded at all times and always
takes priority over 3-rd party translations.

If you want a different translation from the one in the base game,
add a translation context to the string.
