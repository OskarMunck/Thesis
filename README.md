# Data-Driven Podcast Advertising: A Novel Framework
## Exploring the Potential of Topic Modelling and Text Segmentation for Native Ad Placement

Below follows some initial analyis.

## Repository file structure
0: Preprocessing and data wrangling  
1: Explorative data analysis  
2: Topic modelling -  our own implementation of the BERTopic pipeline  
3: Analysis of topic models  
4: Modified topic tiling implementation to be used with transformer based topic modelling  
5: Assign topics to segments of podcasts  
6: Code not used in final version of project


## Project summary
The growing podcast industry necessitates innovative advertising strategies, prompting this study to explore the use of data science methods to enable native advertisement placement, in which ads align with the surrounding content. This research seeks to locate advertisement spots at points of topical shifts as well as assign meaningful topics to the content surrounding those shifts in podcast transcripts. A transformer-based clustering approach, integrated with a text segmentation algorithm, is developed for this purpose, advancing previous literature in text segmentation. By modelling the Spotify Podcast Dataset, the developed methodology’s ability to identify meaningful advertisement spots in podcasts and assign topics from the corpus to these segments is validated. This proof-of- concept study not only technically enables native advertising but also proposes a business framework for its monetization, outlining potential integration into podcast platforms. The study also positions the relevance of the methodology in relation to network effects and platform theory.


### Research questions
RQ 1: How can data science methods locate topical shifts in podcast transcripts?  
RQ 2: How can data science methods find topics that are meaningful for podcast advertisement?


## Dataset 
In order to explore these two quesitons, we have used the Spotify Podcast Dataset which have the following properties: 

* Number of unique shows: 18290
* Number of unique publishers: 17490
* Languages: 20
* Number of episodes: 105360
* Max no episodes of one show: 1072, Min: 1
* Number of shows represented by only one episode: 8632
* Number of shows represented by less than 10 episodes: 16354

The dataset contains transcriptions of 105 360 podcasts from mainly English speaking shows from 18 290 different shows.

![duration dist](Images/duration_dist.png)

Almost all podcasts have a duration of less than 90 minutes. A uniform distibution up to ~60 minutes is prevalent.

In the data we see that there are a large discrepancy in topic distributions assigned by the authors: 

![category distribution](Images/categories.png)

We are going to have to subset the data due to computational constraints. 
Lets investigate the word count and duration distributions of the two largest categories to find an appropriate subset.

![distributions](Images/eduvssport.png)

We see that even though the education category contain more transcripts, the sports category transcripts are on average longer. Therefore we have chosen to subset the data according to the sports category for all downstream tasks. 

Now when we have a suitable subset for the tasks that we want to perform, we can present our modelling framework.

![Modelling pipeline](thesis_figures/modelling_framework.png)

The first step after deviding all the transcripts into embedding-sized-documents are to embedd them using the all-MiniLM-L6-v2 pre-traines s-BERT from huggingface. After that we reduced dimensionality in a two step approach using PCA and t-SNE to limit computational complexity in succeeding steps.

![Embedings](Images/embeddings.png)

After applying HDBSCAN to the embeddings with three different configurations of the hyperparamteres, we get the following clusters after removing noise: 

![Embedings](Images/clusters.png)

The topic cluster sizes have the following distribution of the 25 top topics of each HDBSCAN model: 

![topic distributions](Images/topic_model_dists.png)

Some stats about the clusters from the three cluster models

| mpts        | No. clusters   | Noise  | Largest cluster | Silhouette score
| ------------- |:-------------:| :-----:|:----: | :-----:
| 15      | 2881 |  179935| 4521 | 0.4315
| 50      | 556      |   185500 |11970 | 0.3564
| 200 | 156      | 181389 |13255 | 0.3625


After applying TopicTiling* according to our modified version that makes use of topic probability density vectors from the transformer based clustering with HDBSCAN, we get the following WindowDiff score when testing out different configurations of the window paramteter in TopicTiling* and the Mpts hyperparameter of HDBSCAN: 

![Window_diff model eval](Images/model_selection.png)

based on the above plot we see that different m_pts configuraitons barely make any difference but altering the window parameter does. We observe a negative exponential relationship between WindowDiff and window size for all models. After the window is increased to 20, we see less of an improvment. Therefore, we have selected a model with window size = 20 for all forthcomming evaluation. 

**To read the results of the topic assignments to segments and conclusions of our thesis, please see the pdf: [MSc_thesis](masters_thesis_final.pdf)**

### A note on hyperparameter tuning for future iterations of TopicTiling*

In the table below the impact of cluster model configuration on cosine similarity range size and average depth scores can be observed across the full annotation set for the three models. The average depth score for the three models increases as mpts shifts from 15 to 50 and then decreases again as mpts go to 200. The cosine similarity range is approximately the same for the two models with mpts set to 15 and 50 and then drops down for the mpts 200 model. A high average depth score in relation to the average cosine similarity range means that the model produces high local maxima and low local minima relative to the cosine similarity range used. This could implicate that the model is more robust against randomness in local variability of the sequential cosine similarities of a transcript.

| mpts | Avg. cos(θ) range size   | Avg. depth score | 
| ---- |:-------------:| :-----:|
| 15  | 0.714 | 0.456| 
| 50  | 0.720 | 0.551 |
| 200 | 0.664 | 0.448 |

To illustrate an example of this, we have in the below figure plotted the cosine similarity between windows over all positions sequentially for one of the annotated transcripts as the blue line. The red crosses mark where boundaries are predicted to be situated and the actual boundaries are shown as yellow vertical lines. As seen in the accompanying table which displays the descriptive statistics, as mpts increases from 15 to 200 the range the cosine similarity takes on increases from 0.811 to 0.888, and the depth score average increase from 0.339 to 0.431. Clustering with mpts = 50 gets a range of 0.862 and an average depth score of 0.487. As can be observed in the rightmost column is that for the two models where the range is smaller and the average depth score is higher (mpts = {50, 200}), the models are better at placing segmentation boundaries as evidenced by the WindowDiff scores. However, this can only serve as a demonstration of how the cluster model may impact TopicTiling*. To make any conclusions about this finding, an evaluation of a larger set of annotated data would be required than what is possible in this study.

| mpts | Avg. cos(θ) range size   | Avg. depth score | WD
| ---- |:-------------:| :-----:| :----: 
| 15  | 0.811 | 0.339| 0.26s
| 50  | 0.862 | 0.487 |0.22
| 200 | 0.888 | 0.431 |0.22


![Cosine sim ex](Images/cosine_sim_ex.png)