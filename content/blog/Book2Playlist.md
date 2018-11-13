---
title: "Book2Playlists"
date: 2018-09-28T13:44:46-04:00
draft: false
hero_image: "wallpic1.jpeg"
description: "The inspiration behind this project comes from my passion for reading while listening to music. Creating custom playlists for each book would then allow me to have a better playlist that suits the book."
---
![](https://www.sfcv.org/sites/default/files/styles/reduced_size/public/u43270/books_on_music_header.jpg?itok=A-yAy4jl)


### Overview

The inspiration behind this project comes from my passion for reading while listening to music. I think there has been a trend where more people listen to music while reading or doing work but there are still people that need complete silence otherwise it just doesn't work for them. Me, personally I've always been a big fan of listening to music when I'm doing anything because it creates a personal soundtrack to my life. With that said listening to music while reading provides a soundtrack for the book and the music helps enhance the experience similar to a movie soundtrack.

Book2Playlist aims to create custom playlists for books in order to help enhance the reading experience. Using the text inside the book to turn it into a playlist with songs that are similar to the text in the book. Using the concept of topic modeling and topic matching across two different corpuses I will attempt to connect book text to song lyric text. Finally I will then match the song lyrics topics back to more recent songs on Spotify leveraging Spotify's audio features. I will explain each step in detail below for further clarification.

### Data Collection & Topic Modeling

I collected ~3,000 books from Project Gutenberg, 57,000 song lyrics from Metro Lyrics as my main corpus to use for topic modeling. I began the topic modeling process by lemmatizing my corpus. Lemmatization is the process of taking a word and bringing it down to its common base. Words like organize, organizes, and organizing all are different forms of the same base word. Lemmatization helps us reduce the number of words in our corpus allowing us to do a better job of topic modeling in the next steps.

Next, I used TF-IDF to tokenize my corpus from text to a frequency that I can input to help establish weights. The two most common options for tokenization are count vectorizer or TF-IDF. Count Vectorizer is the simpler version and takes a straight count of the words in the entire corpus, while TF-IDF takes into consideration that some common words might appear too frequently and this would remove the importance of rarer words. TF-IDF does this by taking the count of the term (Term Frequency) in a document and multiplying by the total number of documents by the number of documents the term appears in. This second part is what offsets the weighting to account for common words.

- ![](https://cdn-images-1.medium.com/max/1600/1*8XpbsR4HdAHBXy5MgpIyug.png)

For the corpus of books, I used NMF to perform the topic modeling. NMF is a method for topic modeling that takes a term-document matrix and generates a set of topics that represent weighted sets of co-occurring terms. The discovered topics form a basis that provides an efficient representation of the original documents. With any NLP topic modeling, it is definitely as much a art as it is a science. Going through the process I repeatedly ran NMF again and again while adjusting and tuning my TF-IDF in order to find better topics. Some of the key parameters to adjust are min_df and max_df. Min_df is a parameter in TF-IDF that is used for removing terms that appear too infrequently, meaning if min_df is set to 0.5 it ignores terms that appear in less than 50% of the documents. While max_df is used for removing terms that appear too frequently. For example if max_df is set to 0.5 TF-IDF will ignore terms that appear in more than 50% of the documents. Another part of tuning the model included updating my stop words. Stop words are words that you see appear in your topics but explicitly don't want. The stop words you have selected will then be removed from the corpus before you begin. An example of some of my stop words were ('ooh','ye','tis','ow',...).

## Topic Matching Using Google Word2Vec

Now with multiple topics for both the books corpus and the song lyrics corpus I need a method to match topics to create a connection. I was able to leverage Google word2vec model. The Google word2vec pre-trained model is one where Google trained on roughly 100 billion words from a Google News dataset that has word vectors for a vocabulary of 3 million words, and phrases. Using this pre-trained model and its n_similarity function, which returns a value of how similar 2 sets of topics were. As you can see in the image below, the Google Word2Vec does a good job at matching topics from 2 separate datasets.

Initially had 20 topics and began matching using the google word2vec but found that majority of the book topics were being matched to around 30% of the song topics. This led to low variability and obviously not the purpose of the project. Increasing the topic count to 100 topics allowed the matching to increase to around 70% meaning the book topics matched to a more diverse set of song topics. The project could have ended here with the book topics being matched to song topics and then serving the user those songs from my database of songs from Metro Lyrics but I wanted to add more recent songs as well as instrumental songs. So, I decided to get more recent songs from Spotify using their public api.

![Topic Matching](/img/Topic_match.png)

## Gathering more songs from Spotify Public Playlists

Using the public API and leveraging the great [Spotipy](https://github.com/plamere/spotipy) library, I was able to get approximately 111,000 songs from Spotify. If you are interested in the code, I used to scraped the songs you can take a look at my code on github. Spotify has public playlists that they create and by entering their username I was able to go through every single one of their playlists and get all the songs. Another great part about Spotify's public api is the audio features. Spotify's API provides you with the audio features of a song. The available features range from speechiness, loudness, and even danceability. If you are curious to the full set of audio feature, the [documentation](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/) provides a detailed explanation of each feature.

So, in order for me to match my song lyric topics to Spotify's more recent songs I needed a link. The audio features served as that link for me. Remember I have a 100 topics created from the song lyrics each topic with several songs. Now each of these songs have audio features as well if they exist on the Spotify library and after finding the audio features of each of the songs in the topics, I averaged them to have a audio features per topic. Now each of my 100 song topics has average audio features that I can use to find new similar songs.

## Similarity Matching

In order for me to find similar songs as my topic I leveraged cosine similarity matching. How cosine similarity works is that first you have the data mapped onto a feature space. The data in this case describing each song is the audio features. In the example below the feature space is mapped in two dimensions but it is the same as we increase the number of dimensions. Then by finding the cosine of the angle between the two vectors we can find how close they are in that feature space. We do this for all the data points and by returning the points that have the smallest cosine value from the point of interest we will find the most similar point (i.e. similar songs).

![](http://i0.wp.com/techinpink.com/wp-content/uploads/2017/07/cosine.png?resize=300%2C230)

Other distance metrics I included were Manhattan and Euclidean distance. Manhattan distance is the distance between two points measured along axes at right angles similar to city blocks in Manhattan. Euclidean distance is the distance metric most of us learn in geometry class which draws a straight line between two points to find the distance.

In some case where multiple sections of books match up to the same song topics, I did not want the most similar song appearing across multiple playlists so I introduced some variability in my algorithm. I took the 20 closest songs by cosine similarity and then randomly selected a song so even if two people select the same book they would have their own customized playlist. I have included my flow chart which details my process in the image below.

![Flow Chart](/img/Book2playlist_flow.png)

## Extra Flask features
To allow for variation in the playlists I have allowed the user to select which distance measure you would like to select between Manhattan distance, Euclidean or Cosine. Another feature is for the users who may not like listening to songs with words in them and prefer instrumentals. From the songs available from my Spotify scrape, I was able to filter on the audio features instrumentalness and speechiness leaving only instrumental or primarily instrumental songs.

## Conclusion

Using topic modeling across two different corpuses I was able to create links between them ultimately leading to customized playlists per book. Leveraging Google pre-trained word2vec model and Spotify's audio features were a key part to this entire project. I am working on deploying this web app so that other users can play with it as well as adding in more modern books as the project Gutenberg books tend to be a bit older. In the meantime, you can check out the demo of my web app below.

[![DEMO](/img/book2playlist_coverpic.png)](http://www.youtube.com/watch?v=xIzSUM5POFE)

Thanks for reading. Here's the link to my github repo for this [project](https://github.com/temunix2/Book2Playlist).
