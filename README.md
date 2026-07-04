# zip-encoding-test

Contains example zip archive files, for people to test filename encoding support.

## Source files

All archives contain the same 4 empty files inside `src/`:

- `HelloWorld.txt` — "Hello World" in English
- `你好世界.txt` — "Hello World" in Chinese
- `こんにちは世界.txt` — "Hello World" in Japanese
- `©®™★♦♣♥♠€£￥✓♫.txt` — Common Unicode symbols

## Archives

### Created with old 7-zip (p7zip 17.05)

| Archive | Encoding | Notes |
|---|---|---|
| `old7z_utf8.zip` | UTF-8 | Default for p7zip |
| `old7z_cp932.zip` | CP932 (Shift-JIS) | Japanese codepage |
| `old7z_cp936.zip` | CP936 (GBK) | Chinese Simplified codepage |
| `old7z_iso8859-1.zip` | ISO 8859-1 | Western European |

### Created with new 7-zip (25.01)

| Archive | Encoding | Notes |
|---|---|---|
| `new7z_utf8.zip` | UTF-8 | Default since 7-zip 21.02 |
| `new7z_cp932.zip` | CP932 (Shift-JIS) | Japanese codepage |
| `new7z_cp936.zip` | CP936 (GBK) | Chinese Simplified codepage |
| `new7z_iso8859-1.zip` | ISO 8859-1 | Western European |

## How to create

### Old 7-zip (p7zip 17.05)

```bash
# UTF-8 (default)
7z a -tzip -mcp=0 archive.zip src/*.txt

# CP932
7z a -tzip -mcp=932 archive.zip src/*.txt

# CP936
7z a -tzip -mcp=936 archive.zip src/*.txt

# ISO 8859-1
7z a -tzip -mcp=28591 archive.zip src/*.txt
```

### New 7-zip (25.01)

```bash
# UTF-8 (default)
7z a -tzip archive.zip src/*.txt

# CP932
7z a -tzip -mcp=932 archive.zip src/*.txt

# CP936
7z a -tzip -mcp=936 archive.zip src/*.txt

# ISO 8859-1
7z a -tzip -mcp=28591 archive.zip src/*.txt
```

Note: [Since version 21.02](https://www.7-zip.org/history.txt), 7-zip writes additional OS/Unicode filename field in UTF-8 encoding to zip archives by default. Older versions (like p7zip 17.05) do not.
