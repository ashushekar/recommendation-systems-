# Demographic Filtering

They offer generalized recommendations to every user, based on movie popularity and/or genre. The System recommends the same movies 
to users with similar demographic features. Since each user is different , this approach is considered to be too simple. The basic 
idea behind this system is that movies that are more popular and critically acclaimed will have a higher probability of being liked 
by the average audience.

### How to proceed with Demographic Filtering

Before getting started with this -

1. we need a metric to score or rate movie
2. Calculate the score for every movie
3. Sort the scores and recommend the best rated movie to the users.

We can use the average ratings of the movie as the score but using this won't be fair enough since a movie with 8.9 average rating 
and only 3 votes cannot be considered better than the movie with 7.8 as as average rating but 40 votes. 

Hence we need to calculate weighted average ratings of each movie:

```sh
Weighted Rating (WR) = (R*(v/(v+m))) + (C*(m/(v+m)))
```
where
    v is the number of votes for the movie
    m is the minimum votes required to be listed in the chart;
    R is the average rating of the movie; And
    C is the mean vote across the whole report

We already have v(vote_count) and R (vote_average) and C can be calculated as

```sh
C= df2['vote_average'].mean()
```

So, the mean rating for all the movies is approx C on a scale of 10. The next step is to determine an appropriate value for m, the 
minimum votes required to be listed in the chart. We will use 90th percentile as our cutoff. In other words, for a movie to feature 
in the charts, it must have more votes than at least 90% of the movies in the list.

```sh
m= df2['vote_count'].quantile(0.9)
```

Filtering out movies which qualifies the chart:

```sh
q_movies = df2.copy().loc[df2['vote_count'] >= m] 
``` 

Now apply weightedrating() function and define a new feature score, of which we'll calculate the value by applying this function to 
our DataFrame of qualified movies.

```sh
def weighted_rating(x, m=m, C=C):
    v = x['vote_count']
    R = x['vote_average']
    # Calculation based on the IMDB formula
    return (v/(v+m) * R) + (m/(m+v) * C)
    
q_movies['score'] = q_movies.apply(weighted_rating, axis=1)
```

Finally, let's sort the DataFrame based on the score feature and output the title, vote count, vote average and weighted rating or 
score of the top 10 movies.

```sh
#Sort movies based on score calculated above
q_movies = q_movies.sort_values('score', ascending=False)

#Print the top 15 movies
q_movies[['title', 'vote_count', 'vote_average', 'score']].head(10)
```

Output is:
```sh
                                            title_y  ...     score
1881                       The Shawshank Redemption  ...  8.059258
662                                      Fight Club  ...  7.939256
65                                  The Dark Knight  ...  7.920020
3232                                   Pulp Fiction  ...  7.904645
96                                        Inception  ...  7.863239
3337                                  The Godfather  ...  7.851236
95                                     Interstellar  ...  7.809479
809                                    Forrest Gump  ...  7.803188
329   The Lord of the Rings: The Return of the King  ...  7.727243
1990                        The Empire Strikes Back  ...  7.697884 
```

Under the Trending Now tab of these systems we find movies that are very popular and they can just be obtained by sorting the 
dataset by the popularity column.