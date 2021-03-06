TextPet
=======
A plugin-based binary script extraction and insertion tool for retro game modding and translation.

NOTE: This is still very much a work in progress. You've been warned!

What is TextPet?
----------------
Retro games usually store their scripts in a binary format, mixing text and control code bytes that tell the game what to do -- opening a text box, waiting for a button press, printing a variable. TextPet is a command line extraction and insertion tool for such scripts. It is geared towards games with complex scripting systems, and focuses first and foremost on properly parsing script commands.

Scripts, in TextPet, are contained in collections known as **text archives**. A text archive in binary format starts with a list of relative pointers to the scripts that follow.

TextPet is primarily written for the Mega Man Battle Network series of games, but through its powerful plugin system, it can support a wide range of games. All script commands are defined in external configuration files, meaning adding support for a new game should just come down to documenting its script commands and specifying them in a TextPet plugin file.

Features
--------
Some of the features that TextPet provides include:

* Support table files encoded in UTF-8.
* Extract and insert scripts in plain-text TPL (TextPet Language) format.
* Extract and insert text from script files.
  * Certain script commands can be included in the text output, for instance: `<printMoney>`.
  * If the new text is longer than the original, extra text boxes can be added semi-automatically.
* Extract and insert scripts straight from/to a ROM file, updating pointers if the new scripts are larger than the original.
* Auto-detect the scripting language used for games that use multiple scripting languages.
* Support user-defined aliases for values, for instance: `0 = false`, `1 = true`.

Usage
-----
TextPet is a command line tool. It takes a number of commands and executes them one by one. You can specify all the commands on the command line, or you can place them in an external script file and run that instead.

```
TextPet.exe <command> <command> <command> ...
```
Or simply:
```
TextPet.exe run-script script.tps
```

For the full list of commands, run `TextPet.exe help`.

Credits
-------

* **Greiga Master** - Initial plugins for MMSF3 support.

Example
-------
Here is a (very) simple example of how you can use TextPet in your project.

### Input

**plugins\ascii.tbl**
```
0A=\n
20= 
21=!
2E=.
30=0
31=1
32=2
...
41=A
42=B
43=C
...
61=a
62=b
63=c
...
```

**plugins\games.ini**
```
[Game]
name = ex
full = Example Game
cdbs = ex-cmds
tblf = ascii
vals = mugshots

[TableFile]
name=mugshots
00=None
01=MegaMan
02=ProtoMan
```

**plugins\commands.ini**
```
[CommandDatabase]
name = ex-cmds
splt = keyWait clearMsg		// text box splitter

[Command]
name = end
mask = FF
desc = Closes any open message box and ends script execution.
base = E5
ends = always

[Command]
name = keyWait
mask = FF 00
desc = Pauses script execution until a button is pressed.
base = E6

[Parameter]
name = any
offs = 1
bits = 8
desc = If true, any button can be pressed; otherwise, only A and B will have an effect.
valn = bool

[Command]
name = clearMsg
mask = FF
desc = Clears any currently open message box and resets the text printer position.
base = F1

[Command]
name = mugshotShow
mask = FF FF 00
desc = Displays a mugshot in the current message box.
base = F4 00
mugs = mugshot

[Parameter]
name = mugshot
offs = 2
bits = 8
desc = The mugshot to use.
valn = mugshots
```

**in\800000.msg**
```
02 00 F4 00 01 48 65 6C 6C 6F 20 77 6F 72 6C 64
21 E6 00 F1 F4 00 02 48 65 6C 6C 6F 20 4D 65 67
61 4D 61 6E 2E 0A 54 68 69 73 20 69 73 20 61 20
6E 65 77 20 6C 69 6E 65 2E E6 00 E5
```

**script.tps**
```
load-plugins plugins\
game ex
read-text-archives in\ -f bin
write-text-archives out\ -f tpl
```

### Output
Run with:
```
TextPet.exe run-script script.tps
```
Produces the following file:

**out\800000.tpl**
```
@archive 800000
@size 1

script 0 ex-cmds {
	mugshotShow
		mugshot = MegaMan
	"Hello world!"
	keyWait
		any = false
	clearMsg
	mugshotShow
		mugshot = ProtoMan
	"""
	Hello MegaMan.
	This is a new line.
	"""
	keyWait
		any = false
	end
}
```