# Scenario: "Minneapolis": Break a CSV file

**Level:** Easy  
**Tags:** `csv`, `data-processing`, `python`, `scripting`, `automation`  

## 📝 Problem Description
The task requires breaking a large Comma Separated Values (CSV) file located at `/home/admin/data.csv` into exactly 10 smaller chunks named sequentially from `data-00.csv` up to `data-09.csv`. 

There are two strict operational constraints:
1. Every newly created smaller file **must preserve the exact same header line** (the first line detailing column names) from the original master file.
2. No single generated sub-file can exceed a size limitation of **32KB**. Line splitting rules can be disregarded, meaning chunks can be cleanly sliced at arbitrary byte boundaries.

## 💡 The Solution Strategy
Standard command-line text processing tools like `split` can easily divide a file based on lines or byte metrics, but they do not natively duplicate and prepend the initial header row across all downstream target segments. 

To satisfy both chunk parity and header preservation safely, an inline Python script is the best choice. By loading the file using binary read modes (`'rb'`), we isolate the first row via `readline()`, capture the remaining body bulk, calculate an even 10-way byte-length slice window, and iteratively commit the header combined with each localized data slice to disk.

### The Commands
```bash
# 1. Inspect the initial footprint and sizing parameters of the master file
ls -lh /home/admin/data.csv

# 2. Run the high-precision Python script to chunk data and clone headers
python3 -c "
import math

# Load the target CSV ensuring the header and body are indexed distinctly
with open('/home/admin/data.csv', 'rb') as f:
    header = f.readline()
    body = f.read()

# Compute the uniform mathematical segment length for a 10-way data spread
chunk_len = math.ceil(len(body) / 10)

# Slice array chunks sequentially and commit outputs with clean string padding
for i in range(10):
    chunk = body[i * chunk_len : (i + 1) * chunk_len]
    filename = f'/home/admin/data-{i:02d}.csv'
    with open(filename, 'wb') as out:
        out.write(header + chunk)
"

# 3. Confirm all 10 target files exist and maintain footprints under 32KB
ls -lh /home/admin/data-*.csv

# 4. Verify structural integrity by inspecting headers on boundary files
head -n 1 /home/admin/data-00.csv
head -n 1 /home/admin/data-09.csv
