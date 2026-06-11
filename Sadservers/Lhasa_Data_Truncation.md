# Scenario: "Lhasa": Easy Math

**Level:** Easy  
**Tags:** `linux`, `bash`, `text-processing`, `python`  

## 📝 Problem Description
A data file located at `/home/admin/scores.txt` contains two space-separated columns, where the second column represents numeric test scores. The objective is to compute the precise arithmetic mean (average) of these scores and save the result to `/home/admin/solution`. 

There is a strict formatting constraint: the final output must display **exactly two decimal digits without any rounding** (e.g., `21.349` must become `21.34`, not `21.35`).

## 💡 The Solution Strategy
Standard command-line arithmetic utilities like `bc` or standard formatting functions like Python's `round()` or `f"{value:.2f}"` automatically perform rounding up when the third decimal place is 5 or higher. To bypass this and achieve a strict truncation, the data can be parsed using Python, computing the exact average as a high-precision float string, and using string slicing to cut off everything after the second decimal place.

### The Commands
```bash
# 1. Inspect the structure of the data file
head -n 5 /home/admin/scores.txt

# 2. Execute a Python inline script to parse columns, calculate the mean, truncate, and save the output
python3 -c "
with open('/home/admin/scores.txt') as f:
    scores = [float(line.split()[1]) for line in f if line.strip()]
avg = sum(scores) / len(scores)
s = f'{avg:.6f}'
truncated = s[:s.find('.') + 3]
print(truncated)
" > /home/admin/solution

# 3. Verify the final generated value matches the required layout
cat /home/admin/solution


