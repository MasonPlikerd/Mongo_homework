# DISCLAIMER!!! MongoDB will not run on my computer, no idea why. I have talked with tutors and instructors and we just cant get it figured out. This code is what I would do if everything ran

#Establishments Database Analysis 

## Overview
This project involves working with an `establishments` collection in a MongoDB database. The database contains information about various food establishments in the UK, including details such as hygiene scores, ratings, location data (latitude and longitude), and more. The following activities were carried out to analyze and manipulate the data.

## Activities

### 1. Add a New Restaurant
We added a new halal restaurant, "Penang Flavours," located in Greenwich. This entry included fields such as `BusinessName`, `BusinessType`, `Address`, `RatingValue`, and others. 

```python
# Add the new restaurant "Penang Flavours"
new_restaurant = {
    "BusinessName": "Penang Flavours",
    "BusinessType": "Restaurant/Cafe/Canteen",
    "AddressLine1": "Penang Flavours",
    "AddressLine3": "Greenwich",
    "PostCode": "SE10 9XYZ",
    "RatingValue": None,
    "LocalAuthorityName": "Greenwich",
    "Halal": True
}
db.establishments.insert_one(new_restaurant)
2. Find and Update BusinessTypeID
We searched for the BusinessTypeID corresponding to Restaurant/Cafe/Canteen and updated the "Penang Flavours" restaurant with the correct BusinessTypeID.

python
Copy code
# Update "Penang Flavours" with the correct BusinessTypeID
business_type_id = db.establishments.find_one({"BusinessType": "Restaurant/Cafe/Canteen"}, {"BusinessTypeID": 1})['BusinessTypeID']
db.establishments.update_one({"BusinessName": "Penang Flavours"}, {"$set": {"BusinessTypeID": business_type_id}})
3. Delete All Documents with Local Authority "Dover"
We deleted all records where LocalAuthorityName was "Dover" from the establishments collection.

python
Copy code
# Delete all documents where LocalAuthorityName is "Dover"
db.establishments.delete_many({"LocalAuthorityName": "Dover"})
4. Convert Longitude and Latitude to Decimal
We converted the longitude and latitude fields from strings to decimal values for more accurate geospatial analysis.

python
Copy code
# Convert longitude and latitude to decimal
db.establishments.update_many(
    {"geocode.longitude": {"$type": "string"}, "geocode.latitude": {"$type": "string"}},
    [{"$set": {"geocode.longitude": {"$toDouble": "$geocode.longitude"}, "geocode.latitude": {"$toDouble": "$geocode.latitude"}}}]
)
5. Set Non-Numeric Rating Values to Null
We set non-numeric RatingValue entries (such as "AwaitingInspection" and "Pass") to null.

python
Copy code
# Set non-numeric RatingValue to null
non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
db.establishments.update_many({"RatingValue": {"$in": non_ratings}}, {"$set": {"RatingValue": None}})
6. Convert RatingValue from String to Integer
We converted the RatingValue from string to integer for all records where the RatingValue was a number between 1 and 5.

python
Copy code
# Convert numeric RatingValue from string to integer
db.establishments.update_many({"RatingValue": {"$in": ["1", "2", "3", "4", "5"]}}, 
    [{"$set": {"RatingValue": {"$toInt": "$RatingValue"}}}])
7. Find Establishments with a Hygiene Score of 20
We queried the establishments collection for records with a hygiene score of 20 and displayed the first result. We also converted the result to a Pandas DataFrame.

python
Copy code
# Query for establishments with a hygiene score of 20 and convert to DataFrame
query = {"scores.Hygiene": 20}
results = list(db.establishments.find(query))
df = pd.DataFrame(results)
8. Search by Proximity and Rating
We performed a search for establishments within 0.01 degrees of a given latitude and longitude, with a RatingValue of 5, and sorted by hygiene score.

python
Copy code
# Search for establishments near a given location with a RatingValue of 5, sorted by hygiene score
latitude = 51.5074
longitude = -0.1278
degree_search = 0.01
query = {
    "RatingValue": 5,
    "geocode.latitude": {"$gte": latitude - degree_search, "$lte": latitude + degree_search},
    "geocode.longitude": {"$gte": longitude - degree_search, "$lte": longitude + degree_search}
}
results = list(db.establishments.find(query).sort([("scores.Hygiene", 1)]))
df = pd.DataFrame(results)
9. Aggregation: Find Establishments with Hygiene Score of 0
We used an aggregation pipeline to match establishments with a hygiene score of 0, group them by LocalAuthorityName, and sort the results from highest to lowest.

python
Copy code
# Aggregation pipeline to find establishments with a hygiene score of 0
pipeline = [
    {"$match": {"scores.Hygiene": 0}},
    {"$group": {"_id": "$LocalAuthorityName", "count": {"$sum": 1}}},
    {"$sort": {"count": -1}}
]
results = list(db.establishments.aggregate(pipeline))
df_results = pd.DataFrame(results)
Dependencies
Python 3.x
Pandas
PyMongo
MongoDB
How to Run
Install dependencies: pip install pymongo pandas
Ensure MongoDB is running on your system.
Import or create the required collections and follow the steps outlined above to perform the operations on the establishments collection.
vbnet
