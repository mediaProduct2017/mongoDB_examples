# mongoDB_examples

## why mongoDB?

Flexible schema, such as hierarchical data

Json data structure maps to data structure in many programming languages, such as python dictionaries

Flexible deployment, such as on laptop or on multiple servers

Designed for big data

Aggregation framework

## 使用mongoDB

安装mongoDB

[Install MongoDB](https://docs.mongodb.com/manual/installation/)

[udacity install mongoDB](https://www.udacity.com/wiki/ud032#installing-mongodb)

安装database driver

The data exchanged between pymongo and mongoDB is in a form called BSON, the binary encoding for Json

    pip install pymongo
    
建立对mongoDB的连接

    def get_db():
        # For local use
        from pymongo import MongoClient
        client = MongoClient('localhost:27017')
        # 'examples' here is the database name. It will be created if it does not exist.
        db = client.examples
        # specify we wanna use the examples database
        return db
        
    client = MongoClient('mongodb://localhost:27017/')
        
插入数据

    def add_city(db):
        # Changes to this function will be reflected in the output. 
        # All other functions are for local use only.
        # Try changing the name of the city to be inserted
        db.cities.insert({"name" : "Chicago"})
        
    db.autos.insert(tesla_s)
    
更新

    def main():
        city = db.cities.find_one({"country": "Germany"})
        city["isoCountryCode"] = "deu"
        db.cities.save(city)
        
    def main():
        db.cities.update({"country": "Germany"}, {"$set": {"isoCountryCode": "deu"}}, multi=True)
        
删除数据

    db.cities.remove()  # delete one by one
    db.citeis.drop()  # delete the entire collection at once
        
查询数据

    import pprint

    def get_city(db):
        return db.cities.find_one()
        
    for a in db.cities.find()
        pprint.pprint(a)
        
    autos = db.autos.find({"manufacturer": "Toyota"})
    
    for a in autos:
    pprint.pprint(a)
    
    autos = db.autos.find({"manufacturer": "Toyota", "class": "mid size car"})
    
投影查询

    projection = {"_id": 0, "name": 1}
    autos = db.autos.find({"manufacturer": "Toyota"}, projection)
    
数数

    num_autos = db.autos.find().count()
    
pymongo 中的批量插入

[pymongo 3.0](http://api.mongodb.com/python/current/tutorial.html#bulk-inserts)

[pymongo 2.8](http://api.mongodb.com/python/2.8/tutorial.html#bulk-inserts)
    
在命令行使用mongoimport

    mongoimport -d examples -c autos --file autos.json
    
    -d: database

    -c: collection

range operation

    query = {"population": {"$gt": 250000, "lte": 500000}}
    cities = db.cityies.find(query)
    
    query = {"foundingDate": {"gt": datetime(1837, 1, 1), "lte": datetime(1837, 12, 31)}}
    
Swith database

    use examples
    
exists operator and pretty printer

    db.cities.find({"govermentType": {"$exists": 1}}).pretty()

正则运算符用于查询

    db.cities.find({"motto": {"$regex": "[Ff]riendship|[Hh]appiness"}})
    
In operator: the field contains any item within the in operator

    db.autos.find({"ModelYear": {"$in": [1956, 1957, 1958]}})
    
All operator: the field contains all the items within the all operator

    db.autos.find({"ModelYear": {"$all": [1956, 1957, 1958]}})
    
dot representation

    db.autos.find({"dimentions.weight": {"$gt": 5000}})
    db.tweets.find("entities.hashtags": {"$ne": []}}, {"entities.hashtags.text": 1, "_id": 0})
    
## The aggregation framework

aggregation pipeline

    $group: $sum, $first, $last, $min, $max, $avg

    def most_tweets():
        res = db.tweets.aggragate([
            {"$group": {"_id": "$user.screenname",  
            # group operation, the operation key is user.screenname
                  "count": {"$sum": 1}}},  # accumulation (sum) operation to get count
            {"$sort": {"count": -1}}])  # -1 means descending order
        return res
        # return an iterator
        
[Aggregation Pipeline Operators](https://docs.mongodb.com/manual/reference/operator/aggregation/)

In the current version of pymongo (3.0), aggregation operations return a cursor object. In order to see the elements returned, you can iterate over the cursor object, such as using a for loop, and print out the elements one by one. 

[pipeline operators and project operator](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#pipe._S_project)

    $project
    
    $match
    
    def highest_ratio():
        res = db.tweets.aggragate([
        {"$match": {"user.friends_count": {"$gt": 0},
                    "user.followers_count": {"$gt": 0}}},
         # a filter operator
        {"$project": {"ratio": {"$divide": ["$user.followers_count", "$user.friends_count"]},
                      "screen_name": $user.screen_name}},
         # a shaping operator
        {"$sort": {"ratio": -1}},
        {"$limit": 1}])
        return res
    
    $skip
    
    $limit
    
    $unwind
    
    def user_mentions():
        res = db.tweets.aggragate([
        {"$unwind": "entities.user_mentions"},
        # 把内部的一个array展开，其后往往紧跟$group，对展开后的值进行聚合处理
        {"$group": {"_id": "$entities.user_mentions.screenname", 
                    "count": {"$sum": 1}}},
        {"$sort": {"count": -1}},
        {"$limit": 1}]) 
        return res
    
    $addToSet
    
    def user_mentions():
        res = db.tweets.aggragate([
        {"$unwind": "entities.user_mentions"},
        # 把内部的一个array展开，其后往往紧跟$group，对展开后的值进行聚合处理
        {"$group": {"_id": "$user.screenname", 
        "unique_metions": {"$addToSet: "$entities.user_mentions.screenname"}}},
        {"$sort": {"count": -1}},
        {"$limit": 1}]) 
        return res
        
Who has mentioned the most unique users?

    def unique_user_mentions():
        res = db.tweets.aggragate([
        {"$unwind": "entities.user_mentions"},
        # 把内部的一个array展开，generate one document for every item in the array
        {"$group": {"_id": "$user.screenname", 
        "unique_metions": {"$addToSet: "$entities.user_mentions.screenname"}}},
        {"$unwind": "$unique_metions"},
        {"$group": {"_id": "$_id", 
        "count": {"$sum: 1}}},        
        {"$sort": {"count": -1}},
        {"$limit": 10}]) 
        return res
    
Indexes

    db.nodes.find().pretty()
    
    db.nodes.ensureIndex({"tg":1})
    
[Learn MongoDB from MongoDB -- mongodb university](https://university.mongodb.com/)

[Indexes -- mongodb documentation](https://docs.mongodb.com/manual/indexes/)

geospacial indexes

    db.nodes.ensureIndex([("loc", pymongo.GEO2D)])
    
    db.nodes.find({"loc": {"$near": [41.9, -87.6]}, "tag": {"$exist": 1}}).pretty()
    
    
    