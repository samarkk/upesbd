## Creating an index (with settings)
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}

## Indexing document with auto generated ID:
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

## Indexing document with custom ID:
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}

# Retrieving documents by ID
GET /products/_doc/100

# Updating documents
Updating an existing field
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  } 
}
# Adding a new field
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  } 
}
# Scripted updates
Reducing the current value of in_stock by one
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
# Assigning an arbitrary value to in_stock
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
# Using parameters with in scripts
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
# Conditionally setting the operation to noop
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.in_stock--;
    """
  }
# Conditionally update a field value
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}

# Conditionally delete a document
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.in_stock--;
    """
  }
}

# Upserts
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}

# Replacing documents
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 79,
  "in_stock": 4
}

# Deleting documents
DELETE /products/_doc/101

# Batch processing
Indexing documents
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }
Updating and deleting documents
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200 } }
Specifying the index name in the request path
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }
Retrieving all documents
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

# Importing data into local cluster
# Navigate to the directory containing the file or specify the full path 

curl --cacert [path_to_CA_certificate] -u elastic -H "Content-Type:application/x-ndjson" -XPOST https://localhost:9200/products/_bulk --data-binary "@products-bulk.json"

# Skipping the verification of the certificate (only for local development)
curl -H "Content-Type:application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"

# Searching with the request URI
# Matching all documents
GET /products/_search?q=*
# Matching documents containing the term Lobster
GET /products/_search?q=name:Lobster
# Matching documents containing the tag Meat
GET /products/_search?q=tags:Meat
# Matching documents containing the tag Meat and name Tuna
GET /products/_search?q=tags:Meat AND name:Tuna

# Matching all documents
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

# Searching for a term
# Matching documents with a value of true for the is_active field
GET /products/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
GET /products/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}

# Searching for multiple terms
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": [ "Soup", "Cake" ]
    }
  }
}

# Retrieving documents based on IDs
GET /products/_search
{
  "query": {
    "ids": {
      "values": [ 1, 2, 3 ]
    }
  }
}

# Matching documents with range values
# Matching documents with an in_stock field of between 1 and 5, both included
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
# Matching documents with a date range
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
# Matching documents with a date range and custom date format
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}

# Matching documents with non-null values
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}

# Matching based on prefixes
# Matching documents containing a tag beginning with Vege
GET /products/_search
{
  "query": {
    "prefix": {
      "tags.keyword": "Vege"
    }
  }
}

# Searching with wildcards
# Adding an asterisk for any characters (zero or more)
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*ble"
    }
  }
}
# Adding a question mark for any single character
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg?ble"
    }
  }
}
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veget?ble"
    }
  }
}

# Searching with regular expressions
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}

# Import data for full text search
# navigate to the directory where there is recipes-bulk.json
curl -H "Content-Type: application/x-ndjson" -XPOST "http://localhost:9200/recipe/bulk?pretty" --data-binary "@recipes-bulk.json"

# Flexible matching with match query
# Standard match query
GET /recipes/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}
Specifying a boolean operator
GET /recipes/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}
GET /recipes/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}

# Matching phrases
# The order of terms matters
GET /recipes/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "puttanesca spaghetti"
    }
  }
}

# Searching multiple fields
GET /recipes/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": [ "title", "description" ]
    }
  }
}

# Adding query clauses to the must key
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
# Moving the range query to the filter key
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
# Adding a query clause to the must_not key
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
# Adding a query clause to the should key
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parsley"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
