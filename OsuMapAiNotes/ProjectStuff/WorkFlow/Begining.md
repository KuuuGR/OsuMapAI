# I. Density Concept
### 1. How to Calculate Density
Density, in this context, refers to the number of objects that appear in a beatmap per unit of time.
```equation
∂ = (number of hit objects) / (song length)
```
To calculate the density in an `.osu` map file (refer to `#osuFileDocumentation`), identify the timestamps of the first and last hit objects, then subtract the time of the first hit object from that of the last hit object. This will give you the total duration of the map. With this information, you can then compute the density of hit objects over the song's length.
### 2. Purpose of Density
Density serves multiple purposes. First, it helps gauge the overall difficulty of a beatmap, though it might not always be the most precise metric. More importantly, density can distinguish between different map types such as #JumpyMap, #RegularMap, or #TappyMap. The distinction between these types is particularly interesting, as both #JumpyMap and #TappyMap can be difficult but differ significantly in density.

The primary objective is to analyze how density varies across different map difficulties. This variation might resemble a Gaussian distribution. Using the difficulty level data provided in the `.osu` file (#mapDifficulty), we can group all files by difficulty and examine the density curve. If the curve is Gaussian-shaped, we would expect two tails and a center: the left tail corresponding to #JumpyMap, the center representing a well-balanced #RegularMap, and the right tail corresponding to #TappyMap.

___
# II. Initial Steps for Density Evaluation
## 1. Collect Files
Since all necessary information is available in `.osu` map files, the first step is to collect these files from various directories into a single folder. It is crucial that each file has a unique name, so renaming them sequentially (e.g., 1.txt, 2.txt, ..., 30000.txt) is advisable to avoid duplicates. These files are text-based, each around 100KB in size, so they don’t require much storage space. However, the entire collection of maps, including their corresponding songs, could total approximately 500GB. We need to create a Python script to automate the collection and renaming process.
## 2. Group Files by Difficulty
Next, the collected files should be grouped by difficulty. First, refer to `#osuFileDocumentation` to understand the available difficulty levels. It’s preferable to use numerical values for grouping since the named difficulty levels created by map authors can be inconsistent and ambiguous.
## 3. Measure Density
### a. General info
For each file, measure the density within its respective difficulty group. The script should store this density information in an array. After processing all files, analyse the distribution of densities to create a density profile for each difficulty level.

The first step is to separate the files based on the difficulty levels used in the game. In-game evaluation uses a star rating system, and the corresponding difficulties are represented in the following graphics:

![[DifficultySpectrumOsu.png]]
![[DifficultyOsuStarEvaluation.png]]

This categorisation is critical because the model needs to be trained separately for each difficulty group. Furthermore, within each difficulty level, maps should be further categorised into #JumpyMap, #RegularMap, and #TappyMap, based on their density values.

---
### b Copy and Organize Files
We need to copy the `.mp3` and `.txt` files, ensuring they retain the same names and are stored in a single directory with unique filenames. It's advisable to start with a smaller subset of files initially, as the entire process can be time-consuming. The OSU map collection contains a vast number of `.osu` files, so it’s crucial to consolidate these files in one location before proceeding.

Using unique filenames is necessary, but the actual names themselves are not important, as all relevant information regarding difficulty and game mode is contained within the file content. To simplify the organization and prevent name conflicts, we will create a Python script to automate the copying and renaming process.
###### Script for Copying and Renaming Files
```Python
import os
import shutil

# Constants for source and destination directories
SOURCE_DIR = '/Volumes/T7/Osu!/Songs'
DEST_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu'

def copy_and_rename_osu_files(source_dir, dest_dir):
    # Ensure the destination directory exists
    if not os.path.exists(dest_dir):
        os.makedirs(dest_dir)

    # Initialize a counter for naming the files
    file_counter = 1

    # Walk through the source directory and its subdirectories
    for root, dirs, files in os.walk(source_dir):
        for file in files:
            if file.endswith('.osu'):
                # Construct the full path to the source file
                source_file_path = os.path.join(root, file)
                
                # Create a new filename with the counter and .txt extension
                dest_file_name = f"{file_counter}.txt"
                dest_file_path = os.path.join(dest_dir, dest_file_name)
                
                # Copy the file and rename it at the destination
                shutil.copy2(source_file_path, dest_file_path)
                
                # Increment the file counter for the next file
                file_counter += 1

                # Print progress every 100 files
                if file_counter % 100 == 1:  # Print after every 100 files are copied
                    print(f"Copied {file_counter - 1} files so far...")

    return file_counter - 1

# Execute the function
num_files_copied = copy_and_rename_osu_files(SOURCE_DIR, DEST_DIR)
print(f"Copied and renamed {num_files_copied} files.")
```

### c. Separate Files by Game Mode
There are several game modes in the OSU game format, represented numerically as follows:

- `0 = osu!`
- `1 = osu!taiko`
- `2 = osu!catch`
- `3 = osu!mania`

We will focus only on Mode `0`, which corresponds to osu!. Other game modes will be separated into their own directories for organizational purposes, as specified in the `#osuFileDocumentation`.

###### Script for Separating Files by Mode
The following script separates the files based on their respective modes and moves them into designated folders.
```Python
import os
import shutil

# Constants for directories
SOURCE_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu'
OSU_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/osu'
MANIA_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/mania'
TAIKO_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/taiko'
CATCH_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/catch'

# Ensure the destination directories exist
for directory in [OSU_DIR, MANIA_DIR, TAIKO_DIR, CATCH_DIR]:
    if not os.path.exists(directory):
        os.makedirs(directory)

def move_files_based_on_mode(source_dir):
    file_counter = 0

    # Iterate over all files in the source directory
    for filename in os.listdir(source_dir):
        if filename.endswith('.txt'):
            file_path = os.path.join(source_dir, filename)
            
            # Try reading the file with different encodings
            try:
                with open(file_path, 'r', encoding='utf-8') as file:
                    content = file.read()
            except UnicodeDecodeError:
                try:
                    with open(file_path, 'r', encoding='iso-8859-1') as file:
                        content = file.read()
                except UnicodeDecodeError:
                    print(f"Skipping file due to encoding issues: {filename}")
                    continue
            
            # Determine the mode from the content
            mode = None
            if 'Mode: 0' in content:
                mode = 'osu'
                dest_dir = OSU_DIR
            elif 'Mode: 1' in content:
                mode = 'taiko'
                dest_dir = TAIKO_DIR
            elif 'Mode: 2' in content:
                mode = 'catch'
                dest_dir = CATCH_DIR
            elif 'Mode: 3' in content:
                mode = 'mania'
                dest_dir = MANIA_DIR
            
            # If mode is found, move the file to the corresponding directory
            if mode:
                shutil.move(file_path, os.path.join(dest_dir, filename))
                file_counter += 1

                # Print progress every 500 files
                if file_counter % 500 == 0:
                    print(f"Tidied up {file_counter} files.")

    return file_counter

# Execute the function
num_files_moved = move_files_based_on_mode(SOURCE_DIR)
print(f"Tidied up {num_files_moved} files in total.")
```

### d. Separate Files by Difficulty
The following script categorizes the `.txt` files based on their difficulty level. The difficulty is determined using the `OverallDifficulty` attribute found within the file content.
###### Script for Organizing Files by Difficulty
```Python
import os
import shutil
import re

# Define constants for the source directory and destination directories for each difficulty level
SOURCE_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/osu'
EASY_DIR = os.path.join(SOURCE_DIR, 'Easy')
NORMAL_DIR = os.path.join(SOURCE_DIR, 'Normal')
HARD_DIR = os.path.join(SOURCE_DIR, 'Hard')
INSANE_DIR = os.path.join(SOURCE_DIR, 'Insane')
EXPERT_DIR = os.path.join(SOURCE_DIR, 'Expert')
EXPERT_PLUS_DIR = os.path.join(SOURCE_DIR, 'Expert+')

# Ensure the destination directories exist
for directory in [EASY_DIR, NORMAL_DIR, HARD_DIR, INSANE_DIR, EXPERT_DIR, EXPERT_PLUS_DIR]:
    if not os.path.exists(directory):
        os.makedirs(directory)

def move_files_based_on_difficulty(source_dir):
    """
    This function reads each .txt file in the source directory,
    extracts its difficulty level, and moves it to the corresponding directory
    based on the OverallDifficulty value.
    """
    file_counter = 0
    difficulty_pattern = re.compile(r'OverallDifficulty:(\d+\.?\d*)')

    # Iterate over all files in the source directory
    for filename in os.listdir(source_dir):
        if filename.endswith('.txt'):
            file_path = os.path.join(source_dir, filename)

            # Attempt to read the file and extract its difficulty level
            try:
                with open(file_path, 'r', encoding='utf-8') as file:
                    content = file.read()
            except UnicodeDecodeError:
                try:
                    with open(file_path, 'r', encoding='iso-8859-1') as file:
                        content = file.read()
                except UnicodeDecodeError:
                    print(f"Skipping file due to encoding issues: {filename}")
                    continue

            # Extract the OverallDifficulty value from the file content
            match = difficulty_pattern.search(content)
            if match:
                overall_difficulty = float(match.group(1))

                # Determine the target directory based on the difficulty value
                if 0.0 <= overall_difficulty <= 1.99:
                    dest_dir = EASY_DIR
                elif 2.0 <= overall_difficulty <= 2.69:
                    dest_dir = NORMAL_DIR
                elif 2.7 <= overall_difficulty <= 3.99:
                    dest_dir = HARD_DIR
                elif 4.0 <= overall_difficulty <= 5.29:
                    dest_dir = INSANE_DIR
                elif 5.3 <= overall_difficulty <= 6.49:
                    dest_dir = EXPERT_DIR
                elif overall_difficulty >= 6.5:
                    dest_dir = EXPERT_PLUS_DIR
                else:
                    print(f"Unclassified difficulty for file: {filename}")
                    continue

                # Move the file to the corresponding directory
                shutil.move(file_path, os.path.join(dest_dir, filename))
                file_counter += 1

                # Print progress every 500 files moved
                if file_counter % 500 == 0:
                    print(f"Sorted {file_counter} files based on difficulty.")

    return file_counter

# Execute the function and display the result
num_files_sorted = move_files_based_on_difficulty(SOURCE_DIR)
print(f"Successfully sorted {num_files_sorted} files into their respective difficulty folders.")

```


### e. Calculated Density Values for Each Difficulty Level
The first step in determining the boundaries for different map types is to generate distribution graphs. These graphs help visualize the density values across various difficulties and identify the thresholds for each category.

The script below creates a PDF file with histograms for density distributions across different difficulty levels. This helps in visualizing how densities are spread for each difficulty group and setting thresholds.
###### Script for Creating Density Distribution PDFs
```Python
import os
import re
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.backends.backend_pdf import PdfPages
import math

# Constants for directories and files
BASE_DIR = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/osu'
DENSITY_FILE = os.path.join(BASE_DIR, 'densities.txt')
OUTPUT_PDF = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/density_distributions.pdf'

# Difficulty levels
DIFFICULTY_LEVELS = ['Easy', 'Normal', 'Hard', 'Insane', 'Expert', 'Expert+']

# X-axis range settings for each difficulty level (min, max)
X_AXIS_RANGES = {
    'Easy': (0, 3),
    'Normal': (0, 3),
    'Hard': (0, 3),
    'Insane': (0, 4),
    'Expert': (0, 5),
    'Expert+': (0, 10)
}

# Custom colors for each difficulty level
COLORS = {
    'Easy': 'blue',
    'Normal': 'green',
    'Hard': 'orange',
    'Insane': 'red',
    'Expert': 'purple',
    'Expert+': 'brown'
}

def calculate_density_for_files_in_difficulty(difficulty_dir):
    densities = []
    file_counter = 0

    # Regular expressions to capture hit objects and their timestamps
    hit_objects_pattern = re.compile(r'\[HitObjects\]\n(.*?)$', re.DOTALL)
    time_pattern = re.compile(r'\d+,\d+,(?P<time>\d+),')

    # Iterate over all files in the difficulty directory
    for filename in os.listdir(difficulty_dir):
        if filename.endswith('.txt'):
            file_path = os.path.join(difficulty_dir, filename)

            # Try reading the file to find the hit objects section
            try:
                with open(file_path, 'r', encoding='utf-8') as file:
                    content = file.read()
            except UnicodeDecodeError:
                try:
                    with open(file_path, 'r', encoding='iso-8859-1') as file:
                        content = file.read()
                except UnicodeDecodeError:
                    print(f"Skipping file due to encoding issues: {filename}")
                    continue

            # Extract the hit objects section
            hit_objects_section = hit_objects_pattern.search(content)
            if hit_objects_section:
                hit_objects = hit_objects_section.group(1).strip().split('\n')
                num_hit_objects = len(hit_objects)

                # Extract the times of the first and last hit objects
                times = []
                for obj in hit_objects:
                    time_match = time_pattern.search(obj)
                    if time_match:
                        times.append(int(time_match.group('time')))
                
                if times:
                    first_time = min(times)
                    last_time = max(times)
                    map_length_ms = last_time - first_time

                    if map_length_ms > 0:  # Prevent division by zero
                        density_per_second = (num_hit_objects / map_length_ms) * 1000  # Convert to objects per second
                        densities.append(density_per_second)

            file_counter += 1

            # Print progress every 500 files
            if file_counter % 500 == 0:
                print(f"Processed {file_counter} files in {difficulty_dir}")

    return densities

def analyze_densities(base_dir, difficulty_levels):
    density_data = {}

    for level in difficulty_levels:
        difficulty_dir = os.path.join(base_dir, level)
        densities = calculate_density_for_files_in_difficulty(difficulty_dir)
        density_data[level] = densities
        print(f"Finished processing {len(densities)} files in {level} difficulty.")

    return density_data

def save_densities_to_file(density_data, file_path):
    with open(file_path, 'w') as f:
        for level, densities in density_data.items():
            for density in densities:
                f.write(f"{level},{density}\n")

def load_densities_from_file(file_path):
    density_data = {level: [] for level in DIFFICULTY_LEVELS}
    with open(file_path, 'r') as f:
        for line in f:
            level, density = line.strip().split(',')
            density_data[level].append(float(density))
    return density_data

def find_1e_thresholds(counts, bins):
    # Find the max count (peak) and calculate the 1/e threshold
    max_count = max(counts)
    threshold = max_count / math.e

    peak_index = np.argmax(counts)
    
    left_threshold = None
    right_threshold = None

    # Find the left threshold by iterating left from the peak
    for i in range(peak_index, -1, -1):
        if counts[i] <= threshold:
            left_threshold = bins[i]
            break

    # Find the right threshold by iterating right from the peak
    for i in range(peak_index, len(counts)):
        if counts[i] <= threshold:
            right_threshold = bins[i]
            break
    
    return left_threshold, right_threshold

def plot_density_distributions(density_data, output_pdf=OUTPUT_PDF):
    with PdfPages(output_pdf) as pdf:
        fig, axes = plt.subplots(nrows=2, ncols=3, figsize=(18, 12), constrained_layout=True)
        axes = axes.flatten()

        for i, (level, densities) in enumerate(density_data.items()):
            if densities:
                ax = axes[i]

                x_min, x_max = X_AXIS_RANGES.get(level, (0, 10))
                color = COLORS.get(level, 'gray')

                counts, bins, _ = ax.hist(densities, bins=500, range=(x_min, x_max), alpha=0.6, color=color, label=f"{level} Histogram")

                mean = np.mean(densities)
                mode_val = bins[np.argmax(counts)]

                ax.axvline(x=mean, color='blue', linestyle='--', label=f'Mean: {mean:.2f}')
                ax.axvline(x=mode_val, color='red', linestyle=':', label=f'Mode: {mode_val:.2f}')

                # Calculate and plot 1/e threshold lines
                left_threshold, right_threshold = find_1e_thresholds(counts, bins)
                if left_threshold is not None:
                    ax.axvline(x=left_threshold, color='purple', linestyle='-.', label=f'1/e Left: {left_threshold:.2f}')
                if right_threshold is not None:
                    ax.axvline(x=right_threshold, color='purple', linestyle='-.', label=f'1/e Right: {right_threshold:.2f}')

                ax.set_title(f"{level} Density Distribution")
                ax.set_xlabel("Density (objects/second)")
                ax.set_ylabel("Number of Occurrences")
                ax.legend()

        plt.suptitle("Density Distributions by Difficulty Level", fontsize=16)
        pdf.savefig(fig)
        plt.close()

# Main execution
if os.path.exists(DENSITY_FILE):
    print("Loading densities from file...")
    density_data = load_densities_from_file(DENSITY_FILE)
else:
    print("Calculating densities...")
    density_data = analyze_densities(BASE_DIR, DIFFICULTY_LEVELS)
    print("Saving densities to file...")
    save_densities_to_file(density_data, DENSITY_FILE)

plot_density_distributions(density_data)
```
##### Output Graphs
Combined densities for all difficulties:
![[AllDensityDistribution.png]]

---

Here is how the output of this analysis looks:
![[density_distributions left-right.pdf]]

---
### f.  Establish Density Thresholds
After generating the PDF with density distribution plots, we use these graphs to determine the boundaries for different map types based on 1/e threshold values.

To define the thresholds for jumpy and tappy maps, we set the boundaries based on 1/e values for each difficulty level (derived from the PDF). Exceptions apply to the `Easy` mode and the upper border for `Hard` mode, which are explained in the subsequent paragraphs. The threshold values are as follows:

```Python
THRESHOLDS = {
'Easy': (0.62, 1.07),
'Normal': (0.77, 1.29),
'Hard': (0.92, 1.68),
'Insane': (1.19, 2.34),
'Expert': (1.99, 3.21),
'Expert+': (2.42, 4.86)
}
```

---
##### Manual Adjustments for Easy and Hard Distributions

For the `Easy` and `Hard` difficulty levels, we needed to manually adjust the distribution boundaries:

- **Easy Mode**: Due to the limited number of maps in this difficulty, it’s challenging to establish clear borders for jumpy and tappy maps. Therefore, it might be better to include all maps in this mode for training to obtain more accurate results.
    
- **Hard Mode**: The upper limit for the `Hard` difficulty was manually adjusted due to the irregular distribution. An approximate line showing how the distribution might look as a continuous function was drawn manually.

![[DensityEasyDistributionFixed.png]]

For the `Hard` mode, the manual integration resulted in a more balanced distribution:

![[DensityHardDistributionFixed.png]]

###### Comparison Before and After Manual Correction

The table below compares the number of maps classified before and after the manual corrections:

**Before Correction**
```Before
Easy Density Classification:
  Low: 732 occurrences
  Medium: 152 occurrences
  High: 456 occurrences

Hard Density Classification:
  Low: 768 occurrences
  Medium: 4505 occurrences
  High: 2161 occurrences
```

**After Correction**
```After
Easy Density Classification:
  Low: 113 occurrences
  Medium: 771 occurrences
  High: 456 occurrences

Hard Density Classification:
  Low: 768 occurrences
  Medium: 5713 occurrences
  High: 953 occurrences
```

After correction, the number of occurrences in the `Medium` category increased, providing a more balanced and representative distribution. This adjustment helps ensure that the model is trained with a diverse set of data points, improving its overall performance.




### g. Analyzing Density Categories
The next step is to determine how many maps fall within each density range for every difficulty level. This step is crucial, as we need to ensure there is sufficient data in each category for effective model training.
###### Script for Counting Maps in Each Category

The following script categorizes the maps into `Low`, `Medium`, and `High` density categories based on predefined 1/e threshold values for each difficulty level.

```Python
import os

# Constants for directories and files
DENSITY_FILE = '/Users/grzegorzkulesza/Development/MyPractice/PythonScripts/Osu/osu/densities.txt'

# 1/e Thresholds for each difficulty level (taken from your PDF)
THRESHOLDS = {
    'Easy': (0.62, 1.07),
    'Normal': (0.77, 1.29),
    'Hard': (0.92, 1.68),
    'Insane': (1.19, 2.34),
    'Expert': (1.99, 3.21),
    'Expert+': (2.42, 4.86)
}

def classify_density(density, left_threshold, right_threshold):
    if density < left_threshold:
        return 'Low'
    elif left_threshold <= density <= right_threshold:
        return 'Medium'
    else:
        return 'High'

def analyze_densities(file_path, thresholds):
    counts = {level: {'Low': 0, 'Medium': 0, 'High': 0} for level in thresholds.keys()}
    
    with open(file_path, 'r') as f:
        for line in f:
            level, density = line.strip().split(',')
            density = float(density)
            left_threshold, right_threshold = thresholds[level]
            category = classify_density(density, left_threshold, right_threshold)
            counts[level][category] += 1
    
    return counts

def print_analysis_results(counts):
    for level, count_dict in counts.items():
        print(f"\n{level} Density Classification:")
        for category, count in count_dict.items():
            print(f"  {category}: {count} occurrences")

# Main execution
if os.path.exists(DENSITY_FILE):
    density_counts = analyze_densities(DENSITY_FILE, THRESHOLDS)
    print_analysis_results(density_counts)
else:
    print(f"Densities file not found: {DENSITY_FILE}")
```

##### Density Classification Results

The table below shows the distribution of maps across `Low`, `Medium`, and `High` density categories for each difficulty level (based on map collections up to the end of 2023).

```Therminal
Easy Density Classification:
  Low: 113 occurrences
  Medium: 771 occurrences
  High: 456 occurrences

Normal Density Classification:
  Low: 700 occurrences
  Medium: 6119 occurrences
  High: 1161 occurrences

Hard Density Classification:
  Low: 768 occurrences
  Medium: 5713 occurrences
  High: 953 occurrences

Insane Density Classification:
  Low: 1756 occurrences
  Medium: 16972 occurrences
  High: 2389 occurrences

Expert Density Classification:
  Low: 2182 occurrences
  Medium: 11605 occurrences
  High: 1490 occurrences

Expert+ Density Classification:
  Low: 4069 occurrences
  Medium: 44954 occurrences
  High: 7874 occurrences
```

- **Low**: Maps identified as "jumpy" based on their density values falling below the lower threshold for each difficulty level.
- **Medium**: Maps considered "regular" whose density values fall within the defined thresholds.
- **High**: Maps classified as "tappy," which exceed the upper threshold for their respective difficulty level.

# III. Updating `.osu` Files

For training purposes, each file should only contain information about its difficulty and balance factor. After calculating the density, we can categorize it into three ranges:

1. **#JumpyMap** – Represents the lowest density range.
2. **#RegularMap** – Represents the middle density range.
3. **#TappyMap** – Represents the highest density range.

Understanding the distribution of these categories is crucial to determine how many maps fall into each group and adjust the range widths if necessary. This is essential because our AI model requires sufficient data in each category for effective learning.

While three categories should suffice, it’s important to note that variations within these categories will be significant due to differences in difficulty levels.

#### Each `.osu` file should ultimately contain:

1. **Difficulty and Type Information**: For example:
    
    - `J1` for a #JumpyMap with difficulty 1
    - `R9` for a #RegularMap with difficulty 9
    - `T3` for a #TappyMap with difficulty 3
2. **Timing and Position of Hit Objects**: These should remain identical to the original `.osu` file to preserve the map's structure.
    
---

# IV. Helpfull link list
1. [.osu file format](https://osu.ppy.sh/wiki/en/Client/File_formats/osu_%28file_format%29#difficulty)
2. [Song setup window](https://osu.ppy.sh/wiki/en/Client/Beatmap_editor/Song_setup#difficulty)
3. [Difficulty](https://osu.ppy.sh/wiki/en/Beatmap/Difficulty)



