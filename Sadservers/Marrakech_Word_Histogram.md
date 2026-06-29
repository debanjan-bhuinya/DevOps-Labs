# Scenario: "Marrakech": Word Histogram

**Level:** Medium  
**Tags:** `bash`, `text-processing`, `awk`, `data-manipulation`, `troubleshooting`  

## 📝 Problem Description
The objective is to parse a text file containing the book Frankenstein, identify the second most frequently used word, and extract it in uppercase to a solution file (`/home/admin/mysolution`). 

The parsing logic must explicitly disregard case sensitivity, treat specific punctuation marks (`.,;:`) as word separators (spaces), and leave apostrophes intact as part of the word structure. Furthermore, the environment contains a hidden environmental trap: the target file is intentionally misspelled on the disk as `frankestein.txt`.

## 💡 The Solution Strategy
Building a word frequency histogram is a classic DevOps data-manipulation task that relies on piping POSIX core utilities. Instead of writing a complex Python or Go script, we can string together a highly efficient single-line shell pipeline to melt the text down, format it, and count it.

1. **Sanitization:** We use `tr` (translate) to convert target punctuation into spaces, cast all characters to uppercase, and squeeze all whitespace into single newlines. This leaves us with a clean list containing exactly one word per line.
2. **Aggregation:** We alphabetically `sort` the list so identical words are grouped together, which allows `uniq -c` to count the occurrences of each group. We then run a final numerical reverse sort (`sort -nr`) to push the most frequent words to the absolute top of the stream.
3. **Extraction:** We use `head` and `tail` to isolate the second row, and `awk` to print only the word column, dropping it cleanly into the solution file.

### The Commands
```bash
# 1. Execute the full text-processing pipeline against the target file (noting the filename typo)
cat frankestein.txt | tr '.,;:' ' ' | tr '[:lower:]' '[:upper:]' | tr -s '[:space:]' '\n' | sort | uniq -c | sort -nr | head -n 2 | tail -n 1 | awk '{print $2}' > /home/admin/mysolution

# 2. Validate the output string against the target cryptographic signature
md5sum /home/admin/mysolution
# Output returns: 19bf32b8725ec794d434280902d78e18 (Success)
