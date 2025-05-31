Automatically segments Esperanto words into their component morphemes

### Dependencies

Only Linux is guaranteed to be supported

scala (2.11 definitely works)

python3

### src/WordSegmenter.scala

Algorithm to segment words. Uses two basic steps:

1. Find all possible segmentations via trie traversal, apply rules unless otherwise specified
2. Find best segmentation using a Markov model or maximal match algorithm. 

##### Usage
```
    scala WordSegmenter.WordSegmenter trainingFile morphemesByTypeDirectory [-m|r|n|b|t]
```

See experiments/run\_tests.sh for example usage

##### Options
```
    Default: apply rules, use unigram Markov model
    -m: Use maximal morpheme matching instead of Markov model
    -r: Skip disambiguation (step 2)
    -n: Apply no rules in step 1
    -b: Use bigram Markov model
    -t: Use trigram Markov model
```

Concatenate options if using more than one. e.g. use "-mn", not "-m -n"

Note: unigram, bigram, trigram Markov models refer to 2-gram sequence, 3-gram sequence, 4-gram sequence respectively, as in https://en.wikipedia.org/wiki/N-gram#Examples

##### Build
run:
```
    src/build.sh
```

### morphemesByType/

Defines and classifies all valid morphemes.

Non content morphemes are predefined, following the Akademia Vortaro (http://www.akademio-de-esperanto.org/akademia_vortaro/), with manual classification

morphemesByType/normal/generated is built using morphemesByType/normal/build/classify.py

Uses the dictionary "vortaro.xml" from Esperantilo: http://www.xdobry.de/esperantoedit/index_en.html

To regenerate normal roots run:
```
    morphemesByType/normal/build/get_not_normal.sh
    morphemesByType/normal/build/classify.py
```

To remake morphemesByType/sets directory (what WordSegmenter.scala uses), run:
```
    morphemesByType/make_sets.sh
```

### espsof/

Test set from ESPSOF. All presegmented words from the original source are in espsof.txt. testset\_espsof.txt contains all words with known morphemes (also occur in Esperantilo).

To remake test set (e.g. if the set of known morphemes changes), run:
```
    espsof/make_testset.sh
```

### experiments/

Create a tagged training set and test set from the ESPSOF test set, and run WordSegmenter.scala. Analyze the segmentation accuracy.

To create tagged training/test sets, run:
```
    experiments/run_tests.sh -f
```

To run WordSegmenter.scala with predefined options, run:
```
    experiments/run_tests.sh -r
```

To create tagged sets and run WordSegmenter.scala, run:
```
    experiments/run_tests.sh
```

To analyze the segmentation accuracy, run:
```
    experiments/analyze.py expectFile resultsFile [-r]
```
Use -r if and only if -r was used when running WordSegmenter.scala


# Complete Guide: Esperanto Word Segmentation Tool Setup and Usage

Based on the official GitHub documentation and successful implementation experience.

## Overview

This tool automatically segments Esperanto words into their component morphemes using a two-step algorithm:
1. Find all possible segmentations via trie traversal with optional grammar rules
2. Find the best segmentation using Markov models or maximal match algorithms

## System Requirements

### Operating System
- **Linux only** (officially supported)
- Other systems may work but are not guaranteed

### Dependencies
- **Scala 2.11** (confirmed working version)
- **Python3**

## Installation and Setup

### Step 1: Build the Core Application
```bash
# Navigate to the project root directory
cd EsperantoWordSegmenter/

# Build the Scala source code
src/build.sh
```

This creates the compiled classes in the `bin/` directory.

### Step 2: Generate Morpheme Classification Sets
```bash
# Create morpheme type sets that WordSegmenter uses
morphemesByType/make_sets.sh
```

This processes the morpheme definitions and creates the classification files required by the segmentation algorithm.

### Step 3: Prepare Training and Test Data
```bash
# Navigate to the experiments directory
cd experiments/

# Create tagged training and test sets from ESPSOF data
./run_tests.sh -f
```

**Expected output:** `"Creating tagged sets"`

This generates essential files including:
- `train.txt` - Training data for the Markov models
- `test.txt` - Test data
- `expect.train.txt` and `expect.test.txt` - Expected results for evaluation

### Step 4: Verify Setup
```bash
# Run WordSegmenter with predefined options (optional verification)
./run_tests.sh -r
```

**Expected output:** `"Running WordSegmenter"`

This step tests various configurations and generates output files for analysis.

## Usage Instructions

### Important: Working Directory
**All segmentation commands must be executed from the `experiments/` directory** to ensure correct file paths.

### Basic Word Segmentation

#### Single Word Analysis
```bash
cd experiments/

# Maximum accuracy with trigram Markov model and grammar rules
echo "lernolibro" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t
```

**Expected output:**
```
lernolibro	lern'o'libr'o
```

#### Multiple Words from File
```bash
# Create a word list (one word per line)
cat > words.txt << EOF
lernolibro
esperantujo
areopologio
manĝtablo
dormĉambro
EOF

# Process multiple words
cat words.txt | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t
```

#### Save Results to File
```bash
# Save analysis results for later review
cat words.txt | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t > segmentation_results.txt

# View results
cat segmentation_results.txt
```

## Command Options and Accuracy Levels

### Maximum Accuracy (Recommended)
```bash
# Trigram Markov model with grammar rules applied
# This is the highest accuracy configuration
echo "word" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t
```

### Alternative Configurations

#### Bigram Markov Model
```bash
# Good balance of accuracy and processing speed
echo "word" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -b
```

#### Maximal Morpheme Matching
```bash
# Fast but less accurate
echo "word" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -m
```

#### No Grammar Rules
```bash
# Skip rule-based validation (step 1)
echo "word" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -n
```

#### Combined Options
```bash
# Example: Maximal match with no rules
echo "word" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -mn
```

**Note:** Concatenate options (e.g., `-mn`), don't separate them (not `-m -n`).

## Understanding the Algorithm

### Two-Step Process
1. **Trie Traversal:** Find all possible morpheme segmentations using the morpheme database
2. **Disambiguation:** Select the best segmentation using:
   - **Markov Models:** Statistical approach based on morpheme sequence probabilities
   - **Maximal Match:** Choose segmentation with longest morphemes

### Markov Model Types
- **Unigram (default):** 2-gram sequence analysis
- **Bigram (-b):** 3-gram sequence analysis  
- **Trigram (-t):** 4-gram sequence analysis (highest accuracy)

### Morpheme Database
The tool uses morpheme classifications from:
- **Akademia Vortaro** for non-content morphemes
- **Esperantilo dictionary** (vortaro.xml) for content morphemes

## Input/Output Format

### Input Requirements
- **Format:** One word per line, plain text
- **Encoding:** UTF-8 (supports Esperanto diacritics: ĉ, ĝ, ĥ, ĵ, ŝ, ŭ)
- **Source:** Standard input (stdin)

### Output Format
```
word<tab>morpheme1'morpheme2'morpheme3
```

Example:
```
lernolibro	lern'o'libr'o
esperantujo	esperant'uj'o
```

## Processing Text Documents

### Limitation
This tool processes **individual words only**, not complete sentences or paragraphs.

### Workaround for Text Processing
```bash
# Extract words from text and create word list
cat your_esperanto_text.txt | \
  tr ' ' '\n' | \
  tr ',.;:?!()[]{}""' '\n' | \
  grep -v '^$' | \
  sort -u > extracted_words.txt

# Process the extracted words
cat extracted_words.txt | \
  scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t > \
  analysis_results.txt
```

## Project Structure

```
EsperantoWordSegmenter/
├── README.md
├── src/
│   ├── WordSegmenter.scala    # Main algorithm
│   └── build.sh              # Build script
├── bin/                      # Compiled output (generated)
├── morphemesByType/
│   ├── sets/                 # Morpheme classification files (generated)
│   ├── normal/
│   └── make_sets.sh         # Generates classification sets
├── experiments/
│   ├── run_tests.sh         # Main testing script
│   ├── analyze.py           # Accuracy analysis
│   ├── train.txt            # Training data (generated)
│   ├── test.txt             # Test data (generated)
│   └── (various output files)
└── espsof/
    ├── espsof.txt           # ESPSOF dataset
    └── testset_espsof.txt   # Filtered test set
```

## Troubleshooting

### Common Issues and Solutions

#### "FileNotFoundException: train.txt"
**Cause:** Commands executed from wrong directory or training files not generated
**Solution:** 
```bash
cd experiments/
./run_tests.sh -f  # Regenerate training files
```

#### "No segmentation output"
**Cause:** Input format incorrect or words not in morpheme database
**Solution:**
- Ensure one word per line
- Check for special characters or formatting issues
- Try simpler Esperanto words first

#### "Command not found" errors
**Cause:** Missing dependencies
**Solution:**
```bash
# Check Scala installation
scala -version

# Check Python3 installation  
python3 --version
```

### Verification Test
```bash
cd experiments/
echo "lernolibro" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t
```

**Expected successful output:**
```
lernolibro	lern'o'libr'o
```

## Accuracy Analysis

### Evaluate Segmentation Quality
```bash
# Generate test results
echo "testword" | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t > test_output.txt

# Analyze accuracy (if you have expected results)
./analyze.py expected_results.txt test_output.txt
```

### Performance Comparison
```bash
# Test different models on the same word set
cat words.txt | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ > unigram_results.txt
cat words.txt | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -b > bigram_results.txt  
cat words.txt | scala -cp ../bin/ WordSegmenter.WordSegmenter train.txt ../morphemesByType/sets/ -t > trigram_results.txt
```

This comprehensive guide provides everything needed to successfully set up and use the Esperanto Word Segmentation tool, based on both the official documentation and practical implementation experience.
