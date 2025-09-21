# zip-encoding-test
Contains example zip archive files, for people to test filename encoding support.

## Example info

- `test1.zip`: encoded in codepage 932, containing an empty file `t氏の話を信じるな.txt`. Will be displayed as `t巵偺榖傪怣偠傞側.txt` when encoding errors.
- `test2.zip`: encoded in codepage 932, containing an empty file `の.txt`. Will be displayed as `偺.txt` when encoding errors.

## How to create an example zip archive

Take `test1.zip` as example:
- Create an empty file named `t氏の話を信じるな.txt`.
- Install 7-zip **which version earlier than 21.02**, for example 19.00.
- Run command:
```
7z a -tzip -mcp=932 test1.zip t氏の話を信じるな.txt
```
