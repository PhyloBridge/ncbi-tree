# ncbi-tree

[![PyPI version](https://badge.fury.io/py/ncbi-tree.svg?v=1)](https://badge.fury.io/py/ncbi-tree)
[![Downloads](https://img.shields.io/pypi/dm/ncbi-tree)](https://pypistats.org/packages/ncbi-tree)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

**ncbi-tree** is an open source, cross-platform command-line tool for downloading the latest NCBI taxonomy database and converting it to Newick tree format (.tre), with optional plain-text visualization (.txt).

## Why ncbi-tree?

[NCBI Taxonomy](https://www.ncbi.nlm.nih.gov/taxonomy) is indispensable for taxonomic-related studies and many bioinformatics applications. However, one critical downside of NCBI Taxonomy is that it does not provide any tree format (.tre), which is essential for many downstream analyses and dataset processing.

Perfect for bioinformatics researchers who want to:

- Get a Newick tree (.tre) representation of NCBI Taxonomy
- Visualize NCBI Taxonomy using existing .tre explorers, such as [ETE4](https://github.com/etetoolkit/ete4) with the command `ete4 explore -t output.NCBI.tree.tre` among other tools.

## Quick Start

```bash
pip install ncbi-tree
ncbi-tree ./output
```

That's it! The tool will download the latest NCBI taxonomy, generate phylogenetic trees, and create detailed reports.

## Preview of the Output Files

![output.NCBI.tree.tre](https://github.com/PhyloBridge/ncbi-tree/raw/main/assets/output.NCBI.tree.tre.png)

This is the main output file `output.NCBI.tree.tre`, visualized by [ETE4](https://github.com/etetoolkit/ete4) using the command `ete4 explore -t output.NCBI.tree.tre`. Each node in the tree is labeled with an NCBI Taxonomy identifier.

## Features

- [x] **Automatic Download**: Fetches the latest taxonomy data from NCBI FTP servers
- [x] **Version Tracking**: Automatically detects and records the exact server version
- [x] **Smart Caching**: Skips re-download and re-extraction when files already exist
- [x] **Merged Taxa Support**: Handles merged taxonomy IDs from merged.dmp
- [x] **Name Sanitization**: By default, each inital letter is capitalized, and space is replaced by `-`. Configurable name formatting with --no-sanitize option
- [x] **Server-side compatibility**: No more blocking on user input in automated environments. Use `ncbi-tree ./output --no-prompt-1` to automatically generate all files (core + optional); Use `ncbi-tree ./output --no-prompt-0` to generate only core files, skip optional files.

## Installation

```bash
pip install ncbi-tree
```

## Usage

### Basic Usage

```bash
# Download and build taxonomy tree with default settings
ncbi-tree ./output

# Clean up intermediate files after processing
ncbi-tree ./output --no-cache

# Disable name sanitization (keep original spaces)
ncbi-tree ./output --no-sanitize

# Use custom download URL
ncbi-tree ./output --url https://custom-mirror.org/taxdump.tar.gz

# Combined options
ncbi-tree ./output --no-cache --no-sanitize
```

### Server-side, non-blocking, no-interaction

```bash
# Automatically generate all files (core + optional)
ncbi-tree ./output --no-prompt-1

# Generate only core files, skip optional files
ncbi-tree ./output --no-prompt-0
```

### Help

```bash
ncbi-tree --help
ncbi-tree --version
```

## Output Files

### Core Files (Generated Automatically)

1. **`output.NCBI.tree.tre`** - Newick tree with NCBI taxonomy IDs only (compatible with most tree explorers such as ETE4)
2. **`output.NCBI.report.txt`** - Exploratory taxonomy analysis and statistics
3. **`version.txt`** - Server timestamped version for downloaded taxdump.tar.gz

### Optional Files (User Prompted)

After core files are generated, you will be prompted:
```
Would you like to generate optional files (output.NCBI.tree.txt, output.NCBI.named.tree.tre, output.NCBI.ID.to.name.tsv)? [y/N]:
```

If you answer `y`, additional files will be generated **without re-reading data**:

4. **`output.NCBI.tree.txt`** - Plain-text tree with Unicode box-drawing
5. **`output.NCBI.named.tree.tre`** - Newick tree with rank:id:name labels (incompatible with ETE4 tree explorer)
6. **`output.NCBI.ID.to.name.tsv`** - TSV mapping of IDs to names (TaxID, Name, Rank)

## Name Sanitization

By default, taxon names are sanitized for consistent display:
- Spaces replaced with `-`
- Existing `-` escaped as `<->`, which will eventually be escaped back to `-`. Configurable by changing `name = name.replace('-', '<->')` in `sanitize_name` in `core.py`.
- Title case applied
- Special characters removed

**Default (sanitized):**
```
"Human;Homo-Sapiens"
"Norway-Rat;Rattus-Norvegicus"
```

**With `--no-sanitize` flag:**
```
"human; Homo sapiens"
"Norway rat; Rattus norvegicus"
```

## Advanced Configuration

### Custom Name Display

To customize which name types are displayed, edit `NAME_PRIORITIES` in `ncbi_tree/core.py`:

```python
# Default: both common and scientific names
NAME_PRIORITIES = {"genbank common name": 0, "scientific name": 1}
# Result: "Human; Homo sapiens"

# Scientific name only (disable common name)
NAME_PRIORITIES = {"genbank common name": -1, "scientific name": 0}
# Result: "Homo sapiens"

# Common name only (disable scientific name)
NAME_PRIORITIES = {"genbank common name": 0, "scientific name": -1}
# Result: "Human"
```

**Note:** Priority value `-1` disables that name type, `>= 0` enables it (lower number = higher priority).

## Requirements

- Python 3.8 or higher
- requests >= 2.25.0
- tqdm >= 4.50.0

## Technical Details

### Data Source
- **Primary**: NCBI Taxonomy Database (https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/)
- **Updates**: Automatic detection of the latest version with timestamp tracking
- **Size**: ~70-100 MB compressed, ~2.7M+ taxonomy entries at the time of writing (October 2025)
- **Format**: NCBI taxdump format (nodes.dmp, names.dmp, merged.dmp)

### Output Formats
1. **Newick (`.tre`)**: ~22MB for `output.NCBI.tree.tre` and ~120MB for `output.NCBI.named.tree.tre`. Standard phylogenetic tree format compatible with all major tree viewers
2. **Text Tree (`.txt`)**: ~370MB. Unicode-based visualization for terminal/text viewing
3. **TSV Mapping (`.tsv`)**: ~115MB. Tabular format for database integration and lookups
4. **Report (`.txt`)**: ~18KB. Statistical analysis with rank distribution and depth metrics

## License

This project is licensed under the Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0).

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Acknowledgments

- Schoch CL, et al. NCBI Taxonomy: a comprehensive update on curation, resources and tools. Database (Oxford). 2020: baaa062. [PubMed](https://www.ncbi.nlm.nih.gov/pubmed/32761142)
- Sayers EW, et al. GenBank. Nucleic Acids Res. 2019. 47(D1):D94-D99. [PubMed](https://www.ncbi.nlm.nih.gov/pubmed/30365038)