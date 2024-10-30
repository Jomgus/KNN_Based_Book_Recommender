
---

# KNN-Based Book Recommender System Using Goodreads Data

This project aims to build a KNN-based recommender system that suggests books similar to a user-selected title, leveraging features from the Goodreads dataset.
<p align="center">
  <img src="https://files.oaiusercontent.com/file-2X8ndc9hGaLXHmufCa6S9cbW?se=2024-10-30T08%3A38%3A52Z&sp=r&sv=2024-08-04&sr=b&rscc=max-age%3D604800%2C%20immutable%2C%20private&rscd=attachment%3B%20filename%3Db1644b66-c247-497a-aef3-57bbc417d495.webp&sig=yfWx6tqGcVgPS1YU7jkTMgQJivmQMY0jCcRsxH0fI8Q%3D" width="400" alt="Library Image">
</p>

## Project Overview

- **Objective**: Design a recommendation system that identifies and suggests books similar to a given title based on selected features.
- **Dataset**: Goodreads book data including title, author, average rating, language, and other relevant features.
- **Recommendation Approach**: K-Nearest Neighbors (KNN) with a focus on similarity-based book recommendations.

## Key Stages

### 1. Data Sourcing and Preparation
   - **Data Collection**: [Kaggle’s Goodreads dataset](https://www.kaggle.com/datasets/jealousleopard/goodreadsbooks)  offers a comprehensive list of books spanning the entire Goodreads website. Book datasets serve as an excellent basis for analysis and modeling. Every book contains within itself a variety of features that can be explored and compared against others, making this kind of project an interesting opportunity to experiment with those features and generate new insight.

     column descriptions: 
1. bookID = contains the unique ID for each book/series
2. title = contains the titles of the books
3. authors = contains the author of the particular book
4. average_rating = the average rating of the books, as decided by the users
5. ISBN ISBN(10) = number, tells the information about a book - such as edition and publisher
6. ISBN 13 = the new format for ISBN, implemented in 2007. 13 digits
7. language_code = tells the language for the books
8. Num_pages = contains the number of pages for the book
9. Ratings_count = contains the number of ratings given for the book
10. text_reviews_count = has the count of reviews left by users
11. publication_date = date of publication
12. publisher = name of the publisher
   - **Preprocessing**: The data was not perfectly clean at the outset. Bad lines in the dataset had to be dealt with, column titles had to be stripped of whitespaces, and author names had to be standardized.

### 2. Feature Engineering
   - **Feature Selection**: Initially thought to only include the numerical features (e.g., `average_rating`, `num_pages`, `ratings_count`, etc.), then expanded the feature set to include languages.
   - **Encoding**: One-hot encoding was chosen to prepare language_code for the KNN model. The language encoded dataframe was then concatonated with the numerical features dataframe and turned into a feature matrix to load into the model.
   - **Scaling**: Data normalized with Scikit-Learn's min-max scaler.  

### 3. Model Selection and Tuning
   - **Algorithm Choice**: K-Nearest Neighbors (KNN) for content-based recommendations.
   - **Distance Metric**: Use cosine similarity to measure the closeness between books.
   - **Parameter Tuning**:
     
     ![image](https://i.imgur.com/PltwZ1U.png)
     
     Using the traditional elbow method, we can see a uniform distribution with no clear 'elbow' to determine optimal K. The absence of a sharp elbow here is actually expected for a few reasons.
     
1. by now, features have been well standardized
2. one hot encoding languages has created a sparse high-dimensional space
3. book similarities tend to be naturally continuous rather than clustered

    Fortunately, we can instead plot a rate of change, or derivative plot. This instead shows us how quickly the distances between points are changing - essentially measuring the 'speed' at which the first plot is rising. Instead of looking for an elbow here, we are looking for the ROC stabilizing. Our plot shows that the rate of change stabilizes around K=8,           suggesting it is a good choice for K
     
  

### 4. Model Implementation
   - **Similarity Matching**: The fuzzy matching library fuzzywuzzy was used to improve title matching, allowing the system to handle minor variations or errors in user-inputted book titles. This increases the system’s flexibility and usability, matching input titles closely to those in the dataset.
   - **Recommendation Function**: A custom function was designed to retrieve book recommendations based on a user-provided title.
     - Input: User-selected book title.
     - Output: List of similar book titles based on proximity in the feature space.
     - Thresholds: A matching threshold was set in fuzzy matching to ensure reasonable confidence in the title match before making recommendations. This prevents irrelevant suggestions when no close match is found.

### 5. Evaluation and Testing
   - **Quality Assessment**: The system was evaluated manually by inputting a variety of popular book titles and examining the recommendations. Recommendations were judged on similarity in genre, theme, or author to ensure relevance.
   - **Error Analysis**: The recommendations from the base model are generally good, however they sometimes produce recommendations that vary widly in genre and theme. I expect this recommendation system would be more consistent if there were a genre feature that could then be encoded to more accurately match recommendations.

## Results and Insights

- **Example Recommendations**:
  ```
  recommend_books("no country for old men", num_recommendations=5)
  ```
  ```
  Match result: ('no country for old men', 100, 12497)
  Book indices found: [12497]
  Recommendations for 'no country for old men':
  1. Murder on the Orient Express (Hercule Poirot  #10)
  2. My Life in France
  3. Shutter Island
  4. Magic Bites (Kate Daniels  #1)
  5. In the Heart of the Sea: The Tragedy of the Whaleship Essex
  ```
**Strong Matches** : In the Heart of the Sea, Shutter Island, and Murder on the Orient Express are closely aligned with the themes of suspense and moral complexity.

**Moderate Matches** : Magic Bites has some thematic overlaps but may appeal more to those interested in fantasy settings.

**Weaker Match** : My Life in France diverges significantly in tone and genre, making it a less appropriate recommendation.


- **Observations**: There is plenty of genre overlap in the recommendations which inspires confidence in the model. Authors and series seem to be clustered very closely, as inputting something like harry potter will output other books in the harry potter series. This is also a good sign that things are working as intended, as author was not considered in the feature set.   

## Visualizations

![image](https://i.imgur.com/LExwIHX.png)

Twilight has just over 4 million reviews. Something interesting here is that twilight is a series, having such a huge margin between itself and other books in the top 5, the discrepancy between twilight book 1 and any other book in the series must be huge. This tells us there are many people who did not read past the first book. 

![image](https://i.imgur.com/eo2cc3x.png)

From this distribution we can tell that most ratings are clustered around 4 and anything near 5 or below 3 is very rare. Some observations: 

1. self-selection bias where people who finish books are more likely to
  - have enjoyed the book enough to finish
  - take the time to rate it
  - rate it favorably based on their completion 
2. the very low frequency of ratings below 3 might indicate that people who don't enjoy books tend to:
- not finish them
- not rate them at all
- abandon them without leaving a low rating

The strong skew towards higher ratings (particularly around 4 stars) and the notable absence of lower ratings suggests that ratings may be biased towards readers who complete books, as those who abandon books they don't enjoy are less likely to provide ratings at all.

## Technologies Used

- **Programming Language**: Python
- **Libraries**: Scikit-Learn (for KNN and preprocessing), Pandas, Matplotlib, FuzzyWuzzy

## Conclusion and Next Steps

- **Summary of Findings**: The model provides relevant and effective recommendations based on input book titles, showing notable accuracy in matching book themes and tone. Certain limitations exist due to sparse and high-dimensional data, affecting some genre consistency.
- **Future Improvements**:
  - Add Genre Filtering: Incorporating a filter to prioritize or restrict recommendations by genre could further refine results.
  - Enhance Feature Set: Explore additional book attributes, such as year published or popular tags, for added depth.
  - Dimensionality Reduction: Applying techniques like PCA to simplify the feature space could improve runtime efficiency and potentially mitigate high-dimensional data issues.

---
