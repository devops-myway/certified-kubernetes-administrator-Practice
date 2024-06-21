https://www.baeldung.com/linux/jq-command-json

######  jq — an eloquent command-line processor for JSON.

Unfortunately, shells such as Bash can’t interpret and work with JSON directly. This means that working with JSON via the command line can be cumbersome, involving text manipulation using a combination of tools such as sed and grep.

##### Working With Simple Filters
jq is built around the concept of filters that work over a stream of JSON
Each filter takes an input and emits JSON to standard out.

``````sh
echo '{"fruit":{"name":"apple","color":"green","price":1.20}}' | jq '.'

# We can also apply this filter directly to a JSON file
jq '.' fruit.json

``````
###### Let’s hit a simple API using curl to see this in practice

``````sh
curl http://api.open-notify.org/iss-now.json | jq '.'

{
  "message": "success",
  "timestamp": 1572386230,
  "iss_position": {
    "longitude": "-35.4232",
    "latitude": "-51.3109"
  }
}
``````
###### Accessing Properties
We can access property values by using another simple filter: the .field operator. To find a property value, we simply combine this filter followed by the property name.

``````sh
jq '.fruit' fruit.json

# We can also chain property values together, allowing us to access nested objects
jq '.fruit.color' fruit.json

#If we need to retrieve multiple keys, we can separate them using a comma:

jq '.fruit.color,.fruit.price' fruit.json

# Note that if one of the properties has spaces or special characters, we need to wrap the property name in quotes when accessing it from the jq command:

echo '{ "with space": "hello" }' | jq '."with space"'

``````
###### JSON Arrays
We typically use arrays to represent a list of items
square brackets to denote the start and end of an array.

##### iterate over an array
we see the object value iterator operator.[] in use, which will print out each item in the array on a separate line.
``````sh
echo '["x","y","z"]' | jq '.[]'

``````
###### Iterate magine we want to represent a list of fruit in a JSON document:
Let’s see how to extract the name of each fruit from each object in the array:
First, we iterate over the array using .[]. 
Then we can pass each object in the array to the next filter in the command using a pipe |. The last step is to output the name field from each object using .name:

``````sh
[
  {
    "name": "apple",
    "color": "green",
    "price": 1.2
  },
  {
    "name": "banana",
    "color": "yellow",
    "price": 0.5
  },
  {
    "name": "kiwi",
    "color": "green",
    "price": 1.25
  }
]

jq '.[] | .name' fruits.json

``````
###### Accessing Index &  Slicing

``````sh
jq '.[1].price' fruits.json

echo '[1,2,3,4,5,6,7,8,9,10]' | jq '.[6:9]'


``````
###### Using Functions, getting Keys, Lenght return
we may want to get the keys of an object as an array instead of the values
We can use this function to return the array’s length or the number of properties on an object:

``````sh
jq '.fruit | keys' fruit.json

jq '.fruit | length' fruit.json

``````

###### Mapping Values

``````sh
jq 'map(has("name"))' fruits.json

jq 'map(.price+2)' fruits.json

``````
###### Min and Max

``````sh
jq '[.[].price] | min' fruits.json

jq '[.[].price] | max' fruits.json

``````
######  Selecting Values

``````sh
jq '.[] | select(.price>0.5)' fruits.json
jq '.[] | select(.color=="yellow")' fruits.json
jq '.[] | select(.color=="yellow" and .price>=0.5)' fruits.json


``````
