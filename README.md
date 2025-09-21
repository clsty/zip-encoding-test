# zip-encoding-test
Contains example zip archive files, for people to test filename encoding support.

## Example info

- `cp936.test1.zip`: containing an empty file `胭脂.txt`. Will be displayed normally (?) when forcing UTF-8.
- `cp936.test2.zip`: containing an empty file `新建文本文档.txt`. Will be displayed normally (?) when forcing UTF-8.
- `cp932.test1.zip`: containing an empty file `t氏の話を信じるな.txt`. Will be displayed as `t巵偺榖傪怣偠傞側.txt` when encoding errors (forcing UTF-8).
- `cp932.test2.zip`: containing an empty file `の.txt`. Will be displayed as `偺.txt` when encoding errors (forcing UTF-8).

P.S. My system local encoding is in `cp936`, I guess that's why the `cp936` archives seems normal when forcing UTF-8.

## How to create an example zip archive

Take `cp932.test1.zip` as example:
- Create an empty file named `t氏の話を信じるな.txt`.
- Install 7-zip **which version earlier than 21.02**, for example [19.00](https://sourceforge.net/projects/sevenzip/files/7-Zip/19.00/).
- Run command:
```
7z a -tzip -mcp=932 cp932.test1.zip t氏の話を信じるな.txt
```
For Windows you may need to use something like `"C:\Program Files (x86)\7-Zip\7z.exe"` instead of `7z`, unless `7z` is available by setting the `PATH` envvar.

Note: [since version 21.02](https://www.7-zip.org/history.txt), 7-zip writes additional field for filename in UTF-8 encoding to zip archives.
Other archive utils based on 7-zip maybe also have changed this behavior simultaneously.
