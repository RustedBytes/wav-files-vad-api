# wav-files-vad-api

A command-line tool for recursively performing Voice Activity Detection (VAD) on WAV audio files using one or more external API servers. It validates files against a specific format (mono channel, 16-bit PCM, 16kHz sample rate), processes them in parallel, preserves the input directory structure in the output, and provides summary statistics on completion.

## Features

- **Recursive Scanning**: Walks the input directory tree to find all `.wav` files using `walkdir`.
- **Format Validation**: Ensures WAV files meet the required specs (mono, 16-bit PCM, 16kHz) using the `hound` crate.
- **Parallel Processing**: Leverages `rayon` to process files concurrently, with the degree of parallelism matching the number of provided API servers.
- **API Integration**: Distributes load by sending JSON requests to a list of external VAD APIs via `ureq` and handles responses.
- **Robust Error Handling**: Uses `anyhow` for contextual error propagation and clear logging.
- **Directory Preservation**: Mirrors the input folder structure in the output directory.
- **CLI-Friendly**: Built with `clap` for intuitive argument parsing and help output.

## Prerequisites

- Rust 1.75+ (stable channel, due to `2024` edition)
- An external API server running at the specified address(es), accepting POST requests with JSON payloads for VAD.
  - Expected request body: `{ "input_file": String, "output_dir": String, "model": Option<String> }`
  - Expected success response: HTTP status `200 OK`.

## Installation

### From GitHub Releases

Statically-linked Linux binaries are available for download from the Releases page.

### From Source

1.  Clone the repository:
    ```bash
    git clone https://github.com/RustedBytes/wav-files-vad-api.git
    cd wav-files-vad-api
    ```

2.  Build the project:
    ```bash
    cargo build --release
    ```

    The binary will be available at `target/release/wav-files-vad-api`.

## Usage

Run the tool with the required input and output directories, and at least one API server address.

```bash
wav-files-vad-api /path/to/input/dir /path/to/output/dir --addr-api http://localhost:8000/vad
```

### Arguments

-   `INPUT_DIR`: Path to the directory containing WAV files (scanned recursively).
-   `OUTPUT_DIR`: Path to the directory where VAD output files will be saved (created if it doesn't exist).
-   `--addr-api <ADDR_API>`: A comma-separated list of VAD API server URLs. Work will be distributed among them. (Required)
-   `--model <MODEL>`: An optional model name to pass to the VAD API.

### Example

Process all valid WAV files in `./raw_audio/` and save results to `./processed_audio/` using two local API servers for parallel execution:

```bash
./target/release/wav-files-vad-api ./raw_audio ./processed_audio --addr-api http://127.0.0.1:8001/vad,http://127.0.0.1:8002/vad
```

Example output:
```
Skipping invalid WAV file: ./raw_audio/unsupported_format.wav
Error processing ./raw_audio/corrupted.wav: Failed to open WAV file: ./raw_audio/corrupted.wav
VAD failed for ./raw_audio/no_speech.wav: API returned status 500
VAD complete: 42 files processed, 3 skipped.
```

## Dependencies

This tool relies on the following crates (as defined in `Cargo.toml`):

| Crate | Purpose | Version |
|---|---|---|
| `anyhow` | Contextual error handling | `1.0` |
| `clap` | CLI argument parsing | `4.5` |
| `hound` | WAV file reading and validation | `3.5` |
| `rayon` | Data parallelism | `1.11` |
| `serde` | JSON serialization/deserialization | `1.0` |
| `ureq` | HTTP client for API requests | `3.1` |
| `walkdir` | Recursive directory traversal | `2.5` |

## Contributing

1.  Fork the repo.
2.  Create a feature branch (`git checkout -b feature/my-feature`).
3.  Commit changes (`git commit -am 'Add my feature'`).
4.  Push to the branch (`git push origin feature/my-feature`).
5.  Open a Pull Request.

Please ensure code is formatted with `cargo fmt` before submitting.

## License

This project is licensed under the MIT License - see the LICENSE file for details.


## Cite

```
@software{Smoliakov_Wav_Files_Toolkit,
  author = {Smoliakov, Yehor},
  month = oct,
  title = {{WAV Files Toolkit: A suite of command-line tools for common WAV audio processing tasks, including conversion from other formats, data augmentation, loudness normalization, spectrogram generation, and validation.}},
  url = {https://github.com/RustedBytes/wav-files-toolkit},
  version = {0.4.0},
  year = {2025}
}
```

