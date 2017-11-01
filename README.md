# data-512-a2
DATA 512 Human-Centered Data Science A2 Project: Bias in Data

The goal of this assignment is to detect bias on political figures from multiple countries by analyzing of the coverage and the quality of the the politican's Wikipedia articles. The main output of this project is a data visualization of highest-ranked and lowest-ranked countries in term of politican article density. The visuals itself does not tell the entire story, but it hopefully helps uncovering political biases in Wikipedia.
To support the analysis, I produce these artifacts:
- A CSV file with politicans Wikipedia article information (country, article name, revision ID, article quality, and population of the country)
- A Jupyter notebook with all the code
- A comprehensive README file
- A MIT LICENSE file
- A visualization png file

To be able to effectively process and analyze the data, this project requires:
- Politicians Wikipedia dataset
- Population data of each country
- Article quality prediction by Wikimedia ORES machine learning service

The process to turn these datasets into those deliverables as follow:
1. Establish shared library
2. Get article and population datasets
3. Get article quality prediction
4. Combine the datasets
5. Analysis
6. Visualization
7. Results


### Shared Library

This project's requirements are pandas and requests Python packages.


## Getting Article & Population Data

The article dataset CSV file can be downloaded on the figshare(https://figshare.com/articles/Untitled_Item/5513449), while the population dataset CSV file is available on the Population Research Bureau website (http://www.prb.org/DataFinder/Topic/Rankings.aspx?ind=14). This step is pretty straight-forward as long as the correct dataset are downloaded (the try-catch is to make sure it is available in the current directory).


## Getting article quality predictions and Combining the Datasets

Each Wikipedia article is classified into one of 6 categories by ORES (Objective Revision Evaluation Service) machine learning service. These categoris from best to worst are:
FA: Featured article
GA: Good article
B: B-class article
C: C-class article
Start: Start-class article
Stub: Stub-class article

ORES predicts probability number of each category for every article, and it picks category with the highest probability as the article's category. Fortunately, ORES has a public ReST API that accepts an article's revision ID or a batch of articles' revision IDs (up to 140), and it returns the prediction value of said article with a the full set of probability for each category.

After being loaded, each dataset is cleaned and combined by dropping unnecessary columns (Location Type, TimeFrame, Data Type, Foornotes from population dataset), by renaming remaining columns to the specification, and by being inner-joined on *country* attribute (orphaned rows are filtered out). 

The cleaned dataset is split into subsets of 100-rows since ORES API only takes roughly up to 140 revision IDs in one single batch request. The resulting article quality dataset is then merged to the main dataset on the *revision_id* before everything is stored into a CSV file.

Manual testing is added at the end to check the integrity of the dataset as well as its accuracy (I chose Indonesia since it's my birth country and I am pretty familiar with the names of the politician). 


## Analysis

In the analysis part, we have 2 metrics to calculate:
1. The proportion of the number of each country articles per its population per country.
2. The percentage of high-quality articles (either FA or GA article) over total articles.
Using these two metrics, I hopefully am able to prove or disprove the notion there is biases toward certain country politicians in English Wikipedia.

Proportion of articles per country population is calculated by dividing the number of articles of each country with the country population. This can be achieved by grouping the dataset by its *country* and summing the number of the articles. This new groupby dataset is divided by the country's *population* from the population dataset. Finding the highest and lowest proportion is only a matter of sorting.

Calculating high-quality articles are a little trickier since I need to define a boolean function to determine whether each article a high-quality one or not. Aside from that extra function, the number of high-quality articles proportioned to the number of all articles is straight-forward.

To test this analysis module, I select all Indonesian politicians with high-quality articles. The number is disappointingly very low in proportion to population of Indonesia. Countries with high and low proportions will be discussed in the next section.


## Visualization

This section is to visualize the 10 highest-ranked and lowest-ranked countries in term of number of politician articles as proportion of country population and the number of high-quality articles as proportion of all articles. Before I start any plotting, I need to make sure I the countries are ranked appropriately. There are 39 countries with 0 high-quality articles; these countries are tied up for the last place in high-quality articles per total articles rank.
While having the lowest and highest-ranked countries in the same bar chart could be useful, this is not the case since the lowest-ranked countries have much smaller articles-per-population than the highest-ranked. Hence, I split the lowest-ranked countries and highest-ranked countries in different plot.


## Results

### number of politician articles as a proportion of country population
The 10 lowest-ranked country in terms of number of politician articles as a proportion of country population in Wikipedia is dominated by countries with big population (China, India, Indonesia) or by countries where English-speaking editors are less familiar with, such as Bangladesh, Zambia or by countries with closed political system, like North Korea. On the other hand, the top 10 country in terms of number of politician articles as a proportion of country population consists exclusively of countries with very small population. While this metric sheds a light on  bias against politicians from less-familiar countries in Wikipedia, I found the metric/analysis itself has inherently flawed against countries with big population and vice versa. It is by no means an indication that English-speaking Wikipedia biased against China, India, and Indonesia. Afterall, China(1138 articles), India(990 articles), and Indonesia(215 articles) have decent number of total articles in English Wikipedia, and there is no linear correlation between total number of articles and population.

### number of high-quality articles as a proportion of all articles about politicians from that country
The second metric filters out "placeholder" and "empty calories" articles about politicians to reduce the noise. The hypotheses is biases would be more pronounced when there is only a handful relevant articles. However, there is no obvious bias against the lowest-ranked countries in this metric (excluding 39 countries without good or featured article). It consists of countries from 4 different continents with various population (Nigeria has the largest population, tiny Luxembourg has the smalles one), developed economically (Finland, Luxembourg) or developing economically (Nigeria, Czech Republic). The only similarities among these countries is English is not the primary or even secondary everyday language. The other end of the spectrum also shows very little bias. Although United States is in the top 10, it does not even have the highest proportion of well-written articles to all articles. That distinction surprisingly belongs to North Korea by a wide margin!! 9 out of 39 articles about the Hermit Kingdom's politicians are deemed good or featured article. It is even more surprising considering North Korea is in the bottom 10 of number of politician articles as proportion of country population since there is not that many articles. I am under the impression that a North Korean politic expert (or a group of researchers) is dedicated to write most, if not all, of North Korea politician articles. A cursory glance at the articles' edit history confirm that WIkiproject North Korea makes significant contribution to the North Korean-related Wikipedia articles.

