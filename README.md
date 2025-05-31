# UniqueBizFinder

## Overview
The UniqueBizFinder is a business intelligence tool designed to identify businesses that exist in one geographic market but not in another. The current implementation specifically compares business presence between Fargo, ND and Grand Forks, ND using the Google Places API to detect market opportunities and competitive gaps.

## System Architecture
The UniqueBizFinder operates as a data processing pipeline that integrates with external APIs to perform market analysis. The system follows a simple but effective architecture centered around the Jupyter notebook implementation.

### Core System Components
<img width="1452" alt="Screenshot 2025-05-31 at 8 33 31 AM" src="https://github.com/user-attachments/assets/7b8ff6ba-9e97-41b0-ad11-ac40de3371d9" />

## Implementation Structure
The system is implemented as a single Jupyter notebook containing all the business logic, API integration, and data processing functionality. The implementation follows a functional programming approach with clearly defined helper functions.
<img width="762" alt="Screenshot 2025-05-31 at 8 35 04 AM" src="https://github.com/user-attachments/assets/0a7ee3e3-856d-45d6-bc01-0a37cca7d597" />

## Geographic Configuration
The system is configured to compare two specific markets with hardcoded geographic centers:
<img width="628" alt="Screenshot 2025-05-31 at 8 36 31 AM" src="https://github.com/user-attachments/assets/86c4e18a-3b70-43d8-8592-83e8527d38be" />

## Data Processing Pipeline
The system implements a multi-stage data processing pipeline that transforms raw API responses into actionable business intelligence.

### Pipeline Flow
<img width="772" alt="Screenshot 2025-05-31 at 8 37 52 AM" src="https://github.com/user-attachments/assets/cb08aa93-5685-499e-adc1-92ba1fec034f" />

## API Integration and Rate Limiting
The system implements proper Google Places API integration with pagination support and rate limiting compliance.

### API Request Handling
<img width="871" alt="Screenshot 2025-05-31 at 8 39 58 AM" src="https://github.com/user-attachments/assets/63b8f34c-6cef-40fe-85bd-5c7179280490" />

## Output Format and Results
The system generates two primary outputs: a CSV file for data persistence and console output for immediate review.
<img width="708" alt="Screenshot 2025-05-31 at 8 41 53 AM" src="https://github.com/user-attachments/assets/e121f6c9-5ecc-4253-8d12-36d17574d363" />
The system outputs businesses found in Fargo but not in Grand Forks, representing potential market opportunities for expansion or investment analysis.

# Code Notebook

```
import os
import googlemaps
import pandas as pd
from dotenv import load_dotenv

# === SETUP ===
load_dotenv()  # Load environment variables

GOOGLE_API_KEY = os.getenv('GOOGLE_API_KEY')
gmaps = googlemaps.Client(key=GOOGLE_API_KEY)

# === HELPER FUNCTIONS ===

def get_places_by_keyword(location, keyword, radius=10000):
    """
    Search for places in a given location using Google Places API.
    """
    results = []
    page_token = None

    while True:
        if page_token:
            response = gmaps.places_nearby(
                location=location,
                radius=radius,
                keyword=keyword,
                page_token=page_token
            )
        else:
            response = gmaps.places_nearby(
                location=location,
                radius=radius,
                keyword=keyword
            )

        results.extend(response.get('results', []))
        page_token = response.get('next_page_token')

        if not page_token:
            break

        # Google requires a brief delay before using the next_page_token
        import time
        time.sleep(2)

    # Extract relevant info
    data = [{
        'name': place['name'],
        'type': ', '.join(place.get('types', [])),
        'address': place.get('vicinity', ''),
        'place_id': place['place_id']
    } for place in results]

    return pd.DataFrame(data)

def get_unique_fargo_businesses():
    """
    Get businesses in Fargo but not Grand Forks.
    """
    # City Centers (correct format)
    fargo_center = (46.8772, -96.7898)
    grand_forks_center = (47.9253, -97.0329)

    # Keywords to search
    keywords = ['fast food', 'quick serve restaurant', 'bank']

    # Query Fargo and Grand Forks
    fargo_df = pd.concat([get_places_by_keyword(fargo_center, kw) for kw in keywords], ignore_index=True)
    grand_forks_df = pd.concat([get_places_by_keyword(grand_forks_center, kw) for kw in keywords], ignore_index=True)

    # Deduplicate
    fargo_df = fargo_df.drop_duplicates(subset='name')
    grand_forks_df = grand_forks_df.drop_duplicates(subset='name')

    # Find businesses in Fargo but not Grand Forks
    fargo_unique = fargo_df[~fargo_df['name'].isin(grand_forks_df['name'])].reset_index(drop=True)

    return fargo_unique

# === MAIN EXECUTION ===

if __name__ == '__main__':
    fargo_unique_businesses = get_unique_fargo_businesses()

    # Save to CSV
    fargo_unique_businesses.to_csv('fargo_unique_businesses.csv', index=False)

    print("✅ Unique businesses in Fargo but not Grand Forks saved as 'fargo_unique_businesses.csv'")
    print(fargo_unique_businesses.head(10))
```
✅ Unique businesses in Fargo but not Grand Forks saved as 'fargo_unique_businesses.csv'
                                     
 **Name**
 
0                          Sweeto Burrito   
1                             Burger Time   
2               Popeyes Louisiana Kitchen   
3  Freddy's Frozen Custard & Steakburgers   
4                             Smashburger   
5                           Slim Chickens   
6                      Dave's Hot Chicken   
7        Hangry Joe's Hot Chicken & Wings   
8                  Chipotle Mexican Grill   
9                               Taco Shop   

 **Type**
 
0  restaurant, food, point_of_interest, establish...   
1  restaurant, food, point_of_interest, establish...   
2  restaurant, food, point_of_interest, establish...   
3  restaurant, food, point_of_interest, establish...   
4  store, restaurant, food, point_of_interest, es...   
5  meal_takeaway, restaurant, food, point_of_inte...   
6  restaurant, food, point_of_interest, establish...   
7  meal_takeaway, restaurant, food, point_of_inte...   
8  restaurant, food, point_of_interest, establish...   
9  restaurant, food, point_of_interest, establish...   
 
  **Location**
  
6  4445 17th Ave S suite 3b, Fargo  ChIJO4R2FwDLyFIRiA2ZDmjQFWg  
7   1100 19th Ave N suite e, Fargo  ChIJN48pAS_JyFIRbH879xif2sA  
8            1680 45th St S, Fargo  ChIJaT0JVF7LyFIR37riolQttMs  
9      1825 S University Dr, Fargo  ChIJheBS7gXMyFIRRqz8RHPETPQ 
