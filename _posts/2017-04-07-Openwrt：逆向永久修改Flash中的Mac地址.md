---
layout: post
tags: [Openwrt]
comments: true
---

# get mtd device

cat /proc/mtd

```
dev:  size   erasesize  name
mtd0: 00800000 00010000 "ALL"
mtd1: 00030000 00010000 "Bootloader"
mtd2: 00010000 00010000 "Config"
mtd3: 00010000 00010000 "Factory"
mtd4: 007b0000 00010000 "firmware"
mtd5: 0067ac57 00010000 "rootfs"
mtd6: 003a0000 00010000 "rootfs_data"
```

size is 0x00010000  = 65536 = 512*1024

# how to use hexdump

hexdump -help

```
hexdump: invalid option -- h
BusyBox v1.22.1 (2017-02-25 15:19:37 CST) multi-call binary.

Usage: hexdump [-bcCdefnosvx] [FILE]...

Display FILEs (or stdin) in a user specified format

        -b              One-byte octal display
        -c              One-byte character display
        -C              Canonical hex+ASCII, 16 bytes per line
        -d              Two-byte decimal display
        -e FORMAT_STRING
        -f FORMAT_FILE
        -n LENGTH       Interpret only LENGTH bytes of input
        -o              Two-byte octal display
        -s OFFSET       Skip OFFSET bytes
        -v              Display all input data
        -x              Two-byte hexadecimal display
```

just use hexdump -C "FILE" to observe file content



# Observe mtd3 file

hexdump -C  /dev/mtd3

```
00000000  28 76 06 00 60 08 71 85  5d 73 00 00 00 00 00 00  |(v..`.q.]s......|
00000010  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000020  00 00 00 00 20 00 00 00  60 08 71 85 5d 73 60 08  |.... ....q.]s.|
00000030  71 85 5d 71 11 34 00 20  ff ff 00 01 00 00 00 00  |q.]q.4. ........|
00000040  00 00 22 00 00 00 00 00  30 00 00 00 00 00 00 00  |..".....0.......|
00000050  82 00 00 94 40 b2 c0 ca  21 83 82 81 40 ca 21 80  |....@...!...@.!.|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000000a0  c6 c6 c6 c4 c4 c0 c0 c6  c4 c6 c4 c4 c0 c0 00 00  |................|
000000b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000000e0  11 1d 11 1d 1c 35 1c 35  1e 35 1e 35 17 19 17 19  |.....5.5.5.5....|
000000f0  02 00 00 00 d8 80 80 88  00 00 00 00 00 00 00 00  |................|
00000100  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 77 00  |..............w.|
00000130  11 1d 11 1d 15 7f 15 7f  17 7f 17 7f 10 3b 10 3b  |.............;.;|
00000140  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00000200  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000400  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00010000
```

Mac: 60 08 71 85  5d 73

```
00000000  28 76 06 00 60 08 71 85  5d 73 00 00 00 00 00 00
```

# Copy file from mtd3

shell
```
dd if=/dev/mtd3 of=/tmp/test_1.bin bs=512 count=1024 conv=sync
```
shell
```
hexdump -C test_1.bin
```

offset is:

0x00000000+4 = 4

0x00000020+9 = 41

```
00000000  28 76 06 00 60 08 71 85  5d 73 00 00 00 00 00 00  |(v..`.q.]s......|
00000010  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000020  00 00 00 00 20 00 00 00  60 08 71 85 5d 73 60 08  |.... ...`.q.]s`.|
00000030  71 85 5d 71 11 34 00 20  ff ff 00 01 00 00 00 00  |q.]q.4. ........|
00000040  00 00 22 00 00 00 00 00  30 00 00 00 00 00 00 00  |..".....0.......|
00000050  82 00 00 94 40 b2 c0 ca  21 83 82 81 40 ca 21 80  |....@...!...@.!.|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000000a0  c6 c6 c6 c4 c4 c0 c0 c6  c4 c6 c4 c4 c0 c0 00 00  |................|
000000b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000000e0  11 1d 11 1d 1c 35 1c 35  1e 35 1e 35 17 19 17 19  |.....5.5.5.5....|
000000f0  02 00 00 00 d8 80 80 88  00 00 00 00 00 00 00 00  |................|
00000100  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 77 00  |..............w.|
00000130  11 1d 11 1d 15 7f 15 7f  17 7f 17 7f 10 3b 10 3b  |.............;.;|
00000140  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00000200  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000400  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00010000

```


CGI code
```
#!/bin/sh
[ "$REQUEST_METHOD" = "GET"] && read QUERY_STRING
echo "Content-type: text/html;charset=UTF-8"
echo "posted data is $QUERY_STRING" >&2
echo
echo "<HTML><BODY>"
echo "<CENTER>Today is:</CENTER>"
echo "<CENTER><B>"
date
echo "</B></CENTER>"
#echo "Mac Address"
echo "<br>"
echo "<center>please input mac such as:FF.FF.FF.FF.FF.FF</center>"
echo "<br>"
echo "<center>Mac Address:<input type=\"text\" name=\"firstname\" id=\"mac\" value="">"
echo ""
echo "<input type=\"button\" value=\"OK\" id=\"btm\" onclick=\"getValue()\"></center>"
echo ''
echo "<script>"
echo "function getValue() {"
echo "var msg = document.getElementById(\"mac\").value;"
#echo "document.getElementById(\"mac\").innerHTML=msg;"
echo "alert(\"Ready to write Mac \" + msg + \" to camera\")"
echo "}"
echo "</script>"

echo "</BODY></HTML>"
```


Final

get binary file "test_1.bin" from mtd device

shell
```
dd if=/dev/mtd3 of=test_1.bin ibs=512 obs=512 count=1024 skip=0 seek=0 conv=notrunc
```

write mac address to temp.bin

shell   
```
echo -e -n "\x60\x08\x71\x85\x5d\x70" > temp.bin
```

overwrite file "temp.bin" to test_1.bin, offset is 4 and 41

shell
```
dd if=temp.bin of=test_1.bin ibs=1 obs=1 count=6 skip=0 seek=4 conv=notrunc
```
shell   
```
dd if=temp.bin of=test_1.bin ibs=1 obs=1 count=6 skip=0 seek=41 conv=notrunc
```

