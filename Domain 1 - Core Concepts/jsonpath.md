
https://kubernetes.io/docs/reference/kubectl/jsonpath/

##### JSONPath

Every JSON ends up creating a tree of nodes, where each node is a JSON Element. Every JSON object is composed on an inherent hierarchy and structure.
A simple JSON expressing a collection of countries as below. kubectl does not allow regular expressions using JSONPath.

JSONPath	                              Example K8s                                                  Description
-----------------------------------------------------------------------------------------------------------------------------
$	                                                                                        Represents the root object or node.
@	                                                                                        Represents the current object or node.
. or []	                                                                                    This is the child operator.
..	                                                                                        Recursive descendant.
*	                                {.items[*].metadata.name}                               Represent a wildcard to obtain all objects.
[]	                                                                                        Used to iterate through elements.
[,]	                                                                                        Union operator.
[start:end:step]	                                                                        Slice array operator inherited from ES4.
?()	                               {.users[?(@.name=="e2e")].user.password}                 Filter operator.
()	                                                                                        Script expression.

 use of the “range/end” notation to iterate a list and do something for each item, in this case just add a break line.

At the top most level we have a Root node, which is basically the node containing all of the current JSON. Inside this root node, we have following nodes
Description - Description is simple leaf nodes in the tree. maps or dictionaries
Region     -   Region is simple leaf nodes in the tree. maps or dictionaries
Countries - But Countries is a non-leaf node, which further contains more nodes - Countries node contains an array of two countries - array or list
phoneNumbers - phonenumbers is a non-leaf node, which further contains more nodes - phonenumbers node contains an array of two numbers - array or list

``````sh
{
  "Description": "Map containing Country, Capital, Currency, and some States of that Country",
  "Region": "Asia",
  "Countries": [
    {
      "Country": "India",
      "Data": {
        "Capital": "New Delhi",
        "mintemp": 6,
        "maxtemp": 45,
        "Currency": "Rupee"
      }
    },
    {
      "Country": "Nepal",
      "Data": {
        "Capital": "Katmandu",
        "mintemp": 9,
        "maxtemp": 23,
        "Currency": "Nepalese rupee"
      }
    }
  ]
}

---------------------------------
{
    "firstName": "John",
    "lastName": "doe",
    "age": 26,
    "address": {
        "streetAddress": "naist street",
        "city": "Nara",
        "postalCode": "630-0192"
    },
    "phoneNumbers": [
        {
            "type": "iPhone",
            "number": "0123-4567-8888"
        },
        {
            "type": "home",
            "number": "0123-4567-8910"
        }
    ]
}

``````
represent Root node by $ and a relationship from parent to child with >>
Description node will be represented by $ >> Description.
Region node will be represented by $ >> Region.
Root node and the zero'th item in the Countries array
 $ >> Countries[0] where [ ] is the index operator to represent an item at n index in an Array of Countries.

###### How to Query JSON using JSONPath
In order to get children of a given node, we can use the Dot (.) operator or the ['childname'] operator.


In order to get all the Countries we can have JSONPath as:

``````sh
$.Countries
$['Countries']

``````
##### Wildcard operator in JSONPath - means everything under that node.
 if you want to display the Data nodes of all the countries 
``````sh
$.Countries[].Data*
$['Countries'][].Data*

``````
###### Array Index operator in JSONPath
We can use the Array Index [i,j,k...0,1,2 ]  to identify an entry at a particular index.
Array index starts from 0. Hence to refer to the second item in the array we have to use 1 as the index.
let us find out the last Country entry in the Countries array.
``````sh
$.Countries[1]
$['Countries'][1]

``````
###### Access Field From Root
access a field in the root
``````sh
$.firstName
``````
##### Access Field From Nested Object
to access the city name under the address object
``````sh
$.address.city

``````
##### Access Fields From Nested Array
we are going to access data from our list of phone numbers. we use the wildcard number to get every element in the phoneNumbers array.
``````sh
$.phoneNumbers[*].number

# What if we only need the “home” phone number? This is where the filter notation can come handy.

$.phoneNumbers[?(@.type == "home")].number
``````


###### How to Use JSONPath in Kubernetes

to get a list of running pods in JSON format we’d have to run something like this
``````sh
kubectl get pods -o json

``````
##### Get Nth Element in Array
get only the data for an element in the “n” position of an array.
You can see how we get the first element from the items object contained in the Kubernetes JSON response by using []notation
``````sh
$ kubectl -n my-namespace get pods -o jsonpath='{$.items[0]}' | jq

``````
###### Get Single Field from Nth Element in Array
so we’ll get the first element and fetch a single field from the JSON response
``````sh
kubectl -n my-namespace get pods -o jsonpath='{$.items[0].status.hostIP}'

``````
##### Get Single Field From Each Element in Array
We’d like to get a plain list of IP addresses that, potentially, we could use to perform a further action

Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt.
``````sh
kubectl get nodes -ojson
kubectl get nodes -o jsonpath='{$.items[*].status.images}'
kubectl get nodes -o jsonpath={$.items[0].status.images}
kubectl -n my-namespace get pods -o jsonpath={$.items[*].status.hostIP}

10.154.196.228 10.154.202.136 10.154.201.54

``````
##### Get Multiple Fields From Each Element in Array
 range and end keywords and it’ll be simpler than what it looks like.

``````sh
$ kubectl -n my-namespace get pods -o jsonpath={range .items[*]}{.status.hostIP}{"\t"}{.status.phase}{"\n"}{end}

10.154.196.228	Running
10.154.202.136	Running
10.154.201.54	Running

``````
``````sh


``````