---
title: 'Spotify Song Recommender'
date: 2199-01-01
permalink: /posts/2012/08/blog-post-4/
tags:
  - NLP
  - Semantic Search
---


Motivation:
======

In the first part, I employ Gaussian Mixture Models (GMM) to cluster songs according to their audio features, and use decision trees to interpret these groupings. My aim is to explore the data and gain insight into how songs can be grouped.  

In the second part, I use various natural language processing (NLP) techniques to recommend songs based on the podcast episode descriptions and different string representations of songs. The primary objective is to create a blend of podcast episodes and songs based on similar themes, topics, and overall mood. 


The Data:
======  
This project uses data from both songs and podcasts obtained via the Spotify API.  

Podcast Episodes:  
------
The 10 most recent episodes of the top 100 Spotify podcasts of 2023 were pulled. The data includes podcast name, episode name, podcast descriptions, and episode descriptions.

Songs:
------
The Spotify API does not provide the genre of a specific song. Rather, there are genres associated with the artist and sometimes associated with an album. The genres can also be very specific and sometimes even specify the state or city, such as “Ohio Pop” and “Toronto Pop”. To simplify the data collection,  I downloaded and used the top 100 songs of various genres from [TopCharts](https://www.top-charts.com/songs/alternative/united-states/total/2021-W19) and [Shazam](https://www.shazam.com/charts/genre/united-states/country). This also guaranteed a collection of songs from less mainstream genres such as blues, jazz, and gospel. In total, the data for 1,405 songs were collected. The distribution of genres is shown below:  
![](/images/genres.jpg)

Song Audio Features:
------
Using the search function in the Spotify API, each track’s [audio features](https://developer.spotify.com/documentation/web-api/reference/get-audio-features) were pulled. The audio features are engineering features that Spotify provides. A summary of the relevant features is provided below:
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
In addition to song information being pulled from Spotify, the lyrics for each song were obtained using the Genius API. 

Design:  
======  

Part 1: Grouping Songs  
------  

To conduct some exploratory analysis, the songs were grouped using Gaussian Mixture Models (GMM) based on their Spotify audio features. As an example, I set the number of components to be 5. While humans may naturally categorize songs by genre, the distribution of genres within each group showed no discernable pattern, as shown below:  
![Alt Text](genres_per_group.jpg)  
Consequently, I used a decision tree classifier to gain insight into how the data was split into groups. One limitation of using a tree classifier to gain interpretability is the depth of the tree. To address this, I developed a simple function that evaluates the tree’s performance across varying max_depth settings. It then identifies the maximum accuracy and determines the accuracy within a user-defined range of this maximum accuracy. Using the max_depth corresponding to the latter reduces the max_depth of the tree, thus making it more interpretable.   
![Alt Text](accuracy_plot.jpg)  
The final visualization of the tree model was facilitated using the tree.plot_tree function:   
![Alt Text](song_tree.jpg)  
With a test accuracy of 95.73%, the class-wise accuracy is shown below:  
![Alt Text](class_accuracy.png)  
In performing GMM, the features were standardized by centering them around the mean and scaling to unit variance via StandardScaler(). Thus, the threshold values depicted below correspond to ~N(0,1). These values were subsequently converted back to their original scale for the following summary:  

**Group 0**:  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness = 0, 3.5 < Time signature <= 4.5  
**Group 1**:  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness = 0, Time signature ≤ 3.5, Speechiness ≤ 0.05  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness > 0, Time signature ≤ 3.5, Tempo > 185  
**Group 2**:
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness > 0, Time signature ≤ 3.5, Tempo ≤ 185  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness > 0, Time signature > 3.5, Liveness > 0.83  
**Group 3**:  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness = 0, Time signature ≤ 3.5, Speechiness > 0.05  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness = 0, Time signature > 4.5  
**Group 4**:  
&nbsp;&nbsp;&nbsp;&nbsp;Instrumentalness > 0, Time signature > 3.5, Liveness ≤ 0.83  


Part 2: Song Recommendations
------
The next idea was to create a summary of the lyrics to get at the larger abstract concepts being conveyed. First, [facebook/vart-large-cnn](https://huggingface.co/facebook/bart-large-cnn), a BART model trained on CNN Daily Mail to summarize text, was used. The lyrics for each song was pulled from the Genius API, and the BART model was used to summarize the lyrics and add to the song description. Where the BART model was trained on news articles, it did a poor job of extracting the abstract meaning of the lyrics, and sometimes converted it into a news blurb:

“Eyes Without a Face” by Billy Idol:  
  > *Billy idol's new album is out now and is out of print. The album is a collection of songs dedicated to his ex-girlfriend. Billy idol is currently on tour and will be performing in Las Vegas.*  

Prompting ChatGPT to summarize the lyrics results in:

“Eyes Without a Face” by Billy Idol:  
  > *The song "Eyes Without a Face" talks about feelings of hopelessness, disillusionment, and betrayal. The lyrics express sadness and anger at being lied to and deceived, and a sense of loss and despair over a past love. The chorus, "les yeux sans visage" translates to "eyes without a face," symbolizing a lack of humanity and grace. The song also contains references to escapism, criminal behavior, and a sense of alienation.*

Although not perfect and prone to reiteration, these responses are clearly preferred. Thus, the new string representation for a song was created using a combination of the ChatGPT API requests and the audio features from Spotify to fit the string representation with the final three formats:

1. **Audio Feature Description: "description"**    
Format: The song “*song name*” is in the “*genre*” genre. It is a “*audio features*”  
Example: *The song Anti-Hero is in the pop genre. It is a moderate energy, moderately danceable, low instrumental, low acousticness song.*

2. **ChatGPT Description: "new description"**    
Format: "*ChatGPT summary*"  
Example: *The song describes the singer's struggles with depression, self-awareness, and feeling like they are always the problem in their relationships. The lyrics also touch on themes of narcissism and self-destructive behavior. The chorus emphasizes the feeling of always being the problem and never being able to see oneself clearly.*

3. **Combined Description: "combined description"**    
Format: The song “*song name*” is in the “*genre*” genre. It is a “*audio features*”. “*ChatGPT summary*”

These descriptions were then used with a pretrained sentence transformer model, [all-mpnet-base-v2](https://www.sbert.net/docs/pretrained_models.html), to encode all podcast descriptions and find the top three matching songs to each given podcast episode. 


Results:
======
Part 1: Grouping Songs
------

Part 2: Song Recommendations
------
The complete results can be found in the "podcasts_songs.csv" file [here](https://github.com/sirena-depue/Projects). A small snippet of the results are shown below to show the difference between each of the three prompts:  
![Alt Text](podcast_recs.png)

**Audio Feature Description ("description"):**
As shown above, the song title significantly influences recommendations, leading to a lot of variability. Where “The Daily” is a news podcast, the song recommendations seem promising for just based on the title. However, the song “Breaking News” discusses domestic violence and “Headlines” covers themes of money and success. Recommendations for shows like “It’s Not Only Football” suggest songs with titles referencing the night or weekend, while those for “Crime Junkie” lean towards title implying crime – a closer match to the podcast content, but still disregarding song audio features.

**ChatGPT Description ("new description"):** 
As shown above, while the ChatGPT description yields more promising results in terms of contextual content for some podcast episodes, they are not always consistent with the overall mood of the podcast. 

The songs “Pumped Up Kicks” and “Jeremy” discuss troubled youth committing violent acts in schools, which aligns well with the Daily Show Episode “A Guilt Verdict for a Mass Shooter’s Mother”. This song is frequently recommended among political and true crime podcasts. The song “They Own the Media” discusses the manipulation of media narratives by specific entities and is recommended frequently with political and conspiratorial podcasts:  
![Alt Text](song_counts.png)

All songs recommended for “Crime Junkie” contain descriptions of criminal activity, often related to drugs. Despite sharing similar themes, the mood evoked by true crime stories may not align with the mood created by these songs. 

The recommendations for “It’s Not only Football” discuss enjoying good company and participating in recreational activities. Except for “Get you Mad” by Eminem, all suggested songs for the Sanctified podcast are gospel/worship songs. 

Future Improvements:
======
The aim behind the ChatGPT responses was to pair podcasts and songs based on similar content. However, this did not always perform well, especially when some of podcast descriptions are overly broad or episode descriptions lack detail. Additionally, the current approach fails to consider the mood conveyed by both the songs and episodes. While using Spotify’s audio features (energy, danceability, etc.) is meant to address this, the results often prioritize song titles over these features. 

Additionally, there are podcast episodes where song recommendations are not obvious even when made by humans. For example, the episode titled “AMA #15: Fluoride Benefits/Risks & Vagus Nerve Stimulation” by the "Huberman Lab", presents a challenge in suggesting a fitting song. However, episodes exploring more universal themes (i.e. love, loss) tend to yield more suitable recommendations. Finally, the quality of song suggestions is subjective and can vary according to individual preferences and opinions.

Some future improvements to address these shortcomings include:
 -	Collect more song data for increased diversity.
 -	Incorporate user feedback to account for personal preferences.
 -  Switch from the “gpt-3.5-turbo-1106” model to the "gpt-4" model (more expensive!) when requesting ChatGPT for more detailed summaries.
 -	If provided with the transcript of a podcast episode, can compensate for short episode descriptions by using ChatGPT to generate a summary. 
 -	If provided with the transcript of a podcast episode, perform sentiment analysis to gauge the overall mood, aligning with the Spotify audio feature, valence.  

