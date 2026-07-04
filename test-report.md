# ZIP Encoding Test Report

## Environment

| Tool | Version |
|------|---------|
| `7zip-zstd` | 7-Zip 25.01 ZS v1.5.7 R4 (x64), 2026-01-10 |
| `p7zip` | 7-Zip [64] 17.05 / p7zip 17.05, 2017-08-28 |
| `unzip` | UnZip 6.00 (20 Feb 2010), Info-ZIP |

System locale: `zh_CN.UTF-8`

## Samples

### Files Inside Archives
The test archives each contain 4 files:

| File | Description |
|------|-------------|
| `HelloWorld.txt` | ASCII-only filename |
| `©®™★♦♣♥♠€£￥✓♫.txt` | Western symbols (UTF-8: 44 bytes) |
| `こんにちは世界.txt` | Japanese, "Hello World" |
| `你好世界.txt` | Chinese, "Hello World" |

### Archiving Tools

Eight ZIP archives were created, split into two groups based on the archiving tool used:

- `new7z_*.zip` : Created with `7zip-zstd` (7-Zip 25.01 ZS).
- `old7z_*.zip` : Created with Windows 7-Zip 19.00 ([download from SourceForge](https://sourceforge.net/projects/sevenzip/files/7-Zip/19.00/)).

### Filename Encoding Options

- `*_utf8.zip` : Archived with argument `-mcp=0`.
- `*_cp932.zip` : Archived with argument `-mcp=932`.
- `*_cp936.zip` : Archived with argument `-mcp=936`.
- `*_iso8859-1.zip` : Archived with argument `-mcp=28591`.

## Results

### Listing

| ZIP File | Filename Encoding | 7z (25.01) Listing | p7zip (17.05) Listing | UnZip (6.00) Listing |
|----------|-------------------|--------------------------|----------------------|---------------------|
| `new7z_cp932.zip` | UTF-8 bytes | Correct | Correct | `?????????????.txt` |
| `new7z_cp936.zip` | UTF-8 bytes | Correct | Correct | `???????.txt` |
| `new7z_iso8859-1.zip` | UTF-8 bytes | Correct | Correct | `?????????????.txt` |
| `new7z_utf8.zip` | UTF-8 bytes | Correct | Correct | `?????????????.txt` |
| `old7z_cp932.zip` | EUC-JP | `����ɂ��͐��E.txt` garbled | `偙傫偵偪偼悽奅.txt` EUC-JP raw | `???E.txt` + question marks |
| `old7z_cp936.zip` | GBK | `����ˤ�������.txt` garbled | `こんにちは世界.txt` lucky match | `?????-?-??-+??.txt` |
| `old7z_iso8859-1.zip` | ISO-8859-1 | Correct | Correct | `???????.txt` (extracts correctly) |
| `old7z_utf8.zip` | UTF-8 | `����ˤ�������.txt` garbled | `こんにちは世界.txt` correct | `?????-?-??-+??.txt` |

### Extraction

| ZIP File | 7z (25.01) Extraction | p7zip (17.05) Extraction | UnZip (6.00) Extraction |
|----------|-----------------------------|-------------------------|------------------------|
| `new7z_cp932.zip` | Correct | Correct | Correct |
| `new7z_cp936.zip` | Correct | Correct | Correct |
| `new7z_iso8859-1.zip` | Correct | Correct | Correct |
| `new7z_utf8.zip` | Correct | Correct | Correct |
| `old7z_cp932.zip` | `����ɂ��͐��E.txt` garbled | `偙傫偵偪偼悽奅.txt` EUC-JP raw | `???E.txt` question marks + warnings |
| `old7z_cp936.zip` | `����ˤ�������.txt` garbled | `こんにちは世界.txt` correct | `?????-?-??-+??.txt` |
| `old7z_iso8859-1.zip` | Correct | Correct | Correct (listing shows `???`) |
| `old7z_utf8.zip` | `����ˤ�������.txt` garbled | `こんにちは世界.txt` correct | `?????-?-??-+??.txt` |

### ZIP Header Analysis

| Aspect | `new7z_*` | `old7z_*` |
|--------|---------|---------|
| UTF-8 flag (GP bit 3) | Set on non-ASCII entries | Not set on any entry |
| Filename encoding | UTF-8 bytes | Native encoding (EUC-JP, GBK, ISO-8859-1, or UTF-8) |
| Path prefix | `src/` | None |
| Central directory GP flag | 0x00 | 0x00 |

Note: Although the new7z archives set the UTF-8 flag on some local file headers, the central directory GP flag is 0x00 for all archives (both new7z and old7z). The UTF-8 flag in the central directory is also 0x00.

## Key Findings

### 1. `new7z_*`

All three tools correctly extract filenames from the new7z archives, despite the UTF-8 flag being unset in the central directory. This is because the filenames are stored as UTF-8 encoded bytes, and both 7zip-zstd and p7zip detect this correctly through heuristic analysis.

However, **UnZip 6.00 shows `????` for all non-ASCII filenames during listing when extracting**, even though it successfully writes the correct filenames to disk. This is a display limitation, i.e. unzip 6.00 does not perform UTF-8 detection on filenames without the UTF-8 flag set.

### 2. 7-Zip (7zip-zstd 25.01)

- Correctly handles all `new7z_*` archives both on listing and extraction.
- Correctly handles `old7z_iso8859-1.zip` by applying locale-based fallback decoding.
- Fails on `old7z_utf8.zip` and `old7z_cp932.zip`: the UTF-8 flag is not set in the central directory, so 7-Zip applies locale-based decoding (zh_CN.UTF-8) to the raw bytes, resulting in garbled output for non-UTF-8 encoded filenames.

### 3. p7zip (17.05)

- Correctly handles all `new7z_*` archives — both listing and extraction produce correct filenames.
- Correctly handles `old7z_iso8859-1.zip`.
- **Correctly handles `old7z_utf8.zip`**: p7zip's heuristic decodes the raw bytes as UTF-8, matching the actual encoding. This is notable because the UTF-8 flag is not set in the central directory.
- On `old7z_cp936.zip`, p7zip also produces correct output — the Japanese filename happens to be stored as UTF-8 bytes in this archive.
- On `old7z_cp932.zip`, p7zip decodes as EUC-JP, producing `偙傫偵偪偼悽奅` — the raw EUC-JP byte representation of `こんにちは世界`. While not the UTF-8 rendered form, the EUC-JP text is still readable on a Japanese system.

### 4. UnZip (6.00)

- Shows `????` for all non-ASCII filenames in listing across all archives, because unzip 6.00 does not perform locale-based fallback encoding detection.
- On `new7z_*` archives, listing shows `????` but extraction writes correct filenames to disk.
- On `old7z_iso8859-1.zip`, listing shows `????` but extraction produces correct filenames — unzip uses the system locale for extraction even if listing shows question marks.
- On `old7z_utf8.zip` and `old7z_cp932.zip`, both listing and extraction produce garbled/question-mark filenames.
- Emits "mismatching local filename" warnings during extraction for all `old7z_*` archives, indicating a mismatch between the local header and central directory entries.

### 5. Summary

*Note again that the locale is `zh_CN.UTF-8`.*

| Scenario | Extracting (w/o Listing) | Listing |
|----------|-----------|-----------|
| with UTF-8 flag (`new7z_*`) | p7zip = 7-zip = unzip | p7zip = 7-zip > unzip |
| ISO-8859-1 (`old7z_iso8859-1.zip`) | p7zip = 7-zip = unzip | p7zip = 7-zip > unzip |
| UTF-8 w/o flag (`old7z_utf8.zip`) | p7zip > 7-zip = unzip | p7zip > 7-zip = unzip |
| EUC-JP (`old7z_cp932.zip`) | p7zip > 7-zip = unzip | p7zip > 7-zip = unzip |
| GBK (`old7z_cp936.zip`) | p7zip > 7-zip = unzip | p7zip > 7-zip = unzip |

**Recommendation**:
- Archiving: For maximum compatibility, always set the UTF-8 flag when creating ZIP archives. Both the latest `7-zip` and `p7zip` do this by default.
- Extracting: When dealing with legacy ZIP archives without the UTF-8 flag, p7zip 17.05 tends to have more robust heuristic-based encoding detection compared to both modern 7-Zip and UnZip 6.00. However, p7zip has seen little development after 17.05, and 7-zip are generally a better option considering many other aspects on Linux.
