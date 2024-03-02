---
title: 'Podcast and Song Composer'
date: March 1st, 2024
permalink: /posts/2012/08/blog-post-4/
tags:
  - NLP
  - Semantic Search
---


Overview:
======
This project used the Spotify API to obtain song and pocast data, as well as the Genius API to obtain song lyrics. The first part involved clustering songs based on their Spotify audio features using gaussian mixture models (GMM). Despite the natural approach to cluster by genre, the distribution of genres within each cluster did not exhibit discernible patterns. To interpret the clusters, decision trees were employed, with attention given to managing the tree's depth to balance interpretability and accuracy. This project also aimed to create a curated blend of podcast episodes and songs based on shared themes and moods. The process involved converting song data into textual representations, encompassing key audio features and summarized lyrical content, then encoding podcast and song descriptions using a pre-trained sentence-BERT model. 

The Data:
======  

Podcast Episodes:  
------
The 10 most recent episodes of the top 100 Spotify podcasts of 2023 were pulled. The data includes podcast name, episode name, podcast descriptions, and episode descriptions.

Songs:
------
The Spotify API does not provide the genre of a specific song. Rather, there are genres associated with the artist and sometimes associated with an album. The genres can also be very specific and sometimes even specify the state or city, such as “Ohio Pop” and “Toronto Pop”. To simplify the data collection, I downloaded and used the top 100 songs of various genres from [TopCharts](https://www.top-charts.com/songs/alternative/united-states/total/2021-W19) and [Shazam](https://www.shazam.com/charts/genre/united-states/country). This also guaranteed a collection of songs from less mainstream genres such as blues, jazz, and gospel. In total, the data for 1,405 songs were collected. The distribution of genres is shown below:  

<div style="text-align:center">
  <img src="/images/genres.jpg" alt="Image" width="500" height="400" />
</div>


Song Audio Features:
------
Using the search function in the Spotify API, each track’s [audio features](https://developer.spotify.com/documentation/web-api/reference/get-audio-features) were pulled. The audio features are engineered features that Spotify provides. A summary of the relevant features is provided below:
  - **Acousticness**: A confidence measure from 0.0 to 1.0 of whether the track is acoustic.
  - **Danceability**: A measure from 0.0 to 1.0 of a track’s suitability for dancing.
  - **Energy**: A measure from 0.0 to 1.0 of a perceptual measure of intensity and activity.
  - **Instrumentalness**: A confidence measure from 0.0 to 1.0 that predicts whether a track contains no vocals. A value greater than 0.5 represents instrumental tracks, but confidence is higher as the value approaches 1.0. 
  - **Liveness**: Detects the presence of an audience in the recording. A value above 0.8 provides strong likelihood that the track is live.
  - **Loudness**: A measure of the average loudness across the entire track ranging from -60 to 0 decibels. 
  - **Speechiness**: A measure from 0.0 to 1.0 that indicates the presences of spoken words in a track. A value less than 0.33 likely represents music, whereas a value greater than 0.66 likely represents tracks that are likely entirely spoken words. 
  - **Time Signature**: An estimated time signature - a notational convention to specify how many beats are in each bar (or measure). The time signature ranges from 3 to 7 indicating time signatures of "3/4" to "7/4".
  - **Tempo**: The overall estimated tempo of track in beats per minute. 
  - **Valence**: A measure from 0.0 to 1.0 measuring the musical positiveness conveyed by a track. Tracks with high valence sound more positive (e g. happy, cheerful, euphoric), while tracks with low valence sound more negative (e.g. sad, depressed, angry).

Song Lyrics:
------
In addition to song information being pulled from Spotify, the lyrics for each song were obtained using the [Genius API](https://docs.genius.com/). 

Design and Results:  
======  

Part 1: Interpreting High Dimensional Clusters
------  

The tabular data (Spotify audio features) was first scaled using Standard Scaler to be of zero mean and unit variance. The songs were then clustered using Gaussian Mixture Models (GMM), with 5 clusters chosen for an illustrative example. While humans may naturally categorize songs by genre, the distribution of genres within each group showed no discernable pattern, as shown below: 

<div style="text-align:center">
  <img src="/images/genres_per_group.jpg" alt="Image" width="700" height="500" />
</div>

While dimension reduction techniques such as Principal Component Analysis (PCA) are commonly used to interpret clusters, they do not always preserve the high dimensional structure of the data and can be difficult to interpret. Instead, I opted for decision trees, as they naturally partition the feature space. However, this approach is not without limitations. One concern is the depth of the decision tree, which determines the number of decision nodes and therefore adds complexity to the resulting interpretation. To address this, I developed a simple function to access the tree’s performance across various *max_depth* settings and identified the optimal depth that yields the highest accuracy within a user-defined range. Using this method, the tree’s complexity is reduced to a more manageable depth:

<div style="text-align:center">
  <img src="/images/accuracy_plot.jpg" alt="Image" width="800" height="600" />
</div>

The final visualization of the tree model was facilitated using the [tree.plot_tree](https://scikit-learn.org/stable/modules/generated/sklearn.tree.plot_tree.html) function:   

<div style="text-align:center">
  <img src="/images/song_tree.jpg" alt="Image" width="800" height="600" />
</div>

With a test accuracy of 95.73%, the class-wise accuracy is shown below:  

<div style="text-align:center">
  <img src="/images/class_accuracy.jpg" alt="Image" width="250" height="150" />
</div>

As stated above, the features were standardized and therefore the threshold values depicted below are of ~N(0,1). These values were subsequently converted back to their original scale for the following summary:

**Cluster 0**:  
 - Non-instrumental tracks with a time signature between 3.5 and 4.5    
(Instrumentalness = 0, 3.5 < Time signature <= 4.5)  

**Cluster 1**:  
 - Non-instrumental tracks with few vocals and a time signature less than 3.5  
(Instrumentalness = 0, Speechiness ≤ 0.05, Time signature ≤ 3.5)  
 - Tracks with a tempo greater than 185 and a time signature less than 3.5  
(Instrumentalness > 0, Tempo > 185, Time signature ≤ 3.5)    

**Cluster 2**:  
 - Tracks with a tempo less than 185 and a time signature less than 3.5  
(Instrumentalness > 0, Tempo ≤ 185, Time signature ≤ 3.5)  
 - Tracks that are likely to be live with a time signature greater than 3.5.  
(Instrumentalness > 0, Liveness > 0.83, Time signature > 3.5)  

**Cluster 3**:    
 - Non-instrumental tracks with some vocals and time signature less than 3.5.  
(Instrumentalness = 0, Speechiness > 0.05, Time signature ≤ 3.5)     
 - Non-instrumental tracks with a time signature greater than 4.5    
(Instrumentalness = 0, Time signature > 4.5)     

**Cluster 4**:   
 - Tracks with a time signature less than 3.5   
(Instrumentalness > 0, Liveness ≤ 0.83, Time signature > 3.5)     

Although this does provide insight into the clusters, there are some limitations with the provided tree. Where instrumentalness is a confidence measure ranging from 0.0-1.0, an instrumental threshold of 0 implies songs with a value ≤ 0 are *not* instrumental, but the level of instrumentalness for songs with a value > 0 is ambiguous. Similarly, a threshold value of 0.05 for speechiness provides little information for songs with a value > 0.05. 

Part 2: Semantic Search for Podcast and Song Pairings
------

The goal for the second part of this project was to create a blend of podcast episodes and songs based on similar themes and mood. To do this, I used a pre-trained sentence-BERT model, [all-mpnet-base-v2](https://www.sbert.net/docs/pretrained_models.html), to encode all podcast descriptions and find the top three matching songs for each episode. While podcast descriptions are readily available in natural language form, the song data necessitated conversion from tabular to a text represenation. The audio features used in this conversion included energy, danceability, loudness, valence, instrumentalness, and acousticness (see #1 below). While this representation is meant to gauge the overall mood of the song, it fails to address the thematic content of a song. 

To address this, the song lyrics were obtained via the Genius API. Initially a BART model trained on CNN Daily Mail, [facebook/vart-large-cnn](https://huggingface.co/facebook/bart-large-cnn), was used to summarize the lyrics. Where the BART model was trained on news articles, it did a poor job of extracting the abstract meaning of the lyrics, and sometimes produced summaries resembling news blurbs:

“Eyes Without a Face” by Billy Idol:  
  > *Billy idol's new album is out now and is out of print. The album is a collection of songs dedicated to his ex-girlfriend. Billy idol is currently on tour and will be performing in Las Vegas.*  

Subsequently, an alternative approach leveraging OpenAI's API to prompt *gpt-3.5-turbo-1106* was explored. This resulted in:

“Eyes Without a Face” by Billy Idol:  
  > *The song "Eyes Without a Face" talks about feelings of hopelessness, disillusionment, and betrayal. The lyrics express sadness and anger at being lied to and deceived, and a sense of loss and despair over a past love. The chorus, "les yeux sans visage" translates to "eyes without a face," symbolizing a lack of humanity and grace. The song also contains references to escapism, criminal behavior, and a sense of alienation.*

Although not perfect and prone to reiteration, these responses are clearly preferred. Thus, new string representations were created for each song using a combination of the ChatGPT API requests and the Spotify audio features, resulting in three distinct text formats:

1. **Audio Feature Description: "description"**    
Format: The song “*song name*” is in the “*genre*” genre. It is a “*audio features*”  
Example: *The song Anti-Hero is in the pop genre. It is a moderate energy, moderately danceable, low instrumental, low acousticness song.*

2. **ChatGPT Description: "new description"**    
Format: "*ChatGPT summary*"  
Example: *The song describes the singer's struggles with depression, self-awareness, and feeling like they are always the problem in their relationships. The lyrics also touch on themes of narcissism and self-destructive behavior. The chorus emphasizes the feeling of always being the problem and never being able to see oneself clearly.*

3. **Combined Description: "combined description"**    
Format: The song “*song name*” is in the “*genre*” genre. It is a “*audio features*”. “*ChatGPT summary*”

**RESULTS**  

The complete results can be found in the "podcasts_songs.csv" file [here](https://github.com/sirena-depue/Projects). A small snippet of the results are shown below to show the difference between each of the three prompts:  

<div style="text-align:center">
  <img src="/images/podcast_recs.png" alt="Image" width="800" height="600" />
</div>

**Audio Feature Description ("description"):**
As shown above, the song title significantly influences recommendations, leading to a lot of variability. Where “The Daily” is a news podcast, the song recommendations seem promising just based on the song titles. However, the song “Breaking News” discusses domestic violence and “Headlines” covers themes of money and success. Recommendations for shows like “It’s Not Only Football” suggest songs with titles referencing the night or weekend, while those for “Crime Junkie” lean towards title implying crime – a closer match to the podcast content, but still disregarding song audio features.

**ChatGPT Description ("new description"):** 
As shown above, while the ChatGPT description yields more promising results in terms of contextual content for some podcast episodes, they are not always consistent with the overall mood of the podcast. 

The songs “Pumped Up Kicks” and “Jeremy” discuss troubled youth committing violent acts in schools, which aligns well with the Daily Show Episode “A Guilt Verdict for a Mass Shooter’s Mother”. This song is frequently recommended among political and true crime podcasts. The song “They Own the Media” discusses the manipulation of media narratives by specific entities and is recommended frequently with political and conspiratorial podcasts:  

<div style="text-align:center">
  <img src="/images/song_counts.png" alt="Image" width="600" height="400" />
</div>

All songs recommended for “Crime Junkie” contain descriptions of criminal activity, often related to drugs. Despite sharing similar themes, the mood evoked by true crime stories may not align with the mood created by these songs. 

The recommendations for “It’s Not only Football” discuss enjoying good company and participating in recreational activities. Except for “Get you Mad” by Eminem, all suggested songs for the Sanctified podcast are gospel/worship songs. 


Future Improvements:   
======   

Part 1: Interpreting High Dimensional Clusters   
------  

As discussed earlier, the depth of a decision tree can effect the ease of interpreting clusters. Experimenting with different trees and manually parsing out the rules for each class is tedious and in an attempt to simplify this process, I reduced the depth to a satisfactory accuracy level. However, implementing a function to extract rules for each class would streamline the process and facilitate the assessment of the tree's interpretability. This would also allow the user to more easily experiment with trees of greater depth, which may also reduce the issue of ambiguity associated with very low or high threshold values in the decision nodes. 

Part 2: Semantic Search for Podcast and Song Pairings
------

The aim behind the ChatGPT responses was to pair podcasts and songs based on similar content. However, this did not always perform well, especially when some of podcast descriptions are overly broad or episode descriptions lack detail. Additionally, the current approach fails to consider the mood conveyed by both the songs and episodes. While using Spotify’s audio features (energy, danceability, etc.) was meant to address this, the results often prioritize song titles over these features. 

Additionally, there are podcast episodes where song recommendations are not obvious - even when made by humans. For example, the episode titled “AMA #15: Fluoride Benefits/Risks & Vagus Nerve Stimulation” by the "Huberman Lab", presents a challenge in suggesting a fitting song. However, episodes exploring more universal themes (i.e. love, loss) tend to yield more suitable recommendations. Finally, the quality of song suggestions is subjective and can vary according to individual preferences and opinions.

Some future improvements to address these shortcomings include:
 -	Collect more song data for increased diversity.
 -	Incorporate user feedback to account for personal preferences.
 -  Switch from the “gpt-3.5-turbo-1106” model to the "gpt-4" model (more expensive!) when requesting ChatGPT for more detailed summaries.
 -	If provided with the transcript of a podcast episode, can compensate for short episode descriptions by using ChatGPT to generate a summary. 
 -	If provided with the transcript of a podcast episode, perform sentiment analysis to gauge the overall mood, aligning with the Spotify audio feature, valence.  

Finally, although we can already see some promising results, there is no quantitative/qualitative measure of how well the semantic search performs overall. To address this, themes can be extracted from the podcast episodes (ex: politics, sports, relationships, religion). This may also require one-hot encoding for podcast episodes that discuss a mixture of themes. Then, the number of times a song is recommended for each theme can be tracked, giving a clearer idea of how songs are recommended. 

Although we can already see some promising results and some imporvement with the addition of lyric summarizations, there is no obvious way to measure the overall performance of the semantic search, especially since the quality of results are subjective. One proposed strategy is to extract themes from the podcast episodes such as politics, sports, relationships, and religion. This may require techniques such as one-hot encoding to handle episodes discussing a mixture of themes. Subsequently, we can track the frequency with which each song is recommended for various themes. By quantifying the recommendations across different thematic categories, we can gain insights into how well songs are recommended based on thematic relevance, thereby enabling informed improvements to the recommendation system. 