Introduction to NLP 2022/2023

University of Warsaw

MA Cognitive Science

# NLP analysis:
Polish Rap Lyrics

![Shape1](RackMultipart20230727-1-egbr2z_html_237499165a11f2b9.gif)

By Sofija Krivokapić & Anna Sazonov

![](RackMultipart20230727-1-egbr2z_html_f4bddddeb93ab528.png)

# INTRODUCTION

Dear reader,

Welcome to our report for a research project under the name "NLP analysis: Polish Rap Music"! We are very excited to show you our project, methods we used, and all the exciting results we obtained.

## Motivation

Apart from this research being part of our course requirement for _Introduction to Natural Language processing_ class at University of Warsaw (incorporated in Master programme Cognitive Science), with this research we wanted to express our appreciation for Polish rap culture, and add our own contribution to it.

## Summary

In our project we decided to look into the characteristics of Polish rap lyrics by analyzing unique words and checking which artists use more multisyllable words. We also performed topic classification (LDA) to learn about the most salient themes present in Polish rap music.

1.
### Data Collection

Due to the lack of availability of corpora of Polish rap lyrics we created our own corpus through using a genius lyrics API and LyricsGenius library. This step was surprisingly challenging but we now know how to use available tools to scrape data off of web pages.

1.
### Data Analysis

In the data analysis part of our project we employed many basic NLP tools we learned about in class after adapting them to suit the Polish language better.

# DATA COLLECTION & ANALYSIS

## Gathering the data

Given the lack of already available resources and corpora to work with, we resorted to making our own database. This consisted of several steps.

### Obtaining Genius API

Genius ([https://genius.com/](https://genius.com/)) is a well-known website which hosts the world's biggest collection of song lyrics and in general, musical knowledge. So, we figured, If we wanted to obtain vast amounts of lyrics data and texts, this is where we should start.

First step consisted of obtaining Genius API (see how: [https://docs.genius.com/#/getting-started-h1](https://docs.genius.com/#/getting-started-h1)).

### Installing lyricsgenius library

**LyricsGenius** library provides a simple interface to the song, artis, and lyrics data stored on Genius.com. All this is easily accessible through using their public API (see the documentation for this library: [https://lyricsgenius.readthedocs.io/en/master/](https://lyricsgenius.readthedocs.io/en/master/)). For reference, see how this library works in the following chunk of code.

!pipinstalllyricsgenius



importlyricsgenius

fromlyricsgeniusimportGenius

#my\_token is a variable with genius API as a string

my\_token="BbI104BwqEgGemxyZCxkV4EAgdjmJG7JV6snrcSMfG43JN4DkRe4RI\_yRToc8\_kw"

genius=lyricsgenius.Genius(my\_token)

#.search\_songs() looks for all of the songs of a particular artist

songs\_schafter=genius.search\_songs('schafter')

#accessing a particular song

songs\_schafter["hits"][0]["result"]["id"]

#or a foor loop

foriinrange(6):

print(songs\_schafter["hits"][i]["result"]["title"])

### Gathering the names

This was a particularly tricky moment for us, because we weren't completely sure how to decide which artists to include and which ones not. So, we asked our good friend Chat GPT-3 to give us a list of Polish rappers and rap groups by decades. Since the list wasn't exhaustive, we decided to use our own knowledge and in fact, add all rappers we knew. So basically, in the end, we ended up with a list of 82 rappers, not sorted in any way particular. More or less popular, everyone was added.

importpandasaspd

rappers\_list=pd.read\_csv("rapper list - Sheet1 (1).csv")

rappers\_list=rappers\_list["name"]

#rappers\_list = list(rappers\_list)

rappers\_list=['Tede','Paktofonika','Kaliber 44','Fisz Emade','Molesta Ewenement','Wzgórze Ya-Pa 3','Kasta','Płomień 81','Hemp Gru','Eldo','Quebonafide','Paluch','JWP','Miuosh','Vienio','Bonus RPK','Łona i Webber','DonGURALesko','Sokół','Liroy','Kękę','Taco Hemingway','Otsochodzi','Bedoes','Tymek','Żabson','Young Multi','Białas','Solar','Kali','Kubańczyk','Kabe','Borixon','Malik Montana','PlanBe','Włodi','Rasmentalism','Wac Toja','Jan-rapowanie','Eldorado FM','B.R.O','Kuba Knap','Kabe','Buka','Vixen','Szpaku','Biały','Kafar Dixon 37','Shellerini','ZBUKU','Bonson','Gutek','Pikers','Szad','Młody Dzban','Leh','Mata','Syny','UNDADASEA','Zdechły Osa','Stereofonia','Jacuś','Kuki','ĆPAJ STAJL','Belmondawg','Holak','asthma','schafter','Lanek','Pelson','Young Leosia','ten typ mes','Avi','Stare Miasto','Kukon','Warszafski Deszcz','Peja','Łona i Webber','O.S.T.R.','Problem','Pezet']



Then we had to check if the name spellings were correct and whether the artist actually existed on Genius.com, so we made a function to check that. Out of two rappers which were not found, one was removed from the list, first of all - well, because he didn't exist either on Genius.com or IRL, and the second one was misspelled, so we corrected it.

defname\_correct(my\_list):

not\_found=[]

fornameinmy\_list:

artist=genius.search\_artist(name,max\_songs=1,sort='title')

ifartistisNone:

not\_found.append(name)

fornameinnot\_found:

print(f"Artistnotfound:{name}.")



Later, we also realized that some artists may exist on Genius but have none of their songs there, so we ran our list through the function again, this time throwing out five artists (Avi, JWP, Leh, Kasta, Kali) whose songs we weren't able to find.

### Making a dataset of rap songs

In order to perform the analyses we wished, we needed to construct a large .csv file with relevant information for each rapper on our list and for each of their songs. We wanted to have a dataframe with:

- Artist's name
- Song title
- Year of release
- Lyrics
- Tokenized lyrics

We decided to clean the lyrics of some unnecessary words already while downloading them into our database. We made a **regex** pattern which would remove headings and ads which were downloaded with the lyrics.

Then we also performed a simple tokenization with **nltk** tokenizer and saved the tokens into a separate column.

Due to missing information for some artists' songs, we needed to come up with two different functions for downloading their data. For some less popular artists, the functions in the **lyricsgenius** library did not have access to the year that the song was published in and for others this information was accessible. Hence, we ended up with one function that gets the correct year, and a second function that fills the year column with 0 to be manually filled later. (The function below is a combination of the two functions).

importre

fromnltk.tokenizeimportRegexpTokenizer

importpandasaspd

tokenizer=RegexpTokenizer(r'\w+')

pattern = r'(Intro|\d+\s+Contributors[\w+\s]\*\w+|1Contributor|YoumightalsolikeEmbed|Refren|Hook|Lyrics|\d+Embed|Zwrotka(?:\d+)?)'

def get\_everything(artist\_name):

result = genius.search\_artist(artist\_name, max\_songs = 30, sort = 'popularity')

list\_of\_dictionaries=list()

forsonginresult.songs:

cleaned\_lyrics=re.sub(pattern,"",song.lyrics)

tokenized\_lyrics=tokenizer.tokenize(cleaned\_lyrics)

ifgenius.search\_song(title=song.title)isnotNone:

list\_of\_dictionaries.append({

"artist name":artist\_name,

"title":song.title,

"year":genius.search\_song(title=song.title).\_\_dict\_\_['\_body']['release\_date\_components']['year'],

"clean lyrics":cleaned\_lyrics,

"tokens":tokenized\_lyrics})

else:

list\_of\_dictionaries.append({

"artist name":artist\_name,

"title":song.title,

"year":0,

"clean lyrics":cleaned\_lyrics,

"tokens":tokenized\_lyrics})

returnpd.DataFrame.from\_dict(list\_of\_dictionaries)

defmake\_csv(artist\_list):

result=pd.DataFrame()

forartistinartist\_list:

artist\_df=get\_everything(artist)

result=pd.concat([result,artist\_df])

result.to\_csv(f"{artist}.csv",index=False)

returnresult



One may wonder why we did not create just one function to do both options like the one above. That was due to timeout issues. That is also why we downloaded the data artist by artist with a limit of 30 songs per artist. Otherwise the operation kept timing out and progress was lost.

So, after having gone through our list of rap artists and having a separate .csv for each artist, we combined those into one big dataframe.

importpandasaspd

dataset=pd.DataFrame()

forartistinrappers\_list:

single\_df=pd.read\_csv(f"{artist}.csv")

dataset=pd.concat([dataset,single\_df])

dataset.to\_csv("Rappers\_collected\_10\_May.csv",index=False,encoding="UTF-8")



After a while we discovered that our dataset's token column was a string so we converted it into a list.

fornumberinrange(len(loaded\_dataset)):

loaded\_dataset['tokens'][number]=eval(loaded\_dataset['tokens'][number])



### Removing stopwords

Since NLTK stopwords are not supported for Polish, we found a list of Polish stopwords on [github](https://github.com/bieli/stopwords/blob/master/polish.stopwords.txt).

withopen("polish.stopwords.txt","r",encoding='utf-8')asfile:

data=file.read()

data\_into\_list=data.split("\n")



As we didn't remove the stopwords before making the dataset, we just went through the column of tokens in our dataset removing those words which were stopwords and did not carry much useful information for our analysis.

fornumberinrange(len(loaded\_dataset)):

forwordinloaded\_dataset['tokens'][number]:

ifword.lower()indata\_into\_list:

loaded\_dataset['tokens'][number].remove(word)

Our analyses of unique words and multisyllable words were done on a dataset with removed stopwords.

### Stemming and lemmatization

For the purpose of LDA (Latent Dirichlet Allocation) which we had planned to perform on our dataset, we had to lemmatize our tokens. LDA is a way of classifying themes present in a text. Before performing this analysis the tokens should be lemmatized, stemmed and short words should be removed (in an [example](https://towardsdatascience.com/topic-modeling-and-latent-dirichlet-allocation-in-python-9bf156893c24) of performing LDA on a base of English newspaper articles, len(token) \< 3 were removed before analysis).

We used SpaCy for lemmatization.

!pipinstallspacy

!python-mspacydownloadpl\_core\_news\_lg

importspacy

# Load the Polish language model

nlp=spacy.load('pl\_core\_news\_lg')



In the chunk of code below you can see how we lemmatized the tokens column of our dataset.

fornumberinrange(len(dataset\_without\_stopwords)):

lemmatized\_words=[]

# Lemmatize each word in the 'tokens' column of the current row

forwordindataset\_without\_stopwords.loc[number,'tokens']:

doc=nlp(word)

lemmatized\_words.append(doc[0].lemma\_)

# Replace the 'tokens' column with the lemmatized words

dataset\_without\_stopwords.at[number,'tokens']=lemmatized\_words

# Print the first 10 lemmatized words in each row's 'tokens' column

forrowindataset\_without\_stopwords['tokens']:

print(row[:10])



### Extra cleaning

After performing a preliminary analysis of our data we discovered that it still needs some cleaning due to the presence of short, not very meaningful words, kind of like stopwords, which we didn't catch during our data cleaning stage.

Turns out that we did not remove stopwords carefully enough and performed our preliminary analysis with them in the dataset. We also noticed that many Polish past forms of verbs were lemmatized into two words. For example, 'chciałbym' was lemmatized into 'chcieć być', or 'wiedziałabym' into 'wiedzieć być'.

importre

# regex pattern

pattern=r'(\b\w+)\sbyć\b'

fornumberinrange(len(dataset\_stemmed)):

cleaned\_stems=[]

forstringindataset\_stemmed['tokens'][number]:

clean\_string=re.sub(pattern,r'\1',string)

cleaned\_stems.append(clean\_string)

dataset\_stemmed.at[number,'tokens']=cleaned\_stems

As you can see in the chunk of code above, this issue was solved with the help of regex.

## Analyzing the data

After we created our dataset we could begin its analysis. Rap lyrics are often praised for being original if they contain distinctive or long words which are more difficult to create rhymes with. In order to learn more about Polish rap artists' lyric writing habits we analyzed how many unique words an artist uses, the average number of syllables in the words they use and how many of the words in their songs are multisyllabic.

### Unique words per artist

In order to ascertain which Polish rapper uses the most unique words we counted how many unique words an artist has in their lyrics.

We encountered a problem of having varying numbers of songs per artist - with some having 30 (maximum number of songs per artist in our database) and others having just one or two. Which is why we decided to exclude artists who had less than 10 songs (Biały, Ziomcy, Stare Miasto, Gutek, Stereofonia) from this analysis.

importpandasaspd

per\_artist\_unique=pd.DataFrame(columns=['Rapper','Count'])# Create an empty DataFrame with specified column names

forartistinsongs\_per\_artist.index:

exclude=list()

num\_songs=songs\_per\_artist[artist]

ifnum\_songs\<10:

exclude.append(artist)

fornumberinrange(len(lyrics\_per\_artist['artist name'])):

name\_rapper=lyrics\_per\_artist['artist name'][number]

ifname\_rapper==artist:

total\_words=len(lyrics\_per\_artist['tokens'][number])/num\_songs

per\_artist=len(set(lyrics\_per\_artist['tokens'][number]))/num\_songs

percentage=(per\_artist/total\_words)\*100

data={'Rapper':name\_rapper,'Count':per\_artist,'Total\_words':total\_words,'unique by total':percentage}# Create a dictionary with 'Rapper' and 'Count' keys

per\_artist\_unique=per\_artist\_unique.append(data,ignore\_index=True)

else:

pass

print(exclude)

per\_artist\_unique.sort\_values(by=['Count'],axis=0,ascending=False)



The artists who scored highest in terms of percentage of unique words per song were **Szad** , **Pelson** , **Młody Dzban** , **Włodi** and **ten typ mes**.

![](RackMultipart20230727-1-egbr2z_html_2bd7212027cb3877.png)

On the other hand, those with the lowest percentages of unique words were: **KaBe, asthma, Wzgórze Ya-Pa 3, Young Mult** i and **Tymek**.

![](RackMultipart20230727-1-egbr2z_html_1d1e883696eba086.png)

In the plot below you can see the relationship between unique words per song and words per song for all artists. In our jupyter notebook there is an interactive version of this plot where one can see which bubble belongs to which artist. The sizes of the bubbles represent the percentages of unique words an artist uses.

![](RackMultipart20230727-1-egbr2z_html_79891d99843e76aa.png)

### Top 10 words per artist

In order to see which rappers used which words most we counted word occurrences among their lyrics on lemmatized words with stopwords removed. We believe that this allows us to glimpse into the themes each artist sings about most often.

fromcollectionsimportCounter

defmost\_used\_words(df,artist\_list):

common\_words\_by\_artist=list()

forartistinartist\_list:

artist\_df=df[df['artist name']==artist]

all\_tokens=[tokenfortokens\_listinartist\_df['tokens']fortokenintokens\_list]

fortokeninall\_tokens:

iflen(token)\<4:

# we chose to exclude words shorter than 4 because they were not very informative

all\_tokens.remove(token)

else:

pass

word\_counts=Counter(all\_tokens)

most\_common\_words=word\_counts.most\_common(10)

forword,countinmost\_common\_words:

one\_artist={

'artist':artist,

'word':word,

'count':count

}

common\_words\_by\_artist.append(one\_artist)

returncommon\_words\_by\_artist



#### Top 40 words among our artists' top 10s

Since it would be difficult to illustrate each of our artist's top 10 words, we made a word cloud with the **top 40 words among our artists' top 10s**.

![](RackMultipart20230727-1-egbr2z_html_64f45c5ee4e1e60c.png)

### Mean number of syllables per artist

In order to check the average length of words used by an artist in their lyrics we created a function which counts the number of syllables for each word in a list and then extracts the mean.

importpyphen

# Load the Polish hyphenation dictionary

dic=pyphen.Pyphen(lang='pl\_PL')

defcalculate\_mean\_syllables(list\_1):

syllable\_counts=[dic.inserted(word).count('-')+1forwordinlist\_1]

returnnp.mean(syllable\_counts)

# Create an empty DataFrame with specified column names

per\_artist\_syllables=pd.DataFrame()

fornumberinrange(len(lyrics\_per\_artist['tokens'])):

syllables=calculate\_mean\_syllables(lyrics\_per\_artist['tokens'][number])

rapper\_name=lyrics\_per\_artist['artist name'][number]

data=pd.DataFrame({'Rapper':rapper\_name,'Mean':syllables},index=[0])

per\_artist\_syllables=pd.concat([per\_artist\_syllables,data])

# List the rappers by mean syllable number descending

per\_artist\_syllables.sort\_values(by=['Mean'],ascending=False)

The artists who scored the highest mean syllables per word were Kuki, Bonus RPK, Molesta Ewenement, Paktofonika, and Peja.

![](RackMultipart20230727-1-egbr2z_html_ec1c8e0d8132a111.png)

### Percentage of multisyllabic words per artist.

We also looked at how many percent of an artist's text consists of words longer than one syllable.

importpyphen

defcalculate\_syllables(list\_2):

syllable\_counts=[dic.inserted(word).count('-')+1forwordinlist\_2]

returnsyllable\_counts

# Create an empty DataFrame with specified column names

per\_artist\_syllables\_percentage=pd.DataFrame()

fornumberinrange(len(lyrics\_per\_artist['tokens'])):

syllables\_2=calculate\_syllables(lyrics\_per\_artist['tokens'][number])

rapper\_name=lyrics\_per\_artist['artist name'][number]

percentage=sum(1forsyllableinsyllables\_2ifsyllable\>1)/len(syllables\_2)\*100

data1=pd.DataFrame({'Rapper':[rapper\_name],'Syll':[syllables\_2],'% or multisyllabic':[percentage]})

per\_artist\_syllables\_percentage=pd.concat([per\_artist\_syllables\_percentage,data1])

per\_artist\_syllables\_percentage

The mean percentage of multisyllabic words among our dataset was 68.3%.

The artists with the highest percentage of multisyllabic words were Kuki, Bonus RPK, Pelson, Stereofonia and Taco Hemingway. The artists with the lowest percentages of multisyllabic words were asthma, Young Multi, Kabe, Schafter and Bonson.

### Latent Dirichlet Analysis (LDA).

LDA is a way of performing unsupervised theme discovery in a dataset made up of words. It allows us to learn about themes present in a dataset (what we are most looking forward to doing) and additionally it can later serve as a way of classifying a song into one of the topics found in the data.

The gensim library is needed in order to perform LDA. We also imported the simple\_preprocess module.

!pipinstallgensim

importgensim

fromgensim.utilsimportsimple\_preprocess



Then we created a dictionary from our dataset\_stemmed tokens column containing the number of times a word appears in the training set.

dictionary=gensim.corpora.Dictionary(dataset\_stemmed['tokens'])



Next we filtered out tokens which appeared in less than 5 songs or more than half of the songs. After the two steps, we kept 100 000 of the most frequent tokens.

dictionary.filter\_extremes(no\_below=5,no\_above=0.5,keep\_n=100000)



For each document - in our case for each song - we created a dictionary reporting how many words and how many times those words appear - a bag of words. Then we inspected how this operation worked out.

bow\_corpus=[dictionary.doc2bow(doc)fordocindataset\_stemmed['tokens']]

#Preview how this worked

bow\_doc\_4310=bow\_corpus[10]

foriinrange(len(bow\_doc\_4310)):

print("Word {} (\"{}\") appears {} time.".format(bow\_doc\_4310[i][0],

dictionary[bow\_doc\_4310[i][0]],bow\_doc\_4310[i][1]))

The code above resulted in the output pictured below. We were happy with the result.

![](RackMultipart20230727-1-egbr2z_html_7abed9e7a7c03a73.png)

We decided to run LDA in two different ways, using Bag of Words and TF-IDF. In both approaches we trained our LDA model using gensim.models.LdaMulticore.

lda\_model=gensim.models.LdaMulticore(bow\_corpus,num\_topics=3,id2word=dictionary,passes=2,workers=2)



At first we tried to run it by setting the number of topics to 5 but due to the fact that the topics seemed to all contain very similar words, we decided to cut down the number of topics we were looking for to three.

foridx,topicinlda\_model.print\_topics(-1):

print('Topic: {} \nWords: {}'.format(idx,topic))



![](RackMultipart20230727-1-egbr2z_html_5bc1493004a6bf93.png)

In order to get a grasp of which of the topics found in our data could refer to what we generated simple word clouds based on the order of the words assigned to each of the topics:

![](RackMultipart20230727-1-egbr2z_html_4161b3699ac69966.png)It seems that topic 1 has the unique word 'dobry' - good, and focuses on possibility and life

![](RackMultipart20230727-1-egbr2z_html_4161b3699ac69966.png) ![](RackMultipart20230727-1-egbr2z_html_4161b3699ac69966.png)

Topic 2 can be characterized by the presence of the word 'miasto' which means city and again focuses on life but also has words such as 'swój' (own, as in one's own), 'człowiek' (human, person) and 'widzieć' (to see). Topic 3 focuses on time, life, doing and giving. All of the topics have 'kurwa' in them, the most popular of Polish vulgarities.

We also ran LDA using TF-IDF. For that, we created a TF-IDF model object using models and transformed our corpus to have tf-idf scores.

fromgensimimportcorpora,models

tfidf=models.TfidfModel(bow\_corpus)

corpus\_tfidf=tfidf[bow\_corpus]

# Previewing tf-idf scores for our first song

frompprintimportpprint

fordocincorpus\_tfidf:

pprint(doc)

break



We trained the TF-IDF LDA on the tf-idf corpus, and again had to revise the number of topics down from 5 to three.

lda\_model\_tfidf=gensim.models.LdaMulticore(corpus\_tfidf,num\_topics=3,id2word=dictionary,passes=2,workers=4)



foridx,topicinlda\_model\_tfidf.print\_topics(-1):

print('Topic: {} Word: {}'.format(idx,topic))

 ![](RackMultipart20230727-1-egbr2z_html_32109500e806e44d.png)

![](RackMultipart20230727-1-egbr2z_html_e527d1548d91f709.png)

In the TF-IDF model we can see some more variety in the topics making it a bit easier to come up with an interpretation. Topic 1 seems to center around life, the world, the city and the day. Topic 2 seems to center around life and people, rap and expressing oneself ('powiedzieć' - to say and 'ej'). Topic 3 has the word 'dom' in it which means home or house, as well as the word 'młody' meaning young and 'czuć' - to feel. This could be interpreted as a topic of home and youth and feelings.

Interestingly enough, before handing our project in, we also discovered that running the tf-idf model training with a larger number of passes produced the most out of the ordinary topics.

![](RackMultipart20230727-1-egbr2z_html_712fbb2ddaaef238.png)

**Evaluating the models.**

With LDA, we could not only learn about hidden themes in our data but also see how a document from outside of our corpus would be classified by our models.

For that we ran a quick lemmatization on a song by Młody Leh (Miasto nocą) whom we were unable to include in our original analysis. The song lyrics, turned into a bow, figure in the code below under 'dictionary\_words'.

forindex,scoreinsorted(lda\_model[dictionary\_words],key=lambdatup:-1\*tup[1]):

print("\nScore: {}\t \nTopic: {}".format(score,lda\_model.print\_topic(index,10)))

 ![](RackMultipart20230727-1-egbr2z_html_7a90a5d9f499afa6.png)

We expected the song to be classified in the topic connected to miasto - city due to its title but it seems that our model decided that it is more about life, people and possibility (topic number one in the bag of words based model).

forindex,scoreinsorted(lda\_model\_tfidf[dictionary\_words],key=lambdatup:-1\*tup[1]):

print("\nScore: {}\t \nTopic: {}".format(score,lda\_model\_tfidf.print\_topic(index,10)))

 ![](RackMultipart20230727-1-egbr2z_html_5a5384d957e561af.png)

The tf-idf model classified it into its second topic which we defined as the theme of life, people, rap and self-expression.

# CONCLUSION

This project has definitely taught us both a lot. We are actually hoping to develop it further in the coming month. We would like to add a section of analysis based on decades - which we intended to do but ended up being short on time. Another area worth developing is our database, it is satisfactory but could be filled with missing songs and artists whose lyrics we were not able to scrape from Lyrics genius.
