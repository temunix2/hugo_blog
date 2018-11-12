---
title: "Book2Playlists"
date: 2018-09-28T13:44:46-04:00
draft: true
hero_image: "hero.jpg"
description: "The inspiration behind this project comes from my passion for reading while listening to music. Creating custom playlists for each book would then allow me to have a better playlist that suits the book."
---
![](https://www.sfcv.org/sites/default/files/styles/reduced_size/public/u43270/books_on_music_header.jpg?itok=A-yAy4jl)


### Overview

The inspiration behind this project comes from my passion for reading while listening to music. I think at this point my friends are split 50/50 on people that listen to music while reading and people that need complete silence otherwise it just doesn't work for them. Me, personally I've always been a big fan of listening to music when I'm doing anything because it creates what I think is a personal soundtrack to my life. So when listening tot music while reading helps create a soundtrack to the book similar to how a movie has a soundtrack that helps the audience vibe with the emotions the characters are going through.

This project which I've decided to call Book2Playlist will do exactly that. Take a Book and using the text inside the book turn it into a playlist with songs that are similar to the text in the book. Using the concept of topic modeling and topic matching across two different corpuses I will attempt to connect book text to song lyric text. Finally I will then match the song lyrics topics back to more recent songs on Spotify leveraging Spotify's audio features. I will explain each step in detail below for further clarification.

### Data Collection & Topic Modeling

I collected ~3,000 books from Project Gutenberg, 57,000 song lyrics from Metro Lyrics as my main corpus to use for topic modeling. I began the topic modeling process by lemmatizing my corpus. Lemmatization is the process of taking a word and bringing it down to its common base. Words like organize, organizes, and organizing all are different forms of the same base word. Lemmatization helps us reduce the number of words in our corpus allowing us to do a better job of topic modeling in the next steps.

Next, I used TF-IDF to tokenize my corpus from text to a frequency that I can input to help establish weights. The two options for this are Count Vectorizer or TF-IDF. Count Vectorizer is the simpler version and takes a straight count of the words in the entire corpus, while TF-IDF takes into consideration that some common words might appear too frequently and this would remove the importance of rarer words. TF-IDF does this by taking the count of the term (Term Frequency) in a document and multiplying by the total number of documents by the number of documents the term appears in. This second part is what offsets the weighting to account for common words.

- ![](https://cdn-images-1.medium.com/max/1600/1*8XpbsR4HdAHBXy5MgpIyug.png)

For the corpus of books, I used NMF to perform the topic modeling. NMF is a method for topic modeling that takes a term-document matrix and generates a set of topics that represent weighted sets of co-occurring terms. The discovered topics form a basis that provides an efficient representation of the original documents. With any NLP topic modeling there it is definitely as much a art as it is a science. Going through the process I repeatedly ran NMF again and again while adjusting and tuning my TF-IDF in order to find better topics. Some of the key parameters to adjust are min_df and max_df. Min_df is a parameter in TF-IDF that is used for removing terms that appear too infrequently, meaning if min_df is set to 0.5 it ignores terms that appear in less than 50% of the documents. While max_df is used for removing terms that appear too frequently. For example if max_df is set to 0.5 TF-IDF will ignore terms that appear in more than 50% of the documents. Another part of tuning the model included updating my stop words. Stop words are words that you see appear in your topics but explicitly don't want. The stop words you have selected will then be removed from the corpus before you begin. An example of some of my stop words were ('ooh','ye','tis','ow',...).

## Topic Matching Using Google Word2Vec

Word2Vec used to match topics using google word2vec n_similarity function. This returned a value of how similar 2 sets of topics were. As you can see in the example below, the Google Word2Vec does a good job at matching topics from 2 separate datasets.

Initially had 20 topics and began matching using the google word2vec but found that majority of the book topics were being matched to around 30% of the song topics. This led to low variability and not the purpose of the project. Increase topic count to 100 topics and found the matching to increase to around 70% meaning the book topics matched to a more diverse set of song topics. The project could have ended here with the book topics being matched to song topics and then serving the user those songs from my database of songs from Metro Lyrics but I wanted to have more recent songs as well as instrumental songs. So, I decided to get more recent songs from Spotify using their public api.

![Topic Matching](/img/Topic_match.png)

## Gathering more songs from Spotify Public Playlists

Using the public API and leveraging the great library [Spotipy](https://github.com/plamere/spotipy), I was able to get approximately 111,000 songs from Spotify. If you are interested in the code, I used to scraped the songs you can take a look at my code on github. Spotify has public playlists that they create and by entering their username I was able to go through every single one of their playlists and get all the songs. Another great part about Spotify's public api is the audio features. Spotify's API provides you with the audio features of a song. The available features range from speechiness, loudness, and even danceability. If you are curious to the full set of audio feature, the [documentation](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/) provides a detailed explanation of each feature.

So, in order for me to match my song lyric topics to Spotify's more recent songs I needed a link. The audio features served as that link for me. Remember I have a 100 topics create from the song lyrics each with several songs. Now each of these songs have audio features as well if they exist on the Spotify library and after finding the audio features of each of the songs in the topics, I averaged them to have a audio feature per topic. Now each of my 100 song topics have an average audio feature that I can use to find new similar songs.

## Similarity Matching

In order for me to find similar songs as my topic I leveraged cosine similarity matching. How cosine similarity works is that first you have the data mapped onto a feature space. The data in this case describing each song is the audio features. In the example below the feature space is mapped in two dimensions but it is the same as we increase the number of dimensions. Then by finding the cosine of the angle between the two vectors we can find how close they are in that feature space. We do this for all the data points and by returning the points that have the smallest cosine value from the point of interest we will find the most similar point (i.e. similar songs).

Manhattan Distance

Euclidean Distance

![](http://i0.wp.com/techinpink.com/wp-content/uploads/2017/07/cosine.png?resize=300%2C230)


In some case where multiple sections of books match up to the same song topics, I did not want the most similar song appearing across multiple playlists so I introduced some variability in my algorithm. I took the 20 closest songs by cosine similarity and then randomly selected a song so even if two people select the same book they would have their own customized playlist. I have included my flow chart which details my process in the image below.

![Flow Chart](/img/Book2playlist_flow.png)

## Conclusion

I created a flask app to demonstrate this concept and am currently working on deploying it so other users can play with it. An extra functionality that I added in was the option to select which distance measure you would like to select between Manhattan distance, Euclidean or Cosine. Another is the option to select instrumentals only. After speaking with some peers, I found that some people do not like listening to words while reading so I added the feature to listen to instrumental music only.

You can check out the demo below.

[![DEMO](/img/book2playlist_coverpic.png)](http://www.youtube.com/watch?v=xIzSUM5POFE)

Thanks for reading. Here's the link to my github repo for this [project](https://github.com/temunix2/Book2Playlist).
