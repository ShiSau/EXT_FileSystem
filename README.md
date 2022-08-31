# EXT_FileSystem
<BR>
  
### Brief Intro
> Ext uses indexing! Index nodes (“inodes”) which contain many pointers to file chunks. I will be implementing a rough implementation of Ext


  <br>
  
####  Block Implementation in code

Block 0: Superblock
* Just contains the
disk label
* Null terminated


Block 1: Bitmaps
* 256 bytes (2048 bits)
for inodes
* 256 bytes (2048 bits)
for data
* 0x80 = 1 0 0 0 0 0 ...
– Root directory entry


Blocks 2-65 (64 total): inodes
*  0x1111 = TYPE_DIR
*  0x0001 = links
*  0x00000000 = size
*  Directs[3]:
– 0x0000: inode 0
– Unused
– Unused
* Indirects: unused
0x2222 = TYPE_FILE
* 0x0001 = links
* 0x00000BB9 = size (3001)
*  Directs[3]:
– 0x0001: inode 1
– 0x0002: inode 2
– 0x0003: inode 3
* Indirects: 0x0004
– 4th data block (#69) contains an
array of 2-byte inode numbers

Blocks 66-2113 (2048 total): data
* Entry 1
– 0x0000: inode 0
– 0x2e 0x00: “.”
* Entry 2
– 0x0000: inode 0
– 0x2e 0x2e 0x00: “..”
* Entry 3
– 0x0001: inode 1
– 0x67 0x6f ... : “lots_of_A.txt”
* Entry 4 - 15
– 0xFFFF: unused

The code implements dbrowse function with the following working options: 
* dir 
* cd <dir>
* read <file>
* pwd
* stat
* help
