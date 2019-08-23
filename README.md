# fisher-lm

The main notebook in this repository documents code for pre-processing the Fisher corpus transcripts into an easier-to-work-with (=relational) format, and into a format that facilitates creating an n-gram model using `kenlm` (= one utterance per line with non-speech events and disfluencies removed or altered to taste).

## Dependencies
 - Fisher corpus transcripts.
 - **`more_itertools`**
 - **`funcy`**
 - **`joblib`**
 - **`Unix`-like OS:** The notebook uses `*nix` shell command magics (mostly `cat` and `head`) as a lightweight means of peeking at files.
 - **`pandas`+`plotnine`:** The notebook uses `pandas` and `plotnine` to plot pre-segmented utterance lengths and the distribution over times between consecutive utterance-onsets.
 - **`kenlm`:** At the end of the notebook, I use shell command magics to call `kenlm`; I also import the `kenlm` python package.
 
The last three items aren't essential for processing/interfacing with the Fisher corpus. `funcy`, `more_itertools`, and `joblib` happen to be used, but could certainly be replaced without too much trouble.

## Outputs

For both the 'main' transcriptions done by the LDC and the BBN ones, this notebook produces
 1. a single .json file containing all of the information contained in the original data formats, plus a processed version of each utterance.
 2. a single .txt file containing vocabulary from the (processed) utterances.
 3. a single .txt file containing one (processed) utterance per line - suitable for input to kenlm.
 
The code at the end will use kenlm to produce .arpa and .mmap files.

## Utterance processing

### Double parentheses
The main (LDC) transcriptions features double parentheses around wordform (sequences) the transcriber wasn't sure of, with text in the double parentheses indicating the transcriber's best guess (if any). Here are some examples (each taken from different conversations):
```
512.82 515.29 A: i so much wanted to be (( ))
```
```
67.12 68.65 A: yeah the last (( )) yeah
```
```
81.27 83.44 B: were you close to where the (( )) tornados
```
```
2.53 4.06 B: (( [noise] hello how are you doing my ))
```
```
91.72 96.48 B: oh bio terror terrorism is a little (( out of prevented )) i don't know (( -bout ))
```
I've removed all double parentheses and kept whatever's inside the parentheses (if anything). If nothing appears, it's been replaced with a custom "unknown" token `<rem>`. (The `kenlm` "unknown" token `<unk>` cannot appear in the data you hand it.)

### Non-speech noises

Anything appearing in square brackets (e.g. `[noise]` above) has been removed in the processed version of each utterance.

### Broken off and resumed words

Wordforms that are broken off in the middle or that are resumed end (or start) in the transcriptions with a dash:
```
281.04 283.89 A: i f- i know i found my job on line i it was
```
```
175.78 178.65 B: uh huh it's r- i love it
```
```
I THINK HUMAN BEINGS ARE PR- UM [LIPSMACK] FRIENDSHIP RELAT- RELATIONSHIPS WITH PEOPLE ARE A LOT MORE IMPORTANT THAN MONEY  (fe_03_05863-A-0012)
```
```
91.72 96.48 B: oh bio terror terrorism is a little (( out of prevented )) i don't know (( -bout ))
```
```
okay yeah s- we- -ll disney is pretty safe and pretty good
```
```
they have the pales- -tin- -instinians and all that stuff going on
```

While occasionally a word is started, interrupted, and then resumed (as in the last example) in a way that indicates what the speaker intended, there are very few of these - few enough of these that it's not worth 'fixing' them.

Any wordform starting or ending with a dash has been replaced with `<rem>`.

### Case

All wordforms have been converted to lowercase.


## Parameter selection notebook

This notebook is intended to facilitate parameter selection for a language model based on the Fisher corpus using `kenlm`. It constructs and measures the perplexity of language models for various parameter choices (documented in the notebook). It does so by dividing a shuffled list of corpus utterances 90/10 into a training and test set and checking the perplexity of each parameter combination on the test set.