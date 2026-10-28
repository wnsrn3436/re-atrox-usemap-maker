# Atrox UseMap Maker

[한국어](README.ko.md)

Atrox ships with a map editor, but there is no way to load a map you made into the game. So I took apart the game's resource archive, the pak file, at the byte level and found every location where a campaign mission map is stored. This tool overwrites those slots with user maps so you can play use maps from the mission select screen.

**Compatible version**: Atrox v1.10 build 1


## How to use

Download it from Releases, extract it, put your map files in the `UseMap` folder, and run it. It opens no window and closes on its own when it is done.

The file name decides which mission slot the map goes into.

| Name | Faction |
|---|---|
| `h_1` to `h_9` | Hominian |
| `c_1` to `c_9` | Creatise |
| `i_1` to `i_8` | Intelion |

It looks for `atrox.pak` in the default install path. If it is not there, a file dialog opens, so pick `dev\atrox.pak` inside the Atrox install folder.

Then start Atrox and pick that mission on the mission select screen to play the map you put in.

To move on to the next mission after your map ends, enter the path of the original mission file in the map editor trigger (Action > set the map used by the next stage).

| Faction | Path |
|---|---|
| Hominian | `Scenario\hominian01.spm`, `hominian02a.spm`, `hominian02b.spm`, `hominian03.spm` to `hominian08.spm` |
| Creatise | `Scenario\createse01.spm` to `createse09.spm` |
| Intelion | `Scenario\intelion01.spm` to `intelion08.spm` |

It edits the pak file in place, so it is a good idea to back up `atrox.pak` before running it.


## How it works

**I found where the mission maps sit inside the pak and made a table of them.** Analysing the pak at the byte level showed the byte position where each of the 26 campaign missions across the three factions is stored, and those values are hard coded into one script per faction. Feed in a mission number and it returns the offset.

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

**To overwrite, it jumps to that position and pushes the bytes in.** It opens the pak in write mode, moves to the offset, then reads the user map file one byte at a time and writes each byte straight in. It does not rewrite the file structure or update any index. It only swaps out what sits in that slot.

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

**It looks for the pak in the default install path first.** If it is not there, a file dialog opens so you can pick it yourself. The program has no screen or buttons and just runs to the end, so all of it sits in the Create event of a single object.


## Files

| Path | Contents |
|---|---|
| `source/atrox-usemap-maker.gmk` | Original project file |
| `source/split/` | Text tree produced by GmkSplitter |
| Releases | Executable, usage notes and one example map |


## License

zlib. See [LICENSE](LICENSE).
