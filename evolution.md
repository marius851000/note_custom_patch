# Research about how to evolve a monster via script

*most of the offset are for EU rom*

I needed to command evolution when I wanted.

The code performing an evolution from the evolution sprint (or whatever is the name I don't remember) is located at ``0x0238ac80``. I called it ``evolve_menu``. It seems to be a state based function, using some global to store it's state. I wont use those global, and instead reproduce how it work. The code that evolve the selected pokemon by itself start at offset ``0x0238b618``

Here are a description of function I used in the below patch (replacing ``Process51``, that is, ``ProcessSpecial(51, 0, 0)``, a.k.a the function that set the team to only be leader + partner).


```asm
.relativeinclude on
.nds
.arm


;US offsets
;.definelabel FnProcess52, 0x02056AB0
;.definelabel AssemblyPointer, 0x020b0a48

;EU offsets
.definelabel FnProcess51, 0x02056d48
.definelabel FnProcess52, 0x02056e2c
.definelabel AssemblyPointer, 0x020B138C
.definelabel is_named_by_specie_name, 0x020563ec
.definelabel get_name_for_specie, 0x02052a00
.definelabel copy_string, 0x020255e0
.definelabel disband_current_team, 0x02056cdc
.definelabel add_to_roster_player, 0x02056ad0
.definelabel from_assembly_to_struct1, 0x02053450
.definelabel from_struct1_to_assembly, 0x02053818 ;r0 = monster entry in assembly, r1 = the other monster entry in the other list, pointer

; due to space constraint, assume we plan to evolve only the player
;.definelabel monster_to_evolve, 0

.open "rom/arm9.bin", 0x2000000
// replace call to FnProcess51 when setting team (replaced with FnProcess51)

.org 0x02048c2c
bl FnProcess52
```

This function, called at initialisation, call the overwritten Process51. It is replaced by Process52 (set team to leader only).

```asm
// evolve the selected monster
.org FnProcess51
.area 0xE0
  push    {lr, r4}
  ldr r0, =AssemblyPointer //pointer to assembly pointer
  ldr r4, [r0] // r0 = pointer to assembly

```

Assembly pointer (at ``0x020B138C``) is a pointer to where the Chimcheno assembly data is stored.

This is a struct with the following form.

The ``monster_data_entry`` store all the monster in the Chimcheno assembly, while current_team store (seemingly as a list of 4 entry, so if you want to select another one, you may need to iter 4 time). As I'm only interested in the player here, I used only the first entry (directly pointed)

Contrary to what may be expected, the game doesn't refresh the assembly entry when a dungeon is finished. It may need to be manually synced with assembly data (or maybe there is a function that does just that).

```C++
struct AssemblyData {
    // from offset 0 to 0x9365 (excluded). An entry has a size of 0x44
    struct AssemblyEntry monster_data_entry[555];
    other_data...;
    // at 0x984c
    struct PokemonInCurrentTeam * current_team;
}

struct AssemblyEntry {
    bool active;
    uchar level;
    undefined1 field_0x2;
    undefined1 field_0x3;
    // 0x4
    short specie;
    undefined field_0x6;
    undefined field_0x7;
    undefined2 field_0x8;
    short max_hp; /* capped at 999. */
    char atk;
    char sp_atk;
    char def;
    char sp_def;
    undefined4 field_0x10;
    undefined4 field_0x14;
    undefined4 field_0x18;
    undefined4 field_0x1c;
    undefined field_0x20;
    undefined field_0x21;
    undefined field_0x22;
    undefined field_0x23;
    undefined field_0x24;
    undefined field_0x25;
    undefined field_0x26;
    undefined field_0x27;
    undefined field_0x28;
    undefined field_0x29;
    undefined field_0x2a;
    undefined field_0x2b;
    undefined field_0x2c;
    undefined field_0x2d;
    undefined field_0x2e;
    undefined field_0x2f;
    undefined field_0x30;
    undefined field_0x31;
    undefined field_0x32;
    undefined field_0x33;
    undefined field_0x34;
    undefined field_0x35;
    undefined field_0x36;
    undefined field_0x37;
    undefined field_0x38;
    undefined field_0x39;
    //0x3A
    char name[10];
};


//size: 0x68
struct PokemonInCurrentTeam {
    //0
    bool is_used?;
    undefined field_0x1;
    uchar level;
    undefined1 field_0x3;
    undefined1 field_0x4;
    undefined field_0x5;
    undefined2 field_0x6;
    //0x8
    short assembly_id;
    undefined field_0xa;
    undefined field_0xb;
    //0xc
    short specie;
    undefined field_0xe;
    undefined field_0xf;
    short field_0x10;
    undefined field_0x12;
    undefined field_0x13;
    undefined field_0x14;
    undefined field_0x15;
    undefined field_0x16;
    undefined field_0x17;
    undefined4 field_0x18;
    undefined field_0x1c;
    undefined field_0x1d;
    undefined field_0x1e;
    undefined field_0x1f;
    undefined field_0x20;
    undefined field_0x21;
    undefined field_0x22;
    undefined field_0x23;
    undefined field_0x24;
    undefined field_0x25;
    undefined field_0x26;
    undefined field_0x27;
    undefined field_0x28;
    undefined field_0x29;
    undefined field_0x2a;
    undefined field_0x2b;
    undefined field_0x2c;
    undefined field_0x2d;
    undefined field_0x2e;
    undefined field_0x2f;
    undefined field_0x30;
    undefined field_0x31;
    undefined field_0x32;
    undefined field_0x33;
    undefined field_0x34;
    undefined field_0x35;
    undefined field_0x36;
    undefined field_0x37;
    undefined field_0x38;
    undefined field_0x39;
    undefined field_0x3a;
    undefined field_0x3b;
    undefined field_0x3c;
    undefined field_0x3d;
    //0x3e
    struct astruct_15 some_inner_struct;
    undefined field_0x44;
    undefined field_0x45;
    undefined field_0x46;
    undefined field_0x47;
    undefined field_0x48;
    undefined field_0x49;
    undefined field_0x4a;
    undefined field_0x4b;
    undefined4 field_0x4c;
    undefined4 field_0x50;
    undefined4 field_0x54;
    undefined field_0x58;
    undefined field_0x59;
    undefined field_0x5a;
    undefined field_0x5b;
    undefined field_0x5c;
    undefined field_0x5d;
    //0x5e
    char name[10];
};
```

```asm
  ; load the player monster from the more permanent memory
  mov r0, #0
  ldr r1, =AssemblyPointer //pointer to assembly pointer
  ldr r1, [r1]
  ldr r2, =0x984c
  add r1, r1, r2
  ldr r1, [r1]
  bl from_struct1_to_assembly
```
 
This ``from_struct1_to_assembly`` load the dungeon data (including level-up) into the assembly data. It is located at ``0x02053818``.

input r0 (``monster_id``) is id of the monster in the assembly (here, 0 for the player)

input r1 is the pointer to the dungeon monster data.

```asm
  ; change the name if appropriate
  mov r0, r4
  bl is_named_by_specie_name
```
``is_named_by_specie_name`` (at ``0x020563ec``) is a function that take (as r0) a pointer to an ``AssemblyEntry`` (here, the player -- aka the first entry), and return (on r0) true if the pokemon is named from the same name as it's current specie. This is a simple comparaison with the specie name that is already loaded in RAM.

```asm
  ; evolve the pokemon in itself
  ; r1/monster_specie_ptr = &monster->specie
  add r1, r4, 0x4
  ; r2/specie = *r1/monster_specie_pointer
  ldrh r2, [r1]
  ; r2/specie += 1
  add r2, 1
  ; *r1/monster_specie_ptr = r2/specie
  strh r2, [r1]
```
above code increase the specie id of the monster in the assembly entry (Offset ``0x4`` of an ``AssemblyEntry``)


```asm
  cmp r0, 0
  mov r0, r2
  beq after_name_change
  ; reuse parameter r0 from above
  bl get_name_for_specie
```

This part first decide if the name should be changed or not (from r0, aka result of ``is_named_by_specie_name``)

It then call ``get_name_for_specie`` (``0x02052a00``)
- ``in_r0`` is the monster id
- ``out_r0`` is a pointer to the name of the monster

```asm
  mov r1, r0
  add r0, r4, 0x3a
  mov r2, 10
  bl copy_string
  after_name_change:
```

copy the monster name to the ``0x3a`` offset of an ``AssemblyEntry`` (aka, it's name).

``copy_string`` (``0x020255e0``) :
- ``in_r0:char *`` (``to``) : target pointer to where to write the string
- ``in_r1:char *`` (``from``) : pointer to the source string
- ``in_r2:uint`` (``size``) : the max string length

```asm
  ; change stats
  ; base stats are just increased by 5
  ; r0/loop_counter = 0
  mov r0, 0
  stat_change:
  ; r1/pointer_to_stat = (int)(&monster->base_atk) + r0/loop_counter
  mov r1, r4
  add r1, r1, 0xC
  add r1, r1, r0
  ; r2/stat_value = *r1/pointer_to_stat
  ldrb r2, [r1]
  ; r2/stat_value += 5
  add r2, r2, 5
  ; *r1/pointer_to_stat = r2/stat_value
  ; no need to cap, as the max value is 0xff
  strb r2, [r1]
  ; loop_counter += 1
  add r0, r0, 1
  ; continue the loop if necessary
  cmp r0, 6
  bne stat_change


  ; change pv
  add r0, r4, 0xA
  ldrh r1, [r0]
  add r1, 10
  strh r1, [r0]
```

The above code change the ``AssemblyEntry`` for the player, to have the correct level up data. Base stats (attack, special attack, defense, sepcial defense. Those four are stored continuously in memory) increase of ``5``, PV of ``10``

```asm


  ; from assembly to the more permanent struct
  ldr r0, =AssemblyPointer //pointer to assembly pointer
  ldr r0, [r0] // r0 = pointer to assembly
  ldr r1, =0x984c
  add r0, r0, r1
  ldr r0, [r0]

  mov r1, 0 ;0->monster_to_evolve
  bl from_assembly_to_struct1
```

The above code store the assembly entry to dungeon pokemon data.

``from_assembly_to_struct1_wrapper`` (actually called ``from_assembly_to_struct1`` in the patch) (``0x02053450``):
- ``r0_in:PokemonInCurrentTeam *``: the target pokemon dungeon data
- ``r1_in:uint`` (``assembly_id``): the id of the source monster in the assembly table


```asm
  pop     {pc, r4}
  .pool
.endarea

.close
```