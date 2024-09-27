
# 1. Density Concept

#### 1. How to Calculate
Density in this context refers to the number of objects per unit of time in a beatmap.
```equation
∂ = (number of hit objects)/(song length)
```

To calculate the number of hit objects in a `.osu` map file (refer to #osuFileDocumentation), subtract the time of the first hit object from the time of the last hit object to determine the map's length. All of this information is available in the map file.

#### 2. Purpose of Density
Density serves multiple purposes. Firstly, it helps determine the difficulty of a map, although it's not the most precise metric. More importantly, density can indicate whether a map is a #JumpyMap, #RegularMap, or #TappyMap. The distinction between the first and last types is particularly interesting because both can be challenging but differ significantly in density. The primary goal is to investigate how density varies across different map difficulties. This variation might resemble a Gaussian curve. The difficulty level of a map (#mapDifficulty) is provided in the `.osu` file. Therefore, the plan is to group all files by difficulty and analyze the density curve. If the curve resembles a Gaussian distribution, we would expect two tails and a center. The first tail corresponds to #JumpyMap, the center correlates with a well-balanced #RegularMap, and the far right tail corresponds to #TappyMap. I anticipate that the curve might be flatter for easier maps and taller for harder maps, but this hypothesis needs to be verified.

# 2. What We Need First

#### 1. Collect Files
Since all the necessary information is available in the `.osu` map files, we should collect these files from various directories into one folder. It's essential that each file has a unique name, so renaming them sequentially (e.g., 1.txt, 2.txt, ..., 30000.txt) is advisable to avoid duplicates. These files are text files, each about 100KB in size, so they don't require much storage space. However, the entire collection of maps with songs may amount to around 500GB. We need to create a Python script to automate this process.
#### 2. Group Files
We should group the files by difficulty. First, we should consult the #osuFileDocumentation to understand the available difficulty levels. It's preferable to use numerical values for grouping since named difficulties by map creators can be inconsistent and sometimes confusing.
#### 3. Measure Density
Measure the density for each file within its respective difficulty group. The script should store this density information in an array, and after processing all files, we should analyse the distribution of densities. Each difficulty level should have its own density distribution. First step is to separate files by this difficulty in-game category. 
![[DifficultySpectrumOsu.png]]
In-game evaluation is star base system and corresponding difficulties look like below graphic
![[DifficultyOsuStarEvaluation.png]]
This separation is most important because we need to make model train in each ot these. And in each difficulty we also need to separate by jumpy, regular and tappy maps kind. 


# 3. Prepare Data

#### 1. Copy Files
We need to copy the `.mp3` and `.txt` files, ensuring they have the same name and are stored in a single directory with unique filenames. It might be wise to start with a smaller subset of files, as the process can be time-consuming. The OSU map collection contains a large number of `.osu` files, so we need to gather these files in one place first. Unique filenames are necessary, but the actual names are not crucial since all relevant information about difficulty and game mode is contained within the file content. We need a script to automate this process.
###### Script for copy and rename
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


#### 2. Separate Files
##### 1. Separate by mode
There are several game modes (0 = osu!, 1 = osu!taiko, 2 = osu!catch, 3 = osu!mania) as outlined in the #osuFileDocumentation. We will focus only on Mode: Int = 0, which is osu!.
###### Script for separate by mode
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

##### 2. Separate by difficulty
###### Script for difficult tidy up
```Python
import os
import shutil
import re

# Constants for directories
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
    file_counter = 0
    difficulty_pattern = re.compile(r'OverallDifficulty:(\d+\.?\d*)')

    # Iterate over all files in the source directory
    for filename in os.listdir(source_dir):
        if filename.endswith('.txt'):
            file_path = os.path.join(source_dir, filename)

            # Try reading the file to find the difficulty
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

            # Find the OverallDifficulty value
            match = difficulty_pattern.search(content)
            if match:
                overall_difficulty = float(match.group(1))
                
                # Determine the target directory based on difficulty
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

                # Print progress every 500 files
                if file_counter % 500 == 0:
                    print(f"Sorted {file_counter} files based on difficulty.")

    return file_counter

# Execute the function
num_files_sorted = move_files_based_on_difficulty(SOURCE_DIR)
print(f"Sorted {num_files_sorted} files into difficulty folders.")
```

#### 3. Calculate density

##### 1. Distribution
This step is probably important in the first time to investigate how to create range of #RegularMap , #JumpyMap and #TappyMap 

![[AllDensityDistribution.png]]
##### 2. Calculated values of densities for every difficult mode
First wy have distribution graphs to determine borders.
###### Script for create pdf with distributions for every difficulty
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

Heres how look output for this analyse:
![[density_distributions left-right.pdf]]

To calculate borders of jumpy and tappy maps we use 1/e Thresholds for each difficulty level (taken from PDF). Exception are easy mode both borders and hard upper border (explained in next paragraph).  These values are: 

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

Next step is to check how many maps we have in this range. This is especially important because we need enough data for trening for the model. 

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

The result of the amount of data (ranking maps collected from packs up to end o 2023 year.)

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
Low value show how many jumpy maps are after 1/e thresholds for each difficulty level separation. Medium are regular maps and High is tappy maps. 

###### easy and hard distribution manual integration explanation
Because lack of #easy maps make hard to examine distribution for jumpy and tapping maps. Probably in this project is better use all maps for easy mode if better results should be obtain. 
![[DensityEasyDistributionFixed.png]]

For hard maps upper limit that designate begining of tappy maps is corrected manualy because of ragged distribution graph. Aproximate line showing how this distribution as a continious function may looks like also was drown manually. 
![[DensityHardDistributionFixed.png]]

###### Number of maps before and after manual correction. 
After correction number of occurrences of left and side is more balanced. I don't know is this should be a determinant but distribution, especially for hard maps looks quite normal or even right sided log normal. This is also occurred in other difficulties, and what is quite common that the most maps are regular so this amount is higher. 
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




#### 4. Update .osu Files
For training data, we need each file to contain only the difficulty and balance factor. Once we have the density, we can categorize it into ranges: the first range represents #JumpyMap, the second range is #RegularMap, and the third is #TappyMap. Understanding the distribution of these ranges is crucial to determine how many maps fall into each category and to adjust the range widths accordingly. This is important because our AI model needs sufficient data for effective learning. I believe three categories should suffice, though the variation will be significant due to differences in difficulty.
###### Each file should ultimately contain:
 1. Information on difficulty and type, for example, J1 for a #JumpyMap with difficulty 1, R9 for a #RegularMap with difficulty 9, and T3 for a #TappyMap with difficulty 3.
 2. The timing and position of hit objects—identical to the original `.osu` file.



# 5. Helpfull link list

1. [.osu file format](https://osu.ppy.sh/wiki/en/Client/File_formats/osu_%28file_format%29#difficulty)
2. [Song setup window](https://osu.ppy.sh/wiki/en/Client/Beatmap_editor/Song_setup#difficulty)
3. [Difficulty](https://osu.ppy.sh/wiki/en/Beatmap/Difficulty)



