CXL_label_analysis_markdown
================
Ben Ibbott
2024-11-28 

Written in R

# Requirements

``` r
library(dplyr)
library(jsonlite)
library(progress)
library(tidyr)
library(ggplot2)
```

# Data imports

## CSV’s of already labelled data

Can keep adding labelled datasets to here. Will be a good way of
remvoing labelled entries from entire dataset

``` r
#CSV containing updated df with results post labelme processing
ACC_CCP <- read.csv("/Users/user/CXL_internship/CXL_R_Directory/Labelled_data/labelled_data_ACC-CCP.csv") 
DW2_DWT <- read.csv("/Users/user/CXL_internship/CXL_R_Directory/Labelled_data/labelled_data_DW2-DWT.csv") 
Animals_below_20000_entries <- read.csv("/Users/user/CXL_internship/CXL_R_Directory/Labelled_data/labelled_animals_below_20000.csv")

# Merge the data frames by rows
merged_labeled_df <- bind_rows(DW2_DWT, ACC_CCP, Animals_below_20000_entries)
```

## Entire JSON metadata file

``` r
json_data <- fromJSON("/Users/user/CXL_internship/New_Zealand_dataset/trail_camera_images_of_new_zealand_animals_1.00.json") # Metadata taken directly from LILA New Zealand webpage

# Convert to a data frame 
lila_metadata_df <- as.data.frame(json_data$images) # this includes project ACC - DWT images (projects I have already labelled)
```

Metadata Counts

``` r
lila_metadata_counts <- lila_metadata_df %>%
  count(species, sort = TRUE)

sum(lila_metadata_counts$n) #total = 2453840
```

    ## [1] 2453840

# Species grouping

## For all animals in dataset

``` r
total_counts <- lila_metadata_df %>%
  count(species, sort = TRUE)

sum(total_counts$n) #Total = 2453840
```

    ## [1] 2453840

## For already labelled data

Group all entries by the species label and create a column showing how
many entries per animal

``` r
labelled_counts <- merged_labeled_df %>% 
  count(original_label_x, sort = TRUE) %>%
  rename(species = original_label_x) #renaming column not working 

# Just to confirm all the images are present 
sum(labelled_counts$n) # Total = 201913 (as of 24.12.13)
```

    ## [1] 201913

^Note - every data entry is a individual detection, there could be
multiple detections in one image.

## For all unlabelled data

This creates a summary table of all 3 variables (total, labelled,
unlabelled)

``` r
unlabelled_counts <- total_counts %>%
  left_join(labelled_counts, by = "species") %>%
  mutate(n.y = replace_na(n.y, 0)) %>%
  mutate(unlabelled_counts = n.x - n.y) %>% # 'n' is column for species counts
  mutate(unlabelled_counts = if_else(unlabelled_counts < 0, "<0", as.character(unlabelled_counts))) %>%
  rename(total_counts = n.x,
         labelled_counts = n.y)
```

# Data filtering

Firstly, What are all the project and how many entries do they have ? -
This is somewhat redundant now as I am no longer labelling based on
projects, instead focused on species

``` r
projects <- lila_metadata_df %>%
  count(project, sort = TRUE)
```

## Creating DF for unlabelled images

Now making a df without entries that I have already labelled, sorting
based on filtering filenames present in the labelled_df from the entire
metadata

``` r
unlabelled_images <- lila_metadata_df %>%
  anti_join(merged_labeled_df, by = "file_name")

str(unlabelled_images)
```

    ## 'data.frame':    2272324 obs. of  6 variables:
    ##  $ location : chr  "ACC_57" "ACC_unknown" "ACC_unknown" "ACC_T54" ...
    ##  $ file_name: chr  "ACC/banded_rail/A29781D2-8ADF-4438-9DA5-423405F45334.JPG" "ACC/kiwi/09EFC3A9-22EC-4880-9B89-4D4A9C2A5DEE.JPG" "ACC/kiwi/450401A9-1FBF-453D-B555-B887DCC6DAFE.JPG" "ACC/mouse/25BFDCF6-E334-4489-A4AC-A054ED660B7F.JPG" ...
    ##  $ id       : chr  "ACC/banded_rail/A29781D2-8ADF-4438-9DA5-423405F45334.JPG" "ACC/kiwi/09EFC3A9-22EC-4880-9B89-4D4A9C2A5DEE.JPG" "ACC/kiwi/450401A9-1FBF-453D-B555-B887DCC6DAFE.JPG" "ACC/mouse/25BFDCF6-E334-4489-A4AC-A054ED660B7F.JPG" ...
    ##  $ datetime : chr  "2023-06-20 08:32:08" "2023-05-04 04:44:18" "2023-05-04 04:44:17" "2023-05-29 04:06:47" ...
    ##  $ project  : chr  "ACC" "ACC" "ACC" "ACC" ...
    ##  $ species  : chr  "banded_rail" "kiwi" "kiwi" "mouse" ...

## Filtering animals \<20,000 entries

The purpose of this was to label all images for animals where they have
\<20,000 images. — This was finished so this df is no longer needed. —

Start by creating a list of all species to keep

``` r
species_below_20000 <- lila_metadata_counts %>%
  filter(n <= 20000) %>%
  pull(species)

print(species_below_20000)
```

    ##  [1] "parakeet"                  "weka"                     
    ##  [3] "white_tailed_deer"         "pig"                      
    ##  [5] "hare"                      "bellbird"                 
    ##  [7] "harrier"                   "pukeko"                   
    ##  [9] "dunnock"                   "chaffinch"                
    ## [11] "kereru"                    "silvereye"                
    ## [13] "chamois"                   "sealion"                  
    ## [15] "pipit"                     "yellow_eyed_penguin"      
    ## [17] "weasel"                    "fantail"                  
    ## [19] "tui"                       "magpie"                   
    ## [21] "myna"                      "greenfinch"               
    ## [23] "yellowhammer"              "quail_brown"              
    ## [25] "takahe"                    "dog"                      
    ## [27] "tahr"                      "starling"                 
    ## [29] "sparrow"                   "goat"                     
    ## [31] "human"                     "pateke"                   
    ## [33] "moth"                      "banded_rail"              
    ## [35] "morepork"                  "chicken"                  
    ## [37] "oystercatcher"             "quail_california"         
    ## [39] "paradise_duck"             "black_fronted_tern"       
    ## [41] "mallard"                   "goldfinch"                
    ## [43] "tieke"                     "banded_dotterel"          
    ## [45] "redpoll"                   "little_blue_penguin"      
    ## [47] "kaka"                      "fernbird"                 
    ## [49] "shore_plover"              "canada_goose"             
    ## [51] "spurwing_plover"           "black_backed_gull"        
    ## [53] "white_faced_heron"         "pheasant"                 
    ## [55] "lizard"                    "shag"                     
    ## [57] "brown_creeper"             "black_billed_gull"        
    ## [59] "wrybill"                   "crake"                    
    ## [61] "horse"                     "skink"                    
    ## [63] "grey_warbler"              "rosella"                  
    ## [65] "swan"                      "fiordland_crested_penguin"
    ## [67] "plover"                    "nz_falcon"                
    ## [69] "stilt"                     "long_tailed_cuckoo"       
    ## [71] "mohua"                     "kingfisher"               
    ## [73] "grey_duck"                 "spotted_dove"             
    ## [75] "fluttering_shearwater"     "whitehead"                
    ## [77] "grey_faced_petrol"         "swallow"

Filter the metadata to only keep those species

``` r
animals_few_entries <- unlabelled_images %>%
  filter(species %in% species_below_20000)
```

Lets sum all the animals in the new df to see if the values make sense
when compared to what I have already labelled and the total number of
images. - i.e. if there are 15,000 parakeets and I have labelled
14,000 - This df should contain 1,000 parakeet images

``` r
animals_few_entries_counts <- animals_few_entries %>%
  count(species, sort = TRUE)
table(animals_few_entries_counts)
```

——- This has highlighted some errors in the labelling ——- seems what has
happened is instead of downloading all images somewhere it has
downloaded the same image multiple times? —— Solution - for now need to
label 20k images of animals \<20,000 entries that were missed. ——

## Filtering animals \>20,000 entires

filtering species above 20k

``` r
species_above_20000 <- lila_metadata_counts %>%
  filter(n > 20000) %>%
  pull(species)

print(species_above_20000)
```

    ##  [1] "mouse"     "possum"    "rat"       "robin"     "cat"       "stoat"    
    ##  [7] "kea"       "thrush"    "deer"      "kiwi"      "blackbird" "hedgehog" 
    ## [13] "tomtit"    "wallaby"   "rifleman"  "rabbit"    "ferret"    "cow"      
    ## [19] "sheep"

create a dataframe with only those entries, taking from remaining
unlabelled images (to avoid labelling same images multiple times.)

``` r
animals_many_entries <- unlabelled_images %>%
  filter(species %in% species_above_20000)
```

## Summing species

calculating counts to compare with total expected

``` r
animals_many_entries_counts <- animals_many_entries %>%
  count(species, sort = TRUE)
table(animals_many_entries_counts)
```

Summing animals entries X \< 20,000 \< X

``` r
{
a <- sum(animals_few_entries_counts$n)
b <- sum(animals_many_entries_counts$n)

print(paste0("animals_few_entries: ", a))
print(paste0("animals_many_entries: ", b))
print(paste0("sum: ", a + b)) # This number should be the same as the amount of objects in "unlabelled_images"
print(paste0("total unlabelled_images: ", count(unlabelled_images)))
}
```

    ## [1] "animals_few_entries: 21991"
    ## [1] "animals_many_entries: 2250333"
    ## [1] "sum: 2272324"
    ## [1] "total unlabelled_images: 2272324"

# Target labels function

the function purpose is to show how many labels are needed to reach
20,000 It was decided that the goal is to label 20,000 entries for each
animal (or if an animal has \<20,000 entries, all entries). Thus need to
calculate how many animals have been labelled vs how many more to reach
20,000.

``` r
label_count_analysis <- function(metadata_df, labelled_df, target_labels) {
  
#' @Arguments
#' metadata_df = all data (use counts df)
#' labelled_df = all labelled data (use counts df)
#' target_labels = Target amount of images to label (20k in this case)
  
  # Merge the dataframes
  result <- merge(metadata_df, labelled_df,
                  by = "species", 
                  all = TRUE, # keeps species even if not in both dfs
                  suffixes = c("_total", "_labelled")) # to distinguish duplicates as you are merging dfs
  
  # For species not in metadata set n_total to 0
  result$n_total[is.na(result$n_total)] <- 0
  
  # For species not in labelled data, set n_labelled = 0
  result$n_labelled[is.na(result$n_labelled)] <- 0
  
  # Calculate how many have been labeled
  result$n_unlabelled <- pmax(result$n_total - result$n_labelled, 0)
  
  # Add column showing target, if the target is greater than the total number of images set it to NA.
  result$target <- ifelse(result$n_total < target_labels, 
                          0,
                          target_labels)
  
  # Calculate how many more needed to reach target, NA if metadata total < target
  result$remaining_labels_to_reach_target <- ifelse(result$n_total < target_labels,
                                                    0,
                                                    pmax(target_labels - result$n_labelled, 0))
  
  return(result)
}
```

Calling function

``` r
Animals_to_label <- label_count_analysis(lila_metadata_counts, labelled_counts, target_labels = 10000)
```

# Stratified sampling

- Within this dataset there are 1.2mil mouse entries, there is not time
  to label all of these and at a certain point it will be overkill for
  the model anyway. The difference in performance (this is a guess)
  after training the model with 100k images vs 1mil images will be \<1%
  and that will take a huge amount of time.

- This is why previously we decided to set 20k as a target as this will
  be a good amount of data to train the model whilst balancing time
  spent across all species.

- When sampling if we just selected the first 20k mice images, they
  would likely all be from very similar locations as the projects a
  grouped somewhat based on camera location. This is an issue as we want
  to train the model on a diverse dataset to prevent over-fitting.

- Thus we will sample the data in a stratified manner, taking equal
  amounts of images (where possible) from all locations, even if this
  disproportionately samples across locations. i.e. if location A has
  10,000 images and location B has 100 images. When building a dataset
  of 100 images if it was proportionate it would be 99 images from
  location A and 1 image from location B. When Stratified it will be 50
  images from location A and 50 images from location B.

## Sampling function

Loops across each location, takes 1 image, removes from image pool (to
prevent duplication) and repeats.

``` r
sample_unlabelled_images <- function(unlabelled_images, lca_result) 
  
#' lca_result = label count analysis results -> essentially the output from using the label_count_analysis function

  {

  # Create empty dataframe to store results
  final_sample <- data.frame()
  
  # Create progress bar
  cat("Processing species:\n")
  pb_species <- progress_bar$new(
    format = "  :what [:bar] :percent eta: :eta",
    total = length(lca_result$species)
  )
  
  # For each species
  for(species_name in lca_result$species) {
    pb_species$tick(tokens = list(what = sprintf("%-20s", species_name)))
    
    # Get number of remaining images needed to reach label target 
    remaining_needed <- lca_result$remaining_labels_to_reach_target[
      lca_result$species == species_name
    ]
    
    # Skip if no more images needed
    if(remaining_needed == 0) next
    
    # Extract all the data from entire unlabelled dataset for species that require labelling 
    # i.e. they were not skipped over because "remaining == 0" was not true
    species_data <- unlabelled_images[unlabelled_images$species == species_name, ]
    
    # Get unique locations (i.e the projects ACC, ACT, CCP etc...)
    locations <- unique(species_data$location)
    
    # Create a list of dataframes, one for each location containing the associated entries.
    location_data <- split(species_data, species_data$location)
    
    # Initialize counter and dataframe to store results 
    images_sampled <- 0
    species_sample <- data.frame()
    
    # Keep going until we have enough images or run out of images
    while(images_sampled < remaining_needed && length(location_data) > 0) {
      
      # Remove any empty locations
      location_data <- location_data[sapply(location_data, nrow) > 0]
      
      # Breaks from loop if there are no more images for that species in any location 
      # it will then move onto next species and loop through that one
      if(length(location_data) == 0) break
      
      # Go through each location
      for(loc in names(location_data)) {
        # Stop looping if you have enough images sampled to reach the remaining needed to reach target.
        if(images_sampled >= remaining_needed) break
        
        # Take the first row from this location
        sampled_row <- location_data[[loc]][1,]
        species_sample <- rbind(species_sample, sampled_row)
        
        # Remove the used row from location_data
        location_data[[loc]] <- location_data[[loc]][-1,]
        
        images_sampled <- images_sampled + 1
      }
    }
    
    # Add to final results
    final_sample <- rbind(final_sample, species_sample)
  }
  
  cat("\nProcessing complete!\n")
  return(final_sample)
}
```

Calling function

``` r
#This takes a while to run so skip unless you do not have the relevant stratified sample CSV or need to generate a new one.
animals_10k_target_stratified_sample <- sample_unlabelled_images(unlabelled_images, Animals_to_label)
```

    ## Processing species:
    ## 
    ## Processing complete!

Loading in previosuly created CSV

``` r
animals_10k_target_stratified_sample <- read.csv("/Users/user/CXL_internship/CXL_R_Directory/animals_10k_target_stratified_sample.csv")
```

## Stratified sample visualisation

Count of images per species to confirm against “Animals_to_label”
dataframe

``` r
animals_above_count <- table(animals_10k_target_stratified_sample$species)

print(animals_above_count)
```

    ## 
    ## blackbird       cat       cow      deer    ferret  hedgehog       kea      kiwi 
    ##      7017      3831     10000      9605     10000      9963      9546      9576 
    ##     mouse    rabbit       rat  rifleman     robin     sheep     stoat    thrush 
    ##      5244      9403      9269      9711      9966      9994      9040      9117 
    ##    tomtit   wallaby 
    ##      8044     10000

``` r
# Turning this into a dataframe 
```

View distribution of samples across locations for each species

``` r
# Counts of species across locations
location_distribution <- table(animals_10k_target_stratified_sample$species, # update here with relevant csv
                             animals_10k_target_stratified_sample$location)

# Convert from a CSV to a data frame for easier viewing
location_dist_df <- as.data.frame(location_distribution)

# Update column names for readability 
names(location_dist_df) <- c("Species", "Location", "Count")

distribution_wide <- location_dist_df %>%
  spread(Location, Count)

# Print the result
print(distribution_wide)
```

In the case of this NZ data ^ there are around 2500 differenct camera
trap locations making this table view pretty inefficient.

Bar Plot for easier visualisation (with so many locations its hard to
see y-axis, important part is seeing bar distribution for overall veiw.)

``` r
species_list <- unique(location_dist_df$Species)

for(sp in species_list) {
  # Filter data for this species and remove rows with zero counts
  species_data <- location_dist_df[location_dist_df$Species == sp & location_dist_df$Count > 0, ]
  
  p <- ggplot(species_data, aes(x = reorder(Location, Count), y = Count)) +
    geom_bar(stat = "identity", fill = "#4682B4") +
    coord_flip() +
    theme_minimal() +
    theme(
      panel.background = element_rect(fill = "white"),  
      plot.background = element_rect(fill = "white")    
    ) +
    labs(title = paste("Distribution across locations for", sp),
         y = "Number of Images",
         x = "Location")
  if(sp %in% c("sheep", "rabbit")) { # can remove this line to plot all graphs, or update for specific species 
    print(p)
    }
}
```

![](Analysis_markdown_files/Distribution_figure/Rabbit_distribution_across_locations.png)<!-- -->![](Analysis_markdown_files/figure-gfm/unnamed-chunk-24-2.png)<!-- --> -
Many cases there is large difference across locations (i.e. rabbit),
this is not due to a stratified sampling error it is simply because some
locations have such few sighting of animals. - In reality this
stratified sampling function successfully extracted all the images from
those locations with very few entries, which otherwise likely would not
have been included if randomly sampling/systematically sampling. - You
can also notice how there is a block or bars at the top of most plots
that are equal length. Likely those areas has many more images of that
species but enough had been taken to reach the target number.

## Splitting dataframe

To save time and get some data for the model to train on we have decided
to prioritise labelling 10k images for key species. Those species
being - cat, stoat, possum, mouse, rat, ferret, cow, hedgehog, The
invasives

To deal with this will take the dataframe that contains the stratified
sample for all animals below 10k labels and separate it into to tables.
Key species vs non-key species.

-\> dataframe being split = animals_10k_target_stratified_sample

Defining the key species vs non key species and making 2 separate dfs

``` r
# Define the key species
key_species <- c("cat", "stoat", "possum", "mouse", "rat", "ferret", "cow", "hedgehog")

# Create two dataframes
key_species_df <- animals_10k_target_stratified_sample[animals_10k_target_stratified_sample$species %in% key_species, ]
non_key_species_df <- animals_10k_target_stratified_sample[!animals_10k_target_stratified_sample$species %in% key_species, ]
```

Checking if counts match up

``` r
{ 
# Key species 
print("Key_species:")
print(table(key_species_df$species))

cat("\n")

# Non key species
print("Non_key_species:")
print(table(non_key_species_df$species))

cat("\n")

a <- count(key_species_df)
b <- count(non_key_species_df)

print(paste0("key species: ", a))
print(paste0("non key species: ", b))
print(paste0("sum: ", a + b)) # This number should be the same as the amount of objects in animals_X_target...
print(paste0("total target images: ", count(animals_10k_target_stratified_sample)))
}
```

    ## [1] "Key_species:"
    ## 
    ##      cat      cow   ferret hedgehog    mouse      rat    stoat 
    ##     3831    10000    10000     9963     5244     9269     9040 
    ## 
    ## [1] "Non_key_species:"
    ## 
    ## blackbird      deer       kea      kiwi    rabbit  rifleman     robin     sheep 
    ##      7017      9605      9546      9576      9403      9711      9966      9994 
    ##    thrush    tomtit   wallaby 
    ##      9117      8044     10000 
    ## 
    ## [1] "key species: 57347"
    ## [1] "non key species: 101979"
    ## [1] "sum: 159326"
    ## [1] "total target images: 159326"

# Saving something as a CSV

At different stages it will be smart to save the table/data frames as
CSVs

``` r
#format
# write.csv(dataframe to save, "name_of_CSV.csv", row.names = FALSE)
write.csv(non_key_species_df, "non_key_species_10k_target.csv", row.names = FALSE)
```
