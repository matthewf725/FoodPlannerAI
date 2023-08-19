import overpy
import requests
import geopy.distance
from geopy.geocoders import Nominatim
from bs4 import BeautifulSoup
import openai

def getLink(place):
            # Replace with your actual Yelp API key
    API_KEY = "Yelp API key"

    # General latitude and longitude
    latitude = place[1]
    longitude = place[2]

    # Restaurant name
    restaurant_name = place[0]

    # Construct the search API endpoint URL
    search_url = f"https://api.yelp.com/v3/businesses/search"

    # Parameters for the search query
    params = {
        "term": restaurant_name,
        "latitude": latitude,
        "longitude": longitude,
    }

    # Set up the authorization header with the API key
    headers = {
        "Authorization": f"Bearer {API_KEY}",
    }

    # Make the API request
    response = requests.get(search_url, params=params, headers=headers)
    data = response.json()

    # Extract the business ID of the first restaurant from the response
    if "businesses" in data and len(data["businesses"]) > 0:
        return data
        #restaurant_id = data["businesses"][0]["id"]
def getMenu(alias):
    # Replace with the Yelp URL of the restaurant's menu page
    URL = "https://www.yelp.com/menu/" + alias
    items = []
    itemx = []
    # Send GET request to the restaurant's menu page
    response = requests.get(URL)
    
    # Check if the request was successful
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, "html.parser")

        # Find all menu items using the specified class
        menu_items = soup.find_all("div", class_="menu-item-details")
        menu_prices = soup.find_all("div", class_="arrange")
        if menu_items:
            i = 0
            for item in menu_items:
                item_name = item.find("h4").get_text(strip=True)
                item_description = item.find("p", class_="menu-item-details-description")
                if item_description:
                    item_description = item_description.get_text(strip=True)
                else:
                    item_description = "No description available"
                item_price = menu_prices[i].find("li", class_="menu-item-price-amount")
                if item_price:
                    item_price = item_price.get_text(strip=True)
                else:
                    item_price = "-1"
                item = (item_name, item_price, item_description)
                items.append(item)
                i += 2
        return items
    else:
        print("Error:", response.status_code)


def address_to_lat_long(address):
    geolocator = Nominatim(user_agent="address_converter")
    location = geolocator.geocode(address)
    if location:
        return location.latitude, location.longitude
    else:
        return None
    
def get_bounding_box(latitude, longitude, radius_in_meters):
    origin = (latitude, longitude)
    destination = geopy.distance.distance(meters=radius_in_meters).destination(origin, 45)  # 45-degree angle to account for diagonal distance
    return origin[0], origin[1], destination[0], destination[1]

def get_nearby_places(latitude, longitude, radius_in_meters):
    api = overpy.Overpass()

    # Calculate the bounding box based on the radius
    bbox = get_bounding_box(latitude, longitude, radius_in_meters)

    # Query for restaurants and grocery stores within the bounding box
    query = """
    (
        way["shop"="convenience"]({},{},{},{});
        node["amenity"="fast_food"]({},{},{},{});
        node["amenity"="restaurant"]({},{},{},{});
        node["amenity"="bar"]({},{},{},{});
        node["amenity"="biergarten"]({},{},{},{});
        node["amenity"="cafe"]({},{},{},{});
        node["amenity"="food_court"]({},{},{},{});
        node["amenity"="ice_cream"]({},{},{},{});
        node["amenity"="pub"]({},{},{},{});
        node["shop"="supermarket"]({},{},{},{});
        node["shop"="general"]({},{},{},{});
        node["shop"="spices"]({},{},{},{});
        node["shop"="seafood"]({},{},{},{});
        node["shop"="pastry"]({},{},{},{});
        node["shop"="pasta"]({},{},{},{});
        node["shop"="ice_cream"]({},{},{},{});
        node["shop"="health_food"]({},{},{},{});
        node["shop"="greengrocer"]({},{},{},{});
        node["shop"="frozen_food"]({},{},{},{});
        node["shop"="farm"]({},{},{},{});
        node["shop"="dairy"]({},{},{},{});
        node["shop"="deli"]({},{},{},{});
        
        node["shop"="confectionery"]({},{},{},{});
        node["shop"="chocolate"]({},{},{},{});
        node["shop"="cheese"]({},{},{},{});
        node["shop"="coffee"]({},{},{},{});
        node["shop"="butcher"]({},{},{},{});
        node["shop"="beverages"]({},{},{},{});
        node["shop"="bakery"]({},{},{},{});
        node["shop"="department_store"]({},{},{},{});
        node["shop"="kiosk"]({},{},{},{});
        node["shop"="mall"]({},{},{},{});
        node["shop"="wholesale"]({},{},{},{});
    );
    out;
    """.format(*bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox, *bbox)

    result = api.query(query)

    places = []
    for node in result.nodes:
        if "name" in node.tags:
            newplace = []
            newplace = (node.tags["name"], node.lat, node.lon)
            places.append(newplace)
    for way in result.ways:
        if "name" in way.tags:
            newplace = []
            newplace = way.tags["name"], latitude, longitude
            places.append(newplace)

    return places

def askGPT(items, budget):
    
    api_key = "YOUR_API_KEY"
    # Initialize the OpenAI API client
    openai.api_key = api_key

    # Define the prompt (question) you want to ask
    prompt = ("Create a menu for 7 days and all 3 meals with a budget of $" + budget + ". Below is a list that of food items with a name, a price, and a description which repeats. If a price is not listed it will say -1 and in this case give your best approximation. For homemade items please include a list of ingredients needed and their prices (to save money you can reuse previously bought ingredients if there is excess from before. The menu CANNOT go over budget (including tax) and must include a BREAKFAST, LUNCH, and DINNER. Please include a mixture of meals made at home and these menu items from restaurants. Menu items: "  )

    #Can you give me a grocery list for the week of homemade items for breakfast lunch and dinner and their respective costs and amount to buy?
    
    # Call the OpenAI API to generate a response
    response = openai.Completion.create(
        engine="davinci-codex",  # You can experiment with other engines as well
        prompt=prompt,
        max_tokens=50  # You can adjust this based on how long you want the response to be
    )

    # Print the generated response
    return response.choices[0].text.strip()

def main():
    address = input("Please enter your address: ")
    latitude, longitude = address_to_lat_long(address)
    radius_in_meters = 1000  # Radius in meters (adjust as needed)
    places = get_nearby_places(latitude, longitude, radius_in_meters)
    items = []
    if places:
        for place in places:
            place = getLink(place)
            items.append(getMenu(place["businesses"][0]["id"]))

    else:
        print("No places found.")

    budget = input("Please enter your budget: ")
    menu = askGPT(items, budget)
    print(menu)
    

if __name__ == "__main__":
    main()

