# Yelp-Exploratory-Analysis
Exploratory Analysis and Azure database integration on Yelp Dataset.

original colab notebook: <br>[notebook 1](https://colab.research.google.com/drive/1pcwkSwUwhxXLokp3liKj2RI2K3das7AI)
                         <br>[notebook 2](https://colab.research.google.com/drive/1e9_F3NrT6JRBF1wJ9Z_VXBWiOWAirtNA#scrollTo=d9upUB2JJFQM)

![image](https://github.com/user-attachments/assets/afdf5480-69ae-41a4-adb2-c06dc0128109)


data used : [Yelp academic dataset](https://www.kaggle.com/datasets/yelp-dataset/yelp-dataset)


<h1>Objective</h1>
As an investor that is trying to open a restaurant in Illinois, location is a key factor in deciding the style which the restaurant should be. Therefore, we are looking to use SQL queries with yelp dataset and sqlalchemy python toolkit to locate the best location to open our restaurant.

<h1>Database Connection</h1>
<code># Install unixODBC
!apt-get update
!apt-get install -y unixodbc unixodbc-dev
  # Install pyodbc
!pip install pyodbc
  #Install sqlalchemy
!pip install sqlalchemy
  # Install ODBC Driver 17 for SQL Server
!curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
!curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list | tee /etc/apt/sources.list.d/mssql-release.list
!apt-get update
!ACCEPT_EULA=Y apt-get install -y msodbcsql17</code>

<br>**Connecting Azure SQL database**


<img width="629" alt="Screenshot 2025-06-04 at 10 09 31 PM" src="https://github.com/user-attachments/assets/5a1f0bbc-a421-46a3-90cd-abb1a9da6526" />


<h1>Analysis</h1>

<h2>Tables</h2>

<br>**Business**
<img width="1374" alt="Screenshot 2025-06-04 at 10 24 01 PM" src="https://github.com/user-attachments/assets/8471ea30-ccbe-430b-8f60-083b0d78dca2" />

<br>**Check-in**

<img width="573" alt="Screenshot 2025-06-04 at 10 24 24 PM" src="https://github.com/user-attachments/assets/e721b9e2-5f1d-4d95-809a-76b4b44cc0d6" />


<br>**Review**
<img width="1317" alt="Screenshot 2025-06-04 at 10 22 52 PM" src="https://github.com/user-attachments/assets/1a8622aa-5022-47ea-8070-8edc4409378b" />

<br>**Tip**
<img width="1068" alt="Screenshot 2025-06-04 at 10 20 32 PM" src="https://github.com/user-attachments/assets/25020ef7-0568-490e-b0fd-e90cc81118fb" />

<br>**User (22 columns)**
<img width="1365" alt="Screenshot 2025-06-04 at 10 20 45 PM" src="https://github.com/user-attachments/assets/5ea48f49-10ac-4d22-beb4-4fd5ff070998" />
<img width="1382" alt="Screenshot 2025-06-04 at 10 21 13 PM" src="https://github.com/user-attachments/assets/73179e5a-9198-4d47-bfd6-2b2c44ddc89a" />

<h2>Business Queries</h2>

<h3>1. finding the city</h3>

<code>
  SELECT TOP 20 *, COUNT(*) OVER (PARTITION BY city) AS high_rated_restaurant_count 
  FROM {table_name} 
  WHERE stars >= 3.5  AND [state] = 'IL'  AND categories LIKE '%Restaurants%' AND is_open = 1 
  ORDER BY high_rated_restaurant_count DESC ;
</code>

<img width="521" alt="Screenshot 2025-06-04 at 10 26 18 PM" src="https://github.com/user-attachments/assets/6eea44e7-66fd-42cc-a249-2b03ec2a83eb" />
<img width="676" alt="Screenshot 2025-06-04 at 10 26 29 PM" src="https://github.com/user-attachments/assets/aeb4e1ca-6e4e-49f2-96ac-62123a36395d" />

**displaying the locations using folium**

<img width="703" alt="Screenshot 2025-06-04 at 10 30 28 PM" src="https://github.com/user-attachments/assets/c8f36afa-31f0-43de-b978-c7fc039a35a5" />
<img width="290" alt="Screenshot 2025-06-04 at 10 31 59 PM" src="https://github.com/user-attachments/assets/1f1b92f7-8a3b-4aa9-af57-e6ad3eb59f25" />

<h3>2. Finding cities with High Percentage of Highly Rated Restaurants in Illinois</h3>

We decided that choosing cities with a high concentration of highly rated restaurants could help boost the visibility and reputation of the new restaurant by leveraging the existing food scene.

<code>
SELECT
    b.city,
    COUNT(CASE WHEN b.stars >= 4 THEN 1 END) * 100.0 / COUNT(b.business_id) AS high_rating_percentage,
    COUNT(b.business_id) AS restaurant_count
FROM business AS b
WHERE b.state = 'IL' AND b.categories LIKE '%Restaurant%'
GROUP BY b.city
HAVING COUNT(b.business_id) > 5
ORDER BY high_rating_percentage DESC;
</code>

<img width="506" alt="Screenshot 2025-06-04 at 10 38 33 PM" src="https://github.com/user-attachments/assets/26124ffc-36ae-4ec0-aae3-cfaffe627ddb" />

<br>Edwardsville	and Belleville	has the most high rated restaurants in the region

<h3>3. find the city in IL with the most high rated restaurants</h3>

<code>
  SELECT TOP 20 *, COUNT(*) OVER (PARTITION BY city) AS high_rated_restaurant_count 
  FROM {table_name} 
  WHERE stars >= 3.5  AND [state] = 'IL'  AND categories LIKE '%Restaurants%' AND is_open = 1 
  ORDER BY high_rated_restaurant_count DESC ;
</code>

<img width="1365" alt="Screenshot 2025-06-04 at 10 40 06 PM" src="https://github.com/user-attachments/assets/b34af85f-3ef0-4583-80a5-2b3479cf8b01" />



<h3>4. Calculate review-to-restaurant ratio for each city in Illinois</h3>
<code>
  WITH city_reviews AS (
    SELECT
        city,
        SUM(review_count) AS total_reviews,
        COUNT(business_id) AS restaurant_count,
        CAST(SUM(review_count) AS FLOAT) / COUNT(business_id) AS review_to_restaurant_ratio
    FROM business
    WHERE state = 'IL' AND categories LIKE '%Restaurant%'
    GROUP BY city
)

-- Step 2: Retrieve data for Belleville and the city with the second highest review count
SELECT city, total_reviews, restaurant_count, review_to_restaurant_ratio
FROM city_reviews
WHERE city IN (
    SELECT TOP 2 city
    FROM city_reviews
    ORDER BY total_reviews DESC
)
ORDER BY review_to_restaurant_ratio DESC;
</code>

<img width="626" alt="Screenshot 2025-06-04 at 10 45 31 PM" src="https://github.com/user-attachments/assets/9e8d6478-30f0-4be7-b8b9-6a986b7e2489" />

Edwardsville has a much higher review-to-restaurant ratio than Belleville, suggesting a larger customer base in Edwardsville, making it a better choice for a new restaurant.

<h3>5. Find the top five cities in Illinois with the most restaurants</h3>

<code>
SELECT TOP 5 city, COUNT(*) AS restaurant_count
FROM {table_name}
WHERE [state] = 'IL' AND categories LIKE '%Restaurant%' AND is_open = 1
GROUP BY city
ORDER BY restaurant_count DESC;
</code>

<img width="329" alt="Screenshot 2025-06-04 at 10 47 29 PM" src="https://github.com/user-attachments/assets/61349c89-9ec3-4008-a639-e2299ee25e14" />

<h3>6. Calculate the average number of reviews per restaurant in the city</h3>
<code>
SELECT AVG(review_count) AS avg_reviews
FROM business
WHERE business.city = 'Belleville' AND categories LIKE '%Restaurant%';
</code>

<img width="113" alt="Screenshot 2025-06-04 at 10 48 06 PM" src="https://github.com/user-attachments/assets/d3b7bd3c-6cd8-4228-9754-a0c1fd0046dd" />

<h3>7. Find top-rated restaurants in the city</h3>

<code>
  SELECT TOP 10
    name,
    stars,
    review_count,
    LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(categories, 'Restaurants, ', ''), ', Restaurants', ''), 'Restaurants', ''))) AS cuisine
FROM business
WHERE business.city = 'Belleville' AND categories LIKE '%Restaurant%'
ORDER BY stars DESC, review_count DESC;
</code>

<img width="744" alt="Screenshot 2025-06-04 at 10 49 32 PM" src="https://github.com/user-attachments/assets/3503c5b3-21ce-41fb-bae8-17be7c7c963f" />

<h3>8. Find the most popular cuisines in a specific city</h3>

<code>
  SELECT TOP 10
    LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(categories, 'Restaurants, ', ''), ', Restaurants', ''), 'Restaurants', ''))) AS category_cleaned,
    COUNT(business_id) AS num_restaurants
FROM business
WHERE business.city = 'Belleville' AND categories LIKE '%Restaurant%'
GROUP BY LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(categories, 'Restaurants, ', ''), ', Restaurants', ''), 'Restaurants', '')))
ORDER BY num_restaurants DESC;
</code>

<img width="501" alt="Screenshot 2025-06-04 at 10 50 55 PM" src="https://github.com/user-attachments/assets/1725d655-3d46-40df-ac7f-95ba69ec0e31" />

**The most popular restaurants in Belleville are Chinese, Barbeque and American (Traditional)**

<h3>9. Analyze customer reviews to identify common reviews and conduct sentiment analysis</h3>

Step 1: Retrieve top-rated restaurants in Belleville
<code>
  SELECT TOP 10
    business_id,
    name,
    stars,
    review_count,
    LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(categories, 'Restaurants, ', ''), ', Restaurants', ''), 'Restaurants', ''))) AS cuisine
FROM business
WHERE business.city = 'Belleville' AND categories LIKE '%Restaurant%'
ORDER BY stars DESC, review_count DESC;
</code>

Step 2: Fetch the reviews for these top-rated restaurants and perform sentiment analysis with python

<code>
  business_ids = tuple(top_restaurants_df['business_id'].tolist())
  SELECT text, business_id
  FROM review
  WHERE business_id IN {business_ids}
</code>
<br>
Perform sentiment analysis on each review
  
<code>
def get_sentiment_score(text):
  analysis = TextBlob(text)
  return analysis.sentiment.polarity   Returns a score between -1 and 1
</code>
<code>
Apply sentiment analysis to each review
reviews_df['sentiment_score'] = reviews_df['text'].apply(get_sentiment_score)
</code>

Step 3: Calculate the average sentiment score for each restaurant
<br>
<code>
avg_sentiment_df = reviews_df.groupby('business_id')['sentiment_score'].mean().reset_index()
avg_sentiment_df.rename(columns={'sentiment_score': 'average_sentiment_score'}, inplace=True)
</code>
<code>
Merge the sentiment scores with the top-rated restaurants DataFrame
result_df = pd.merge(top_restaurants_df, avg_sentiment_df, on='business_id')
</code>
Step 4: Sort by average sentiment score in descending order

<code>
result_df = result_df.sort_values(by='average_sentiment_score', ascending=False)

Display the final result
display(result_df)
</code>

<img width="1148" alt="Screenshot 2025-06-04 at 10 55 28 PM" src="https://github.com/user-attachments/assets/39a59787-3d6a-436f-9599-786335cd872a" />

<h3>10.Find the most popular price range among top-rated restaurants</h3>

<code>
  SELECT attributes_RestaurantsPriceRange2 AS price_range, COUNT(business_id) AS num_restaurants
  FROM business
  WHERE business.city = 'Belleville' AND categories LIKE '%Restaurant%' AND stars >= 3.5
  GROUP BY attributes_RestaurantsPriceRange2
  ORDER BY num_restaurants DESC;
</code>

## **what is price range?**

The price range in this context refers to a rating that indicates the relative cost of a restaurant.

In many datasets, including Yelp, price ranges are often represented by a numerical scale, with each number roughly corresponding to a level of expense.
Typically, they are represented as follows:

 	•	1: Low price range (e.g., inexpensive, budget-friendly).
 	•	2: Moderate price range (e.g., average pricing, mid-range).
 	•	3: High price range (e.g., upscale, expensive).
 	•	4: Very high price range (e.g., luxury, premium dining).

<h2>Conclusion</h2>

<br> **most of the restaurants are budget friendly, we should keep our restaurant in the low price range**

Based on the above analysis, we decided to open a **Chinese** in

---

Belleville

