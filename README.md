# Minimap2 with user specified splicing sites

This slightly modified version of minimap adds command line options allowing the tweaking of the gap scoring function for known splicing sites. This enables splicing on non-canonical sites and resolving ambiguities in presence of hight noise levels.

## Usage

This patch reuse the `-u` option. In vanilla minimap, it speicifies what kind of intron pattern is enabled:

 - `-u b` (default with `-x splice`): detects GT-AG pattern on both strands
 - `-u n`: no pattern detection
 - `-u f` and `-u r`: only detect on forward (resp. reverse) strand.
 - `-u intron.bed` or `-u intron.tsv`: Use user-specified splicing sites in either BED or custom TSV format.

For example, `-x splice -u n -u introns.bed` only use the BED file for splicing, while `-x splice [-u b] -u introns.bed` enable the hybrid mode where the splicing is favored at user specified sites a bit more than on infered cannonical GT-AG sites.

## File formats
- BED introns files defines the 0-based, end exclusive, intron intervals.
- With the TSV file format, each line define a splicing site under the format `chr \t pos \t L/R` where the last letter defines whether the site marks the opening (`L`eft) or the closing (`R`right) of an intron.

In either cases the notation is independant of whether the site are a donor or an acceptor. What matter is the side of the site relative to the intron in the orientation of the reference (target) sequence.

```
             >chr1
sequence   : X X X G T A G X X
intron/exon: e e e i i i i e e
coordinates: 0 1 2 3 4 i 5 6 7
        TSV: . . L . . . R . .
        BED: . . . [ . . . ) .

introns.tsv:
chr1	2	L
chr1	5	R

introns.bed:
chr1 3 6
```

## Scoring

Once splicing is enabled (through `-x splice`), the cost of a long gap is given by the second `-O` score (default=32, there is no elongation cost). A non cannonical cost, set by `-C` (default=9), is substracted to the score for each non canonical splicing site. Then a score reward, set by `-S` (default is half of `-C`), is added for each specified sites taken. The gap cost ranges from `O₂ + 2C` (two non cannonical sites) to `O₂ - 2S` (two user specified sites). Specified sites take precendence over those infered from pattern recognition.
