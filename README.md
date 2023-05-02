# Transformer Based Topic Modelling and Dynamic Segmentation of Speech-to-Text Transcribed Data
## Applications for Effective Podcast Monetization

Below follows some initial analyis.

## Repository file structure
0: Preprocessing and data wrangling  
1: Explorative data analysis  
2: Topic modelling -  our own implementation of the BERTopic pipeline  
3: Analysis of topic models  
4: Modified topic tiling implementation to be used with transformer based topic modelling  

## Some descriptive stats of the metadata

**Data in numbers:**
Number of unique values
* show_name    18290
* show_description    18322
* publisher    17490
* language    20
* episode_uri    105360
* episode_name    103660
* Max no episodes of one show    1072, Min    1
* Number of shows represented by only one episode    8632
* Number of shows represented by less than 10 episodes    16354

So, among else, we see that we have 105 360 podcasts from 18 290 different shows. 

![duration dist](Images/duration_dist.png)

Almost all podcasts have a duration of less than 90 minutes. A uniform distibution up to ~60 minutes is prevalent.

In the data we see that there are a large discrepancy in topic distributions assigned by the authors: 

![category distribution](Images/categories.png)

We are going to have to subset the data due to computational constraints. 
Lets investigate the word count and duration distributions of the two largest categories to find an appropriate subset.

![distributions](Images/eduvssport.png)

We see that even though the education category contain more transcripts, the sports category transcripts are on average longer. Therefore we have chosen to subset the data according to the sports category for all downstream tasks. 

Now when we have a suitable subset for the tasks that we want to perform, we can present our modelling pipeline.

![Modelling pipeline](Images/modelling_pipeline.png)

Embedding the documents with BERT and reducing dimensionality in a two step approach using PCA and t-SNE. 

![Embedings](Images/embeddings.png)

After applying HDBSCAN to the embeddings with three different configurations of the hyperparamteres, we get the following clusters after removing noise: 

![Embedings](Images/clusters.png)

The topic cluster sizes have the following distribution of the 25 top topics of each HDBSCAN model: 

![topic distributions](Images/topic_model_dists.png)

After applying TopicTiling* according to our modified version that makes use of topic probability density vectors from the transformer based clustering with HDBSCAN, we get the following WindowDiff score when testing out different configurations of the window paramteter in TopicTiling* and the Mpts hyperparameter of HDBSCAN: 

![Window_diff model eval](Images/model_selection.png)

based on the above plot we see that different m_pts configuraitons barely make any difference but altering the window parameter does. We observe a negative exponential relationship between WindowDiff and window size for all models. After the window is increased to 20, we see less of an improvment. Therefore, we have selected a model with window size = 20 for all forthcomming evaluation. 

As previously discussed, increasing or decreasing the value of m_pts in HDBSCAN did not improve the WindowDiff score. However, it does increase the size of the drops in cosine similarity that segment breaks are predicted at and the number of predicted boundaries. In the figure below we observe the cosine similarity between windows over all positions sequentially plotted for one of the annotated transcripts as the blue line. The red crosses describe where boundaries are predicted to be allocated and the actual boundaries are shown as yellow vertical lines. For these three pipeline configurations, the number of predicted breaks decrease as the minimum cluster size increase. For m_pts=15 we get 27 boundaries, for m_pts=50 we get 24 and for m_pts=200 we get 20 in this specific transcript consisting of 600 sentences. Furthermore, as m_pts increase from 15 to 200, the range that the cosine similarity takes on increase from [0.02, 0.831] to [0.045, 0.934]. Clustering with m_pts= 50 gets a range with values in the middle of the two.

![Cosine sim ex](Images/cosine_sim_ex.png)

## Sit tight! More to come, work in progress :) 