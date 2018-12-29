---
# DO NOT TOUCH — Managed by doc writer
ContentId: f470466d-89b0-4115-ab7a-2448023b0a6d
DateApproved: 12/6/2018

# Summarize the whole topic in less than 300 characters for SEO purpose
MetaDescription: A guide to creating Icon Theme in Visual Studio Code
---

# Icon Theme

Visual Studio Code displays icons next to filenames throughout its UI, and extensions can contribute new sets of file icons that users can choose from.

## Adding a new Icon Theme

You can create your own icon theme from icons (preferably SVG) and from icon fonts. As example, check out the two built-in themes: [Minimal](https://github.com/Microsoft/vscode/tree/master/extensions/theme-defaults) and [Seti](https://github.com/Microsoft/vscode/tree/master/extensions/theme-seti).

To begin, create a VS Code extension and add the `iconTheme` contribution point.

```json
{
  "contributes": {
    "iconThemes": [
      {
        "id": "turtles",
        "label": "Turtles",
        "path": "./fileicons/turtles-icon-theme.json"
      }
    ]
  }
}
```

The `id` is the identifier for the icon theme. It is currently only used internally. In the future, it might be used in the settings, so make it unique but also readable. `label` is shown in the icon theme picker drop-down. The `path` points to a file in the extension that defines the icon set. If your icon set name follows the `*icon-theme.json` name scheme, you will get completion support and hovers in VS Code.

### Icon Set File

The icon set file is a JSON file consisting file icon associations and icon definitions.

An icon association maps a file type ('file', 'folder', 'json-file'...) to an icon definition. Icon definitions define where the icon is located: That can be an image file or also glyph in a font.

### Icon definitions

The `iconDefinitions` section contains all definitions. Each definition has an id, which will be used to reference the definition. A definition can be referenced also by more than one file association.

```json
{
  "iconDefinitions": {
    "_folder_dark": {
      "iconPath": "./images/Folder_16x_inverse.svg"
    }
  }
}
```

This icon definition above contains a definition with the identifier `_folder_dark`.

The following properties are supported:

- `iconPath`: When using a svg/png: the path to the image.
- `fontCharacter`: When using a glyph font: The character in the font to use.
- `fontColor`: When using a glyph font: The color to use for the glyph.
- `fontSize`: When using a font: The font size. By default, the size specified in the font specification is used. Should be a relative size (e.g. 150%) to the parent font size.
- `fontId`: When using a font: The id of the font. If not specified, the first font specified in font specification section will be picked.

### File association

Icons can be associated to folders, folder names, files, file extensions, file names and [language ids](/api/references/contribution-points#contributeslanguages).

Additionally each of these associations can be refined for 'light' and 'highContrast' color themes.

Each file association points to an icon definition.

```json
{
  "file": "_file_dark",
  "folder": "_folder_dark",
  "folderExpanded": "_folder_open_dark",
  "folderNames": {
    ".vscode": "_vscode_folder"
  },
  "fileExtensions": {
    "ini": "_ini_file"
  },
  "fileNames": {
    "win.ini": "_win_ini_file"
  },
  "languageIds": {
    "ini": "_ini_file"
  },
  "light": {
    "folderExpanded": "_folder_open_light",
    "folder": "_folder_light",
    "file": "_file_light",
    "fileExtensions": {
      "ini": "_ini_file_light"
    }
  },
  "highContrast": {}
}
```

- `file` is the default file icon, shown for all files that don't match any extension, filename or language id. Currently all properties defined by the definition of the file icon will be inherited (only relevant for font glyphs, useful for the fontSize).
- `folder` is the folder icon for collapsed folders, and if `folderExpanded` is not set, also for expanded folders. Icons for specific folder names can be associated using the `folderNames` property.
  The folder icon is optional. If not set, no icon will be shown for folder.
- `folderExpanded` is the folder icon for expanded folders. The expanded folder icon is optional. If not set, the icon defined for `folder` will be shown.
- `folderNames` associates folder names to icons. The key of the set is the folder name, not including any path segments. Patterns or wildcards are not supported. Folder name matching is case insensitive.
- `folderNamesExpanded` associates folder names to icons for expanded folder. The key of the set is the folder name, not including any path segments. Patterns or wildcards are not supported. Folder name matching is case insensitive.
- `languageIds` associates languages to icons. The key in the set is the language id as defined in the [language contribution point](/api/references/contribution-points#contributeslanguages). The language of a file is evaluated based on the file extensions and file names as defined in the language contribution. Note that the 'first line match' of the language contribution is not considered.
- `fileExtensions` associates file extensions to icons. The key in the set is the file extension name. The extension name is a file name segment after a dot (not including the dot). File names with multiple dots such as `lib.d.ts` can match multiple extensions; 'd.ts' and 'ts'. Extensions are compared case insensitive.
- `fileNames` associates file names to icons. The key in the set is the full file name, not including any path segments. Patterns or wildcards are not supported. File name matching is case insensitive. A 'fileName' match is the strongest match, and the icon associated to the file name will be preferred over an icon of a matching fileExtension and also of a matching language Id.

A file extension match is preferred over a language match, but is weaker than a file name match.

The `light` and the `highContrast` section have the same file association properties as just listed. They allow to override icons for the corresponding themes.

### Font definitions

The 'fonts' section lets you declare any number of glyph fonts that you want to use.
You can later reference these font in the icon definitions. The font declared first will be used as the default if an icon definition does not specify a font id.

Copy the font file into your extension and set the path accordingly.
It is recommended to use [WOFF](https://developer.mozilla.org/docs/Web/Guide/WOFF) fonts.

- Set 'woff' as the format.
- the weight property values are defined [here](https://developer.mozilla.org/docs/Web/CSS/font-weight#Values).
- the style property values are defined [here](https://developer.mozilla.org/docs/Web/CSS/@font-face/font-style#Values).
- the size should be relative to the font size where the icon is used. Therefore, always use percentage.

```json
{
  "fonts": [
    {
      "id": "turtles-font",
      "src": [
        {
          "path": "./turtles.woff",
          "format": "woff"
        }
      ],
      "weight": "normal",
      "style": "normal",
      "size": "150%"
    }
  ],
  "iconDefinitions": {
    "_file": {
      "fontCharacter": "\\E002",
      "fontColor": "#5f8b3b",
      "fontId": "turtles-font"
    }
  }
}
```

### Folder icons in File Icon Themes

File Icon themes can instruct the File Explorer not to show the default folder icon (the rotating triangles or "twisties") when the folder icons are good enough to indicate the expansion state of a folder. This mode is enabled by setting `"hidesExplorerArrows":true` in the File Icon theme definition file.