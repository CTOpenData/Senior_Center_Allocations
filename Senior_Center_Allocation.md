# Senior Center ARPA Allocation Dataset Creation

#### Installing and Loading Packages

Run: install.packages(“tidycensus”,“tidyverse”,“dplyr”) if packages are
not already installed

    library(tidycensus)
    library(tidyverse)
    library(dplyr)

#### Census API Key

If installing api key for the first time, also run
readRenviron(“~/.Renviron”) to save to your environment

    census_api_key("INSERT YOUR KEY")

#### Pulling and cleaning data from Census/ACS

The data set is created using the ACS 2016-2020 5-year estimates at the
county subdivision level. County subdivisions are equivalent to CT town
boundaries.

Tables Used:

-   Total Population by Town - ACS Table B01001
-   Population 60 and over by Sex - ACS Table B01001
-   White Alone, Non-Hispanic by Age and Sex for Population 65 and
    over - ACS Table B01001H
-   Population 65 and over with a disability - ACS Tables B18101
-   Population 65 and over below the poverty line - ACS Tables B17001

<!-- -->

    ct_aging_pop <- get_acs(
      geography = "county subdivision",
      variables = c(
                    total_pop = "B01001_001",
                    male_pop_60_61 = "B01001_018",
                    male_pop_62_64 = "B01001_019",
                    male_pop_65_66 = "B01001_020", 
                    male_pop_67_69 = "B01001_021", 
                    male_pop_70_74 = "B01001_022",
                    male_pop_75_79 = "B01001_023", 
                    male_pop_80_84 = "B01001_024", 
                    male_pop_85_over = "B01001_025",
                    female_pop_60_61 = "B01001_042",
                    female_pop_62_64 = "B01001_043",
                    female_pop_65_66 = "B01001_044", 
                    female_pop_67_69 = "B01001_045", 
                    female_pop_70_74 = "B01001_046",
                    female_pop_75_79 = "B01001_047", 
                    female_pop_80_84 = "B01001_048", 
                    female_pop_85_over = "B01001_049",
                    male_white_65_74 = "B01001H_014", 
                    male_white_75_84 = "B01001H_015", 
                    male_white_85_over = "B01001H_016",
                    female_white_65_74 = "B01001H_029", 
                    female_white_75_84 = "B01001H_030", 
                    female_white_85_over = "B01001H_031",
                    disability_65_74 = "B18101_016",
                    disability_75_over = "B18101_019",
                    male_below_pov_65_74 = "B17001_015",
                    male_below_pov_75_over = "B17001_016",
                    female_below_pov_65_74 = "B17001_029",
                    female_below_pov_75_over = "B17001_030",
                    male_above_pov_65_74 = "B17001_044",
                    male_above_pov_75_over = "B17001_045",
                    female_above_pov_65_74 = "B17001_058",
                    female_above_pov_75_over = "B17001_059"
                    ),
      state = "CT",
      year = 2020) %>%
      
      #remove MOE and GEOID fields and reformat data from wide to long by town and variable
      pivot_wider(id_cols = NAME,
                  names_from = variable,
                  values_from = estimate) %>%
      
      #creating 60 and over and 65 and over estimates by sex
      mutate(male_pop_60_over = select(.,male_pop_60_61:male_pop_85_over) %>% 
              rowSums(na.rm = TRUE)) %>%
      mutate(female_pop_60_over = select(.,female_pop_60_61:female_pop_85_over) %>%
              rowSums(na.rm = TRUE)) %>%
      mutate(male_pop_65_over = select(.,male_pop_65_66:male_pop_85_over) %>% 
              rowSums(na.rm = TRUE)) %>%
      mutate(female_pop_65_over = select(.,female_pop_65_66:female_pop_85_over) %>%
              rowSums(na.rm = TRUE)) %>%
      
      #Creating total population estimates for 60 and over and 65 and over
      mutate(pop_60_over = male_pop_60_over + female_pop_60_over) %>% 
      mutate(pop_65_over = male_pop_65_over + female_pop_65_over) %>% 
      
      #Creating total population estimates for 60 and over and 65 and over by race/ethnicity
      mutate(white_65_over = select(.,male_white_65_74:female_white_85_over) %>% 
              rowSums(na.rm = TRUE)) %>%
      mutate(non_white_65_over = pop_65_over-white_65_over ) %>% 
      
      #Creating estimate for population 65 and over with a disability
      mutate(disability_65_over = disability_65_74 + disability_75_over) %>% 
      
      #Creating estimate for population 65 and over and below poverty line
      mutate(male_below_poverty = select(.,male_below_pov_65_74:male_below_pov_75_over) %>% 
               rowSums(na.rm = TRUE)) %>%
      mutate(female_below_poverty = female_below_pov_65_74 + female_below_pov_75_over) %>%
      mutate(below_poverty = male_below_poverty + female_below_poverty) %>%
      
      mutate(male_above_poverty = select(.,male_above_pov_65_74:male_above_pov_75_over) %>% 
               rowSums(na.rm = TRUE)) %>%
      mutate(female_above_poverty = female_above_pov_65_74 + female_above_pov_75_over) %>%
      mutate(above_poverty = male_above_poverty + female_above_poverty) %>%

      #removing the empty "subdivisions not defined" rows
      filter(!grepl('County subdivisions not defined', NAME))%>%
      
      # Dropping everything after town name
      # (ex. "Bethel town, Fairfield County, Connecticut" is now just "Bethel")
      mutate(NAME = gsub(" town.*$","", NAME)) %>%
      rename(Town = NAME) %>%
      
      #Dropping all of the calculation variables
      select(-c(male_pop_60_61,
                male_pop_62_64,
                male_pop_65_66, 
                male_pop_67_69, 
                male_pop_70_74,
                male_pop_75_79, 
                male_pop_80_84, 
                male_pop_85_over,
                female_pop_60_61,
                female_pop_62_64,
                female_pop_65_66, 
                female_pop_67_69, 
                female_pop_70_74,
                female_pop_75_79, 
                female_pop_80_84, 
                female_pop_85_over,
                male_white_65_74, 
                male_white_75_84, 
                male_white_85_over,
                female_white_65_74, 
                female_white_75_84, 
                female_white_85_over,
                disability_65_74,
                disability_75_over,
                male_below_pov_65_74,
                male_below_pov_75_over,
                female_below_pov_65_74,
                female_below_pov_75_over,
                male_above_pov_65_74,
                male_above_pov_75_over,
                female_above_pov_65_74,
                female_above_pov_75_over,
                male_pop_60_over,
                female_pop_60_over, 
                male_pop_65_over, 
                female_pop_65_over,
                white_65_over,
                total_pop,
                male_below_poverty,
                female_below_poverty,
                male_above_poverty,
                female_above_poverty
                ))

##### Loading in the Census Rural Designation file

    rural_urban <- read_csv("//opm-fs102/UserRedirections/WonderlyC/Downloads/DECENNIALSF12010.P2-Data.csv")

    #Cleaning file and calculating share urban and rural
    rural_urban <- rural_urban %>%
      mutate(share_rural = `Total!!Rural`/Total) %>%
      mutate(share_urban = `Total!!Urban`/Total) %>%
      rename(Town = `Geographic Area Name`) %>% 
      filter(!grepl('County subdivisions not defined', Town)) %>%
      mutate(Town = gsub(" town.*$","", Town)) %>%
      select(-c(Geography, Total:`Total!!Not defined for this file`))
      
    #Adding the urban and rural shares to the aging population file and calculating the estimates rural and urban by population for 60 and over and 65 and over
    ct_aging_pop <- merge(ct_aging_pop,rural_urban,by="Town")
    ct_aging_pop <- ct_aging_pop %>%
      mutate(urban_estimate_60_over = share_urban * pop_60_over) %>%
      mutate(rural_estimate_60_over = share_rural * pop_60_over) %>%
      mutate(urban_estimate_65_over = share_urban * pop_65_over) %>%
      mutate(rural_estimate_65_over = share_rural * pop_65_over)
