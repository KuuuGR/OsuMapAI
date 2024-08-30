
# 1. Density Concept

##### 1. How to Calculate
Density in this context refers to the number of objects per unit of time in a beatmap.
```equation
∂ = (number of hit objects)/(song length)
```

To calculate the number of hit objects in a `.osu` map file (refer to #osuFileDocumentation), subtract the time of the first hit object from the time of the last hit object to determine the map's length. All of this information is available in the map file.

##### 2. Purpose of Density
Density serves multiple purposes. Firstly, it helps determine the difficulty of a map, although it's not the most precise metric. More importantly, density can indicate whether a map is a #JumpyMap, #RegularMap, or #TappyMap. The distinction between the first and last types is particularly interesting because both can be challenging but differ significantly in density. The primary goal is to investigate how density varies across different map difficulties. This variation might resemble a Gaussian curve. The difficulty level of a map (#mapDifficulty) is provided in the `.osu` file. Therefore, the plan is to group all files by difficulty and analyze the density curve. If the curve resembles a Gaussian distribution, we would expect two tails and a center. The first tail corresponds to #JumpyMap, the center correlates with a well-balanced #RegularMap, and the far right tail corresponds to #TappyMap. I anticipate that the curve might be flatter for easier maps and taller for harder maps, but this hypothesis needs to be verified.

# 2. What We Need First

##### 1. Collect Files
Since all the necessary information is available in the `.osu` map files, we should collect these files from various directories into one folder. It's essential that each file has a unique name, so renaming them sequentially (e.g., 1.txt, 2.txt, ..., 30000.txt) is advisable to avoid duplicates. These files are text files, each about 100KB in size, so they don't require much storage space. However, the entire collection of maps with songs may amount to around 500GB. We need to create a Python script to automate this process.

##### 2. Group Files
We should group the files by difficulty. First, we should consult the #osuFileDocumentation to understand the available difficulty levels. It's preferable to use numerical values for grouping since named difficulties by map creators can be inconsistent and sometimes confusing.

##### 3. Measure Density
Measure the density for each file within its respective difficulty group. The script should store this density information in an array, and after processing all files, we should analyze the distribution of densities. Each difficulty level should have its own density distribution.

# 3. Prepare Data

##### 1. Update .osu Files
For training data, we need each file to contain only the difficulty and balance factor. Once we have the density, we can categorize it into ranges: the first range represents #JumpyMap, the second range is #RegularMap, and the third is #TappyMap. Understanding the distribution of these ranges is crucial to determine how many maps fall into each category and to adjust the range widths accordingly. This is important because our AI model needs sufficient data for effective learning. I believe three categories should suffice, though the variation will be significant due to differences in difficulty.

###### Each file should ultimately contain:
 1. Information on difficulty and type, for example, J1 for a #JumpyMap with difficulty 1, R9 for a #RegularMap with difficulty 9, and T3 for a #TappyMap with difficulty 3.
 2. The timing and position of hit objects—identical to the original `.osu` file.

##### 2. Copy Files
We need to copy the `.mp3` and `.txt` files, ensuring they have the same name and are stored in a single directory while maintaining unique names. It might be wise to start with a smaller subset of files, as the process can be time-consuming.