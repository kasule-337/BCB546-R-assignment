---
title: "R assignment"
author: "Faizo"
date: "2025-03-20"
output: html_document
---

````{r}
# Load necessary library
library(readr)
```

````{r}
# Define GitHub raw URLs for the files
genotypes_url <- "https://raw.githubusercontent.com/kasule-337/BCB546-R-assignment/main/fang_et_al_genotypes.txt"
snp_position_url <- "https://raw.githubusercontent.com/kasule-337/BCB546-R-assignment/main/snp_position.txt"

# Define local file paths
genotypes_file <- "fang_et_al_genotypes.txt"
snp_position_file <- "snp_position.txt"

# Download the files
download.file(genotypes_url, genotypes_file, method = "libcurl")
download.file(snp_position_url, snp_position_file, method = "libcurl")

# Read the files into R
fang_et_al_genotypes <- read_tsv(genotypes_file)
snp_position <- read_tsv(snp_position_file)
```

````{r}
## Load required libraries
library(dplyr)
library(tidyr)
library(readr)
library(tidyverse)
```


# Data Inspection 
The following set of code inspects the data 
```{r}
### This shows the dimension of the dataframe
# Print first few rows to confirm
head(fang_et_al_genotypes)
head(snp_position)
dim(fang_et_al_genotypes) 
dim(snp_position)

###This shows the summary of each data set 
colnames(fang_et_al_genotypes)
colnames(snp_position)
sapply(fang_et_al_genotypes, class)
sapply(snp_position, class)
is.data.frame(fang_et_al_genotypes)
is.data.frame(snp_position)
file.info("fang_et_al_genotypes")
file.info("snp_position.txt")
sum(is.na(fang_et_al_genotypes))
sum(is.na(snp_position))
cor(snp_position[sapply(snp_position, is.numeric)])
any(is.na(fang_et_al_genotypes))
any(is.na(snp_position))
any(duplicated(fang_et_al_genotypes))
any(duplicated(snp_position))
```

#Data processing 
```{r}
### This line of code copies out the data based on the group into a new file 

# Maize_data and teosinte_data
maize_data_group <- fang_et_al_genotypes[fang_et_al_genotypes$Group %in% c("ZMMIL", "ZMMLR", "ZMMMR"), ]
teosinte_data_group <- fang_et_al_genotypes[fang_et_al_genotypes$Group %in% c("ZMPBA", "ZMPIL", "ZMPJA"), ]
```

```{r}
# This line of code will arrange the snp_position data frame by SNP_ID
snp_position_Chr <- snp_position[,-c(2,5:15)] 
snp_position_arranged <- arrange(snp_position_Chr, SNP_ID)
```

```{r}
library(dplyr)

# Remove columns 1 to 3 from the maize_data_group and teosinte_data_group data frame
maize_data_group <- select(maize_data_group, -(1:3))
teosinte_data_group <- select(teosinte_data_group, -(1:3))
# The data will be transposed using this line of code
maize_genotypes_trans <- t(maize_data_group)
teosinte_genotypes_trans <- t(teosinte_data_group)
```

```{r}
# Combining snp_position_arranged with teosinte_genotypes_trans at the 1,1 position
teosinte_snp_genotype <- cbind(snp_position_arranged, teosinte_genotypes_trans)
```

```{r}
# Combining snp_position_arranged with maize_genotypes_trans at the 1,1 position
maize_snp_genotype <- cbind(snp_position_arranged, maize_genotypes_trans)
```

```{r}
# Print the number of rows for each data frame
nrow(maize_genotypes_trans)
nrow(teosinte_genotypes_trans)
nrow(snp_position_arranged)
```

```{r}
#This replaces all the ? data with -
maize_snp_genotype <- maize_snp_genotype %>% mutate_all(function(x) gsub("\\?", "\\-",x))
teosinte_snp_genotype <- teosinte_snp_genotype %>% mutate_all(function(x) gsub("\\?", "\\-",x))
```


```{r}
#For maize
library(dplyr)
library(readr)
for (i in 1:10) {
  print(paste("Processing Chromosome", i))
  ascending_data <- maize_snp_genotype %>% 
    filter(Chromosome == i) %>% 
    arrange(as.numeric(Position))
  
  descending_data <- maize_snp_genotype %>% 
    filter(Chromosome == i) %>% 
    arrange(desc(as.numeric(Position)))
  
  
  write_tsv(ascending_data, paste("Maize_chr", i, "ascending.txt", sep = "_"))
  
  
  write_tsv(descending_data, paste("Maize_chr", i, "descending.txt", sep = "_"))
}
```


#For Teosinte
```{r}
for (i in 1:10) {
  print(paste("Processing Chromosome", i))
  ascending_data <- teosinte_snp_genotype %>% 
    filter(Chromosome == i) %>% 
    arrange(as.numeric(Position))
  descending_data <- teosinte_snp_genotype %>% 
    filter(Chromosome == i) %>% 
    arrange(desc(as.numeric(Position)))
  write_tsv(ascending_data, paste("teosinte_chr", i, "ascending.txt", sep = "_"))
  write_tsv(descending_data, paste("teosinte_chr", i, "descending.txt", sep = "_"))
}
```
# Data visualization
```{r}
# Load necessary libraries
library(ggplot2)
library(dplyr)
library(tidyr)
library(readr)
```

# Convert Chromosome and Position to numeric and remove missing values
```{r}
maize_snp_genotype_clean <- maize_snp_genotype %>%
  mutate(Chromosome = as.numeric(Chromosome),
         Position = as.numeric(Position)) %>%
  drop_na()

teosinte_snp_genotype_clean <- teosinte_snp_genotype %>%
  mutate(Chromosome = as.numeric(Chromosome),
         Position = as.numeric(Position)) %>%
  drop_na()

# SNP Distribution Density per Chromosome for Maize
ggplot(data = maize_snp_genotype_clean) + 
  geom_density(mapping = aes(x = Position, fill = factor(Chromosome)), alpha = 0.6) + 
  facet_wrap(~factor(Chromosome), scales = "free_x") + 
  labs(title = "Maize SNP Distribution on Each Chromosome", 
       x = "Position on Chromosome", 
       y = "Density",
       fill = "Chromosome") +
  theme_minimal() +
  theme(plot.title = element_text(size = 16, face = "bold"),
        axis.title = element_text(size = 14))

# SNP Distribution Density per Chromosome for Teosinte
ggplot(data = teosinte_snp_genotype_clean) + 
  geom_density(mapping = aes(x = Position, fill = factor(Chromosome)), alpha = 0.6) + 
  facet_wrap(~factor(Chromosome), scales = "free_x") + 
  labs(title = "Teosinte SNP Distribution on Each Chromosome", 
       x = "Position on Chromosome", 
       y = "Density",
       fill = "Chromosome") +
  theme_minimal() +
  theme(plot.title = element_text(size = 16, face = "bold"),
        axis.title = element_text(size = 14))

# SNP Count per Chromosome for Maize and Teosinte
maize_snp_counts <- maize_snp_genotype_clean %>%
  group_by(Chromosome) %>%
  summarise(SNP_count = n())

teosinte_snp_counts <- teosinte_snp_genotype_clean %>%
  group_by(Chromosome) %>%
  summarise(SNP_count = n())

# Combine maize and teosinte SNP counts
snp_count_comparison <- bind_rows(
  maize_snp_counts %>% mutate(Source = "Maize"),
  teosinte_snp_counts %>% mutate(Source = "Teosinte")
)

# Bar plot for SNP counts per chromosome
ggplot(data = snp_count_comparison, aes(x = factor(Chromosome), y = SNP_count, fill = Source)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(title = "SNP Counts by Chromosome for Maize and Teosinte",
       x = "Chromosome",
       y = "SNP Count",
       fill = "Source") +
  scale_fill_manual(values = c("Maize" = "#E69F00", "Teosinte" = "#56B4E9")) +
  theme_minimal() +
  theme(plot.title = element_text(size = 16, face = "bold"),
        axis.title = element_text(size = 14),
        legend.title = element_text(size = 14))
```

# Spread and variability of SNP positions of each chromosome using a boxplot
# Maize
```{r}
library(ggplot2)
maize_snp_genotype$Chromosome <- factor(
  maize_snp_genotype$Chromosome,
  levels = as.character(1:10),
  ordered = TRUE
)

ggplot(maize_snp_genotype, aes(x = Chromosome, y = as.numeric(Position), fill = Chromosome)) +
  geom_boxplot() +
  theme_minimal() +
  labs(x = "Chromosome", y = "Position", title = "Boxplot by Chromosome")
```
# Teosinte
```{r}
library(ggplot2)
teosinte_snp_genotype$Chromosome <- factor(
  teosinte_snp_genotype$Chromosome,
  levels = as.character(1:10),
  ordered = TRUE
)

ggplot(teosinte_snp_genotype, aes(x = Chromosome, y = as.numeric(Position), fill = Chromosome)) +
  geom_boxplot() +
  theme_minimal() +
  labs(x = "Chromosome", y = "Position", title = "Boxplot by Chromosome")
```

# Missing data and amount of homozygous,heterozygous and missing
```{r}
# Load necessary libraries
library(dplyr)
library(tidyr)
library(ggplot2)
library(readr)
```

# Step 1: Load and preprocess data
# Assuming your data is already loaded in the following data frames:
# - fang_et_al_genotypes
# - snp_position
```{r}
# Subset maize and teosinte data groups
maize_data_group <- fang_et_al_genotypes %>%
  filter(Group %in% c("ZMMIL", "ZMMLR", "ZMMMR"))

teosinte_data_group <- fang_et_al_genotypes %>%
  filter(Group %in% c("ZMPBA", "ZMPIL", "ZMPJA"))

# Remove columns 1 to 3 and transpose the data
maize_genotypes_trans <- maize_data_group %>%
  select(-(1:3)) %>%
  t()

teosinte_genotypes_trans <- teosinte_data_group %>%
  select(-(1:3)) %>%
  t()
```

```{r}
# Arrange snp_position data and combine with genotype data
snp_position_Chr <- snp_position %>%
  select(-c(2, 5:15)) %>%
  arrange(SNP_ID)

maize_snp_genotype <- cbind(snp_position_Chr, maize_genotypes_trans)
teosinte_snp_genotype <- cbind(snp_position_Chr, teosinte_genotypes_trans)

# Replace '?' with '-' in the genotype data
maize_snp_genotype <- maize_snp_genotype %>%
  mutate_all(~ gsub("\\?", "-", .))

teosinte_snp_genotype <- teosinte_snp_genotype %>%
  mutate_all(~ gsub("\\?", "-", .))
```

```{r}
# Step 2: Add zygosity classification
add_zygosity <- function(data) {
  data %>%
    pivot_longer(cols = -c(SNP_ID, Chromosome, Position), 
                 names_to = "Sample", values_to = "Genotype") %>%
    mutate(Zygosity = ifelse(Genotype %in% c("A/A", "C/C", "G/G", "T/T"), "Homozygous",
                             ifelse(Genotype == "-/-", "Missing", "Heterozygous")))
}

maize_snp_long <- add_zygosity(maize_snp_genotype) %>% mutate(Group = "Maize")
teosinte_snp_long <- add_zygosity(teosinte_snp_genotype) %>% mutate(Group = "Teosinte")

# Combine data for joint analysis
combined_data <- bind_rows(maize_snp_long, teosinte_snp_long)
```

```{r}
# Step 3: Calculate proportions for each sample
proportions <- combined_data %>%
  group_by(Group, Sample, Zygosity) %>%
  summarise(Count = n(), .groups = "drop") %>%
  group_by(Group, Sample) %>%
  mutate(Proportion = Count / sum(Count))
```

```{r}
# Step 4: Plot zygosity proportions for each sample
ggplot(proportions, aes(x = Sample, y = Proportion, fill = Zygosity)) +
  geom_bar(stat = "identity", position = "fill") +
  facet_wrap(~ Group, scales = "free_x", ncol = 1) +
  labs(title = "Proportion of Zygosity by Sample for Maize and Teosinte",
       x = "Sample", y = "Proportion", fill = "Zygosity") +
  theme_minimal() +
  theme(axis.text.x = element_blank(),
        plot.title = element_text(size = 18, face = "bold"),
        axis.title = element_text(size = 14),
        legend.title = element_text(size = 14))

# Step 5: Calculate proportions for each group
group_proportions <- combined_data %>%
  group_by(Group, Zygosity) %>%
  summarise(Count = n(), .groups = "drop") %>%
  group_by(Group) %>%
  mutate(Proportion = Count / sum(Count))
```

```{r}
# Step 6: Plot zygosity proportions by group
ggplot(group_proportions, aes(x = Group, y = Proportion, fill = Zygosity)) +
  geom_bar(stat = "identity", position = "fill") +
  labs(title = "Proportion of Zygosity by Group (Maize and Teosinte)",
       x = "Group", y = "Proportion", fill = "Zygosity") +
  theme_minimal() +
  theme(plot.title = element_text(size = 18, face = "bold"),
        axis.title = element_text(size = 14),
        legend.title = element_text(size = 14))
```

```{r}
# Step 7: Save outputs (optional)
write_tsv(maize_snp_genotype, "processed_maize_snp_genotype.tsv")
write_tsv(teosinte_snp_genotype, "processed_teosinte_snp_genotype.tsv")
```
