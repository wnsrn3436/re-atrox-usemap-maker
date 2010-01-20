# Atrox UseMap Maker

Atrox ships with a map editor, but there is no way to load a map you made into the game. This tool takes the game's resource archive, the pak file, apart at the byte level to find where every campaign mission map is stored, then overwrites those slots with user maps so they can be played from the mission select screen.

**Compatible version**: Atrox v1.10 build 1


## How to use

Download from Releases, extract it, put your map files in the `UseMap` folder and run it. No window appears, and it closes on its own when done.

The file name decides which mission slot the map goes into.

| Name | Faction |
|---|---|
| `h_1` to `h_9` | Hominian |
| `c_1` to `c_9` | Creatise |
| `i_1` to `i_8` | Intelion |

It looks for `atrox.pak` in the default install path. If it is not there, a file dialog opens, so pick `dev\atrox.pak` inside the Atrox install folder.

Then start Atrox and pick that mission from the mission select screen to play your map.

To move on to the next mission after your map ends, set the map editor trigger (Action > set the map used by the next stage) to the path of the original mission file.

| Faction | Path |
|---|---|
| Hominian | `Scenario\hominian01.spm`, `hominian02a.spm`, `hominian02b.spm`, `hominian03.spm` to `hominian08.spm` |
| Creatise | `Scenario\createse01.spm` to `createse09.spm` |
| Intelion | `Scenario\intelion01.spm` to `intelion08.spm` |

It edits the pak file in place, so back up `atrox.pak` before running it.


## How it works

**The mission map positions inside the pak were found and tabulated.** The pak was analysed at the byte level to learn where each of the 26 campaign missions across the three factions is stored, and those values are hard coded into one script per faction. Feed in a mission number and it returns the offset.

```gml
// sk_h.gml - Hominian
switch(argument0)
{
case 1: return 85598720
case 2: return 86157824
case 3: return 86743552
...
}
```

**Overwriting is seeking to that position and pushing bytes in.** The pak is opened for writing, the cursor moves to the offset, and the user map is read byte by byte and written straight in. Nothing in the file structure is rewritten and no index is updated, only the slot is swapped.

```gml
for(i=1; file_exists("usemap\h_"+string(i)+".spm"); i+=1)
{
  open1 = file_bin_open(folder, 2)
  open2 = file_bin_open("usemap\h_"+string(i)+".spm", 0)

  file_bin_seek(open1, sk_h(i))
  for(j=0; j!=file_bin_size(open2); j+=1)
  {
    file_bin_write_byte(open1, file_bin_read_byte(open2))
  }

  file_bin_close(open1)
  file_bin_close(open2)
}
```

**The pak is looked for in the default install path first.** If it is not there, a file dialog opens so you can pick it. With no screen and no buttons, the whole thing lives in the Create event of a single object.


## Files

| Path | Contents |
|---|---|
| `source/atrox-usemap-maker.gmk` | Original project file |
| `source/split/` | Text tree produced by GmkSplitter |
| Releases | Executable, usage notes and an example map |


## License

zlib. See [LICENSE](LICENSE).
