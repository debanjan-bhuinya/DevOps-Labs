# Scenario: "Saint Paul": Merge Many CSVs files

**Level:** Easy  
**Tags:** `csv`, `data-processing`, `bash`, `text-manipulation`, `automation`  

## 📝 Problem Description
The task requires combining 338 distinct Comma Separated Values (CSV) files matching the pattern `/home/admin/polldayregistrations_enregistjourduscrutin?????.csv` into a single comprehensive output file named `/home/admin/all.csv`.

The core constraint is structural: the output dataset must contain **exactly one master header line** at the absolute top of the file containing the column designations, followed cleanly by the combined data records from all 338 files in any order. Simple bulk concatenation (`cat *.csv`) is prohibited as it scatters duplicate headers throughout the payload.

## 💡 The Solution Strategy
To avoid iterating through loops or spinning up memory-heavy high-level programming scripts, native POSIX stream-manipulation tools (`head` and `tail`) provide the most efficient, low-overhead remedy.

1. **Header Extraction:** Isolate and print the very first row (`head -n 1`) from a single file to cleanly seed the `/home/admin/all.csv` target and establish the column blueprint.
2. **Body Stream Filtration:** Leverage `tail` with the unique relative offset argument (`-n +2`) across all target files. This skips line 1 (the header) of every single file and streams the data directly. Coupling this with the quiet flag (`-q`) prevents the kernel from printing structural telemetry headers into our clean data stream.

### The Commands
```bash
# 1. Extract the column header line from the first file in the array to initialize the target file
head -n 1 $(ls /home/admin/polldayregistrations_*.csv | head -n 1) > /home/admin/all.csv

# 2. Quietly parse rows starting from line 2 across all 338 files and append to the target file
tail -q -n +2 /home/admin/polldayregistrations_*.csv >> /home/admin/all.csv

# 3. Audit integrity by verifying the file line count metric
wc -l /home/admin/all.csv

# 4. Confirm the layout looks uniform with a single leading header row
head -n 5 /home/admin/all.csv
