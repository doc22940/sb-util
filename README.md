# sb-util

## Table of Contents

-   [sb-util Proposal (RFC)](https://github.com/bocoup/sb-util/blob/master/README.md#sb-util-proposal-rfc)
    -   [JavaScript API Proposal](https://github.com/bocoup/sb-util/blob/master/README.md#javascript-api-proposal)
    -   [CLI Proposal](https://github.com/bocoup/sb-util/blob/master/README.md#cli-proposal)
-   [Development](https://github.com/bocoup/sb-util/blob/master/README.md#development)

---

## sb-util Proposal (RFC)

We, at Bocoup, propose **sb-util**, a JavaScript and CLI utility that allows developers to query a Scratch Project for collections of sprites, blocks, assets, and project metadata.

sb-util will accomplish this by consuming **.sb3** files generated by Scratch. **.sb3** files are used to save and load projects, and within an **.sb3** file there is a **project.json**, which is the JSON representation of the entities within a Scratch project. sb-util will provide an API that allows developers to query the project.json for project information.

The resulting tool should be usable in test suites, scripts, and applications.

## Javascript API Proposal

-   [Loading a Scratch Project](https://github.com/bocoup/sb-util#javascript-api-proposal)
-   [ScratchProject](https://github.com/bocoup/sb-util#scratchproject)
-   [SpriteCollection](https://github.com/bocoup/sb-util#spritecollection)
-   [Sprite](https://github.com/bocoup/sb-util#sprite)
-   [BlockCollection](https://github.com/bocoup/sb-util#blockcollection)
-   [Block](https://github.com/bocoup/sb-util#block)
-   [AssetCollection](https://github.com/bocoup/sb-util#assetcollection)
-   [Asset](https://github.com/bocoup/sb-util#asset)

API methods that have been implmented are marked with _implemented_.

### **Loading a Scratch Project** - _implemented_

sb-util exposes loading functions to asynchronously instantiate a **ScratchProject**
object. These loading functions handle file I/O and HTTP request handling, decoupling that process from the ScratchProject object itself.

---

**loadSb3(_sb3File_)**

Parameter(s): _sb3File_. String representing local file location of an _.sb3 file or a URI to an _.sb3 file  
Return: _Promise_. This Promise object will resolve to a ScratchProject

```
const sp = await loadSb3('foo.sb3');
```

**loadProjectJson(_projectJsonFile_)**

Parameter(s): _projectJsonFile_. String representing local file location of an project.json file or a URI to an project.json file  
Return: _Promise_. This Promise object will resolve to a ScratchProject

```
const sp = await loadProjectJson('project.json');
```

**loadCloudId(_cloudId_)**

Parameter(s): _cloudId_. Number representing a Cloud ID in Scratch  
Return: _Promise_. This Promise object will resolve to a ScratchProject

```
const sp = await loadCloudId(123456);
```

---

### ScratchProject

**ScratchProject(_projectJSON_)** - _implemented_  
A _ScratchProject_ gets initialized by an object, represented by the project.json. The _assetFetcher_ is an optional constructor argument that represents an object is responsible for retrieving asset buffers for an Asset object.

```
const { ScratchProject } - require('sb-util');

// Use the above loading methods or directly instantiate:
const sp = new ScratchProject(projectJson);
```

**Methods**

**assets()**  
Return: _AssetCollection_ representing all the assets of a project

```
let assets = sp.assets()
```

**blocks()** -- _implemented_  
Return: _BlockCollection_ representing all the blocks in the project. This _BlockCollection_ can be further filtered to get specific blocks

```
let blocks = sp.blocks();
```

**sprites(...args)** - _implemented_  
Return: _SpriteCollection_ representing all sprites in the project. A selector syntax can be passed to this function to get sprites that meet the syntax criteria

```
let sprites = sp.sprites();

let stage = sp.sprites('[isStage=true]').pop();
let sprite1 = sp.sprites('[name="Cat"]').pop();
```

**stage()** - _implemented_  
Return: _Sprite_ a stage

```
let stage = sp.stage();
```

**variables()**
Return: a list of _Variable_ objects in the project

```
let vars = sp.variables();
```

---

### Collections

There are multiple "Collection" classes that all share some collection methods, the shared methods are listed here.

**@\_\_iterator()** - _implemented_
Collection objects are meant to be iterated using the `for ... of` `[Symbol.iterator]` protocol. Items iterated in this way will be emited using the "Wrapper Class" for the collection (e.g. Block for BlockCollection)

**props()** - _implemented_
Return: the Iterable of the raw "properties" objects for the items in the collection.

**count()** - _implemented_
Return: Counts the items in the collection, similar to `.length` on an array but these collections aren't arrays.

**first()** - _implemented_
Return: the first item in the collection wrapped, null if no items in collection.

**index(n)** - _implemented_
Return: Similar to using `array[n]` on an array, this will get the wrapped item at the specified index in the collection, or null if out-of-bounds.

### SpriteCollection

A _SpriteCollection_ represents an iterable collection of objects that represent Sprites.

**Methods**

**query(selector)** - _implemented_  
Parameter(s): _selector_ string in the CSS selector syntax style.
Return: _SpriteCollection_

```
let stage = sp.sprites('[isStage=true]');
let sprite1 = sp.sprites('[name="Cat"]');
```

Possible selector syntax, in attribute selector style:

| Sprite Attribute | Selector Syntax                                                  |
| ---------------- | ---------------------------------------------------------------- |
| isStage          | [isStage={true or false}]                                        |
| layerOrder       | [layerOrder={a number}]                                          |
| draggable        | [draggable={true or false}]                                      |
| rotationStyle    | [rotationStyle={"all around" or "left-right" or "don't rotate"}] |

---

### Sprite

A _Sprite_ is a singleton of _SpriteCollection_, with additional methods that are specific to a single Sprite. A Sprite can be a stage or an individual sprite.

**Methods**

**prop(attribute)** - _implemented_  
Parameter: _attribute_ string.
Return: _any_ value for a given attribute

```
const currentCostume = sprite.prop('currentCostume');
```

Attributes available to pass: _name, isStage, variables, lists, broadcasts, blocks, comments, currentCostume, costumes, sounds, volume, layerOrder, temp, videoTransparency, videoState, textToSpeechLanguage, visible, x, y, size, direction, draggable, rotationStyle_

**blocks()** - _implemented_  
Return: _BlockCollection_

```
const sprite = sp.sprites('[name="Cat"]');
const blocks = sprite.blocks();
```

**assets()**  
Return: _AssetCollection_

```
const assets = sprite.assets();
```

**position()** - _implemented_  
Return: the (X, Y) cooredinates of a Sprite in Object notation

```
const { x, y } = sprite.position();
```

**broadcasts()** - _implemented_  
Return: a list of Objects representing a broadcast, which contains an id and a message

```
const broadcasts = sprite.broadcasts();

// A mapping example
Object.entries(broadcasts).map(([key, value]) => ({ messageId: key, message: value }));
```

**lists()** - _implemented_  
Return: a list of Objects representing a list, which contains an id, name, and an Array of values

```
const lists = sprite.lists();

Object.entries(lists).map(([key, value]) => console.log({key, listName: value[0], values: value[1]}));
```

---

### BlockCollection

A _BlockCollection_ represents and iterable collection of objects that represent Blocks.

**Methods**

**query(selector)** - _partially implemented_  
Parameter(s): _selector_, a string with the convention similar to CSS selector syntax  
Return: _BlockCollection_

This is a mapping of Block object attributes to selector syntax:

| Block Attribute                                                                                 | CSS-like Selector Syntax                                                                    | Status          |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | --------------- |
| opcode ([Full set of opcodes](https://github.com/LLK/scratch-vm/tree/develop/src/blocks))       | Type selector. `blocks.query('event_whenflagclicked')` or `blocks.query('control_if_else')` | implemented     |
| block type ([Full set of block types](https://en.scratch-wiki.info/wiki/Blocks#List_of_Blocks)) | Class selector. `blocks.query('.motion')` or `blocks.query('.sensing')`                     | implemented     |
| block shape ([Full set of block shapes](https://en.scratch-wiki.info/wiki/Blocks#Block_Shapes)) | Pseudo class selector. `blocks.query(':hat')` or `blocks.query(':reporter')`                | not implemented |

The selector syntax can be combined for more fine-grained filtering.

**Get all motion reporter blocks**

```
const motionReporterBlocks = blocks.query('.motion :reporter');
```

---

### Block

A _Block_ is a singleton of _BlockCollection_. It has additional methods, specific to the data held by an individual block.

**Methods**

**prop(attribute)** - _implemented_  
Parameter: _attribute_ string.
Return: _any_ value for a given attribute

Attributes available to pass: _opcode, next, parent, inputs, fields, shadow, topLevel_

**args(selector)**  
This method is similar to _query_. _args_ returns the inputs or fields (depending on the query string) of a block using one of the strings defined below. Certain query strings can return an input or a field.

A sample of selector values for the _args_ method is defined in this table:

| Inputs                                                                                                                                | Fields                                          | Both Input and Field                        |
| ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------- |
| X, Y, DURATION, MESSAGE, SECS CONDITION, SUBSTACK, OPERAND, TIMES, CHANGE, FROM, VALUE, BROADCAST_INPUT, BACKDROP, VOLUME, NUM1, NUM2 | EFFECT, BROADCAST_OPTION, VARIABLE, STOP_OPTION | COSTUME, TOUCHINGOBJECTMENU, TO, SOUND_MENU |

```
const condition = block.args('CONDITION');
const variable = block.args('VARIABLE');
const operand = block.args('OPERAND');
```

**substacks()**  
Returns the substacks for the block as an list of Objects representing a substack. If a block is not capable of having a substack, the list will be empty.

---

### AssetCollection

An _AssetCollection_ represents an interable collection of objects that represent Assets, which are static files included in an \*.sb3 file or somwhere the user designates, used for costumes and sounds.

**Methods**

**query(selector)**  
Parameter(s): A string in the CSS selector style
Return: _AssetCollection_

Possible selector syntax:

| Asset Attribute | Selector Syntax                                                                  |
| --------------- | -------------------------------------------------------------------------------- |
| name            | Attribute selector. `assets.query('name="83a9787d4cb6f3b7632b4ddfebf74367.wav")` |
| dataFormat      | Type Selector. `assets.query('wav')`                                             |

---

### Asset

An _Asset_ is a singleton of _AssetCollection_.

**Methods**

**toBuffer()**
Return: _Promise_ to the file buffer of this Asset

---

### Variable Collection

A _Variable_ collection represents a collection of local and global variables belonging to a sprite.

### Variable

---

## CLI Proposal

_Coming soon_

---

## Development

sb-util is implemented in TypeScript and will be available as a JavaScript library on [npm](https://www.npmjs.com/package/sb-util) and as a CLI tool.

## Install Dependencies

```
npm install
```

## Build

```
npm run build
```

## Run Tests

```
npm test
```

## Linting Code

This project uses [https://prettier.io](Prettier) to format code and [https://eslint.org/](ESLint) for linting (catching errors). To format and lint, run

```
npm run lint
```
