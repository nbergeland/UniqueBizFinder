# UniqueBizFinder
Finds businesses which exist in one market, but not another.  Example used is Fargo, ND vs. Grand Forks, ND.

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
