# GraphDB-Research

## Table of Contents
- [Introduction to Knowledge Graphs and Graph Databases](#introduction-to-knowledge-graphs-and-graph-databases)
- [Core Concepts](#core-concepts)
  - [What are Nodes?](#what-are-nodes)
  - [What are Edges?](#what-are-edges)
  - [Properties and Labels](#properties-and-labels)
- [How Graph Databases Work Under the Hood](#how-graph-databases-work-under-the-hood)
- [Creating Your Own Knowledge Graph](#creating-your-own-knowledge-graph)
  - [Step 1: Define Your Domain Model](#step-1-define-your-domain-model)
  - [Step 2: Choose a Graph Database](#step-2-choose-a-graph-database)
  - [Step 3: Import Your Data](#step-3-import-your-data)
- [Querying Graph Databases](#querying-graph-databases)
  - [Cypher Query Examples](#cypher-query-examples)
  - [SPARQL Query Examples](#sparql-query-examples)
- [Automating Tabular Data Conversion](#automating-tabular-data-conversion)
  - [Excel to Graph Database](#excel-to-graph-database)
  - [CSV to Graph Database](#csv-to-graph-database)
  - [Census Data Example](#census-data-example)
  - [Weather Data Example](#weather-data-example)
- [Exposing APIs for Non-Technical Users](#exposing-apis-for-non-technical-users)
  - [REST API Approach](#rest-api-approach)
  - [GraphQL API Approach](#graphql-api-approach)
- [Practical Examples](#practical-examples)
  - [Census Data Knowledge Graph](#census-data-knowledge-graph)
  - [Weather Data Knowledge Graph](#weather-data-knowledge-graph)
- [Best Practices](#best-practices)
- [Resources](#resources)

---

## Introduction to Knowledge Graphs and Graph Databases

### What is a Knowledge Graph?

A **knowledge graph** is a structured representation of real-world entities and their relationships. It captures semantic information in a way that both humans and machines can understand. Knowledge graphs organize data as interconnected nodes (entities) and edges (relationships), forming a network of information.

**Real-world examples:**
- **Google Knowledge Graph**: Powers search results with factual information about people, places, and things
- **Facebook Social Graph**: Represents users, posts, likes, and friendships
- **Amazon Product Graph**: Connects products, categories, reviews, and purchasing patterns

### What is a Graph Database?

A **graph database** is a specialized database designed to store and query graph-structured data efficiently. Unlike traditional relational databases that use tables, rows, and columns, graph databases use nodes, edges, and properties to represent and store data.

**Key advantages:**
- **Performance**: Graph databases excel at traversing relationships, making complex queries on connected data significantly faster
- **Flexibility**: Schema-less or flexible schema design allows for easy evolution of data models
- **Intuitiveness**: The visual nature of graphs makes it easier to understand and communicate data relationships
- **Pattern Recognition**: Excellent for discovering hidden patterns and connections in data

---

## Core Concepts

### What are Nodes?

**Nodes** (also called vertices) represent entities or objects in your domain. Each node can have:
- **Labels**: Categories or types (e.g., Person, City, Product)
- **Properties**: Key-value pairs storing attributes (e.g., name: "John", age: 30)

**Examples:**
```
Node 1: (:Person {name: "Alice", age: 28, occupation: "Data Scientist"})
Node 2: (:City {name: "Columbus", state: "Ohio", population: 905748})
Node 3: (:WeatherStation {id: "KOLU", location: "Columbus Airport"})
```

### What are Edges?

**Edges** (also called relationships or links) represent connections between nodes. Each edge has:
- **Type**: Describes the nature of the relationship (e.g., LIVES_IN, WORKS_FOR, MEASURED_BY)
- **Direction**: Can be directed or undirected
- **Properties**: Additional attributes about the relationship (e.g., since: 2020, strength: 0.8)

**Examples:**
```
(Alice)-[:LIVES_IN {since: 2020}]->(Columbus)
(Columbus)-[:HAS_STATION]->(KOLU)
(KOLU)-[:RECORDED {date: "2023-06-15", temp: 75}]->(Temperature)
```

### Properties and Labels

**Properties** are key-value pairs that store information about nodes and edges:
```
Person node properties: {name, age, email, phone}
WORKS_FOR relationship properties: {start_date, position, salary}
```

**Labels** categorize nodes and help organize the graph:
```
:Person, :Company, :City, :State, :Country
:WeatherStation, :Temperature, :Precipitation
:CensusBlock, :Demographic, :HouseholdIncome
```

---

## How Graph Databases Work Under the Hood

### Storage Architecture

Graph databases use specialized storage mechanisms optimized for graph traversal:

#### 1. **Index-Free Adjacency**
Each node maintains direct references (pointers) to its adjacent nodes. This means:
- Traversing from one node to another is an O(1) operation
- No need to perform index lookups for relationship traversal
- Query performance is independent of total graph size

**Example:**
```
Node A stores: [pointer to B, pointer to C, pointer to D]
To find A's neighbors: directly follow pointers (no table scans!)
```

#### 2. **Native Graph Storage**
- **Nodes**: Stored with unique IDs, labels, and property maps
- **Relationships**: Stored with start node, end node, type, and properties
- **Indexes**: B-tree or hash indexes on node/relationship properties

**Physical Layout:**
```
Node Store:
ID | Labels        | Properties
1  | :Person       | {name: "Alice", age: 28}
2  | :City         | {name: "Columbus", pop: 905748}

Relationship Store:
ID | StartNode | EndNode | Type      | Properties
10 | 1         | 2       | :LIVES_IN | {since: 2020}
```

#### 3. **Graph Traversal**
When executing a query like "Find friends of friends":
1. Start at person node
2. Follow FRIEND_OF edges (direct pointer access)
3. From each friend, follow their FRIEND_OF edges
4. Collect and deduplicate results

This is dramatically faster than JOIN operations in relational databases.

### Memory Management

Modern graph databases use:
- **Cache layers**: Keep frequently accessed nodes/edges in RAM
- **Page cache**: Optimize disk I/O for graph pages
- **Query optimization**: Analyze query patterns and optimize traversal paths

---

## Creating Your Own Knowledge Graph

### Step 1: Define Your Domain Model

Identify the entities and relationships in your domain:

**Example: Census Data Domain**
```
Entities (Nodes):
- State
- County
- CensusTract
- CensusBlock
- Demographic
- HouseholdIncome

Relationships (Edges):
- State -[:CONTAINS]-> County
- County -[:CONTAINS]-> CensusTract
- CensusTract -[:CONTAINS]-> CensusBlock
- CensusBlock -[:HAS_DEMOGRAPHIC]-> Demographic
- CensusBlock -[:HAS_INCOME_DATA]-> HouseholdIncome
```

### Step 2: Choose a Graph Database

Popular options:

**Neo4j** (Property Graph)
- Most popular graph database
- Cypher query language
- Excellent tooling and community
- Good for: Small to medium graphs, developer-friendly

**Amazon Neptune** (Property Graph & RDF)
- Fully managed cloud service
- Supports both Gremlin and SPARQL
- Good for: AWS-based solutions, high availability

**Apache Jena (TDB)** (RDF/Triple Store)
- Open-source semantic web framework
- SPARQL query language
- Good for: Semantic web applications, ontologies

### Step 3: Import Your Data

We'll cover this in detail in the automation section below.

---

## Querying Graph Databases

### Cypher Query Examples

Cypher is the query language for Neo4j and is highly readable:

#### Basic Pattern Matching
```cypher
// Find all people living in Columbus
MATCH (p:Person)-[:LIVES_IN]->(c:City {name: "Columbus"})
RETURN p.name, p.age

// Find weather stations in Ohio
MATCH (s:State {name: "Ohio"})-[:HAS_CITY]->(c:City)-[:HAS_STATION]->(ws:WeatherStation)
RETURN c.name, ws.id, ws.location
```

#### Advanced Queries
```cypher
// Find census blocks with high income in urban areas
MATCH (state:State)-[:CONTAINS]->(county:County)-[:CONTAINS]->(tract:CensusTract)
      -[:CONTAINS]->(block:CensusBlock)-[:HAS_INCOME_DATA]->(income:HouseholdIncome)
WHERE income.median > 75000 AND tract.urban = true
RETURN block.id, income.median, county.name
ORDER BY income.median DESC
LIMIT 10

// Correlate weather and census data
MATCH (city:City)<-[:LIVES_IN]-(p:Person),
      (city)-[:HAS_STATION]->(ws:WeatherStation)-[:RECORDED]->(temp:Temperature)
WHERE temp.date = date('2023-06-15')
RETURN city.name, count(p) as population, avg(temp.value) as avg_temp
```

#### Aggregations
```cypher
// Calculate average temperature by city
MATCH (c:City)-[:HAS_STATION]->(ws:WeatherStation)-[:RECORDED]->(t:Temperature)
WHERE t.date >= date('2023-01-01') AND t.date <= date('2023-12-31')
RETURN c.name, 
       avg(t.value) as avg_temp,
       min(t.value) as min_temp,
       max(t.value) as max_temp
ORDER BY avg_temp DESC
```

### SPARQL Query Examples

SPARQL is used for RDF/semantic web graphs:

#### Basic Triple Patterns
```sparql
# Find all cities in Ohio
PREFIX geo: <http://example.org/geo#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT ?city ?population
WHERE {
  ?state geo:name "Ohio" .
  ?state geo:hasCity ?city .
  ?city geo:population ?population .
}
ORDER BY DESC(?population)
```

#### Complex Queries
```sparql
# Find demographic patterns
PREFIX census: <http://example.org/census#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?county (AVG(?income) as ?avg_income) (COUNT(?block) as ?num_blocks)
WHERE {
  ?county census:contains ?tract .
  ?tract census:contains ?block .
  ?block census:hasIncome ?income .
  FILTER(?income > 50000)
}
GROUP BY ?county
HAVING (COUNT(?block) > 10)
ORDER BY DESC(?avg_income)
```

---

## Automating Tabular Data Conversion

### Excel to Graph Database

#### Python Script Using pandas and py2neo

```python
import pandas as pd
from py2neo import Graph, Node, Relationship

# Connect to Neo4j
graph = Graph("bolt://localhost:7687", auth=("neo4j", "password"))

# Read Excel file
df = pd.read_excel("census_data.xlsx", sheet_name="Demographics")

# Create nodes and relationships
for index, row in df.iterrows():
    # Create State node
    state = Node("State", 
                 name=row['State'],
                 fips_code=row['State_FIPS'])
    graph.merge(state, "State", "name")
    
    # Create County node
    county = Node("County",
                  name=row['County'],
                  fips_code=row['County_FIPS'],
                  population=row['Population'])
    graph.merge(county, "County", "fips_code")
    
    # Create relationship
    rel = Relationship(state, "CONTAINS", county)
    graph.merge(rel)
    
    print(f"Processed: {row['State']} -> {row['County']}")

print("Data import completed!")
```

### CSV to Graph Database

#### Using LOAD CSV in Cypher

```cypher
// Load census tract data from CSV
LOAD CSV WITH HEADERS FROM 'file:///census_tracts.csv' AS row
MERGE (state:State {fips: row.state_fips, name: row.state_name})
MERGE (county:County {fips: row.county_fips, name: row.county_name})
MERGE (tract:CensusTract {
  id: row.tract_id,
  population: toInteger(row.population),
  median_income: toInteger(row.median_income),
  urban: toBoolean(row.is_urban)
})
MERGE (state)-[:CONTAINS]->(county)
MERGE (county)-[:CONTAINS]->(tract);

// Load weather data from CSV
LOAD CSV WITH HEADERS FROM 'file:///weather_data.csv' AS row
MERGE (station:WeatherStation {id: row.station_id})
ON CREATE SET station.name = row.station_name,
              station.latitude = toFloat(row.latitude),
              station.longitude = toFloat(row.longitude)
MERGE (temp:Temperature {
  date: date(row.date),
  station_id: row.station_id
})
SET temp.value = toFloat(row.temperature),
    temp.unit = row.unit
MERGE (station)-[:RECORDED]->(temp);
```

### Census Data Example

Complete pipeline for census data:

```python
import pandas as pd
from py2neo import Graph, Node, Relationship
from typing import Dict, List

class CensusDataImporter:
    def __init__(self, graph_uri: str, auth: tuple):
        self.graph = Graph(graph_uri, auth=auth)
        
    def import_from_excel(self, filepath: str):
        """Import census data from Excel file"""
        # Read all sheets
        sheets = pd.read_excel(filepath, sheet_name=None)
        
        # Process demographics sheet
        if 'Demographics' in sheets:
            self._process_demographics(sheets['Demographics'])
        
        # Process income sheet
        if 'Income' in sheets:
            self._process_income(sheets['Income'])
            
        # Process geographic boundaries
        if 'Geography' in sheets:
            self._process_geography(sheets['Geography'])
    
    def _process_demographics(self, df: pd.DataFrame):
        """Process demographic data"""
        for _, row in df.iterrows():
            block = Node("CensusBlock",
                        id=row['block_id'],
                        tract_id=row['tract_id'])
            self.graph.merge(block, "CensusBlock", "id")
            
            demo = Node("Demographic",
                       total_population=int(row['total_pop']),
                       male=int(row['male']),
                       female=int(row['female']),
                       median_age=float(row['median_age']),
                       year=int(row['year']))
            self.graph.merge(demo)
            
            rel = Relationship(block, "HAS_DEMOGRAPHIC", demo,
                             year=int(row['year']))
            self.graph.merge(rel)
    
    def _process_income(self, df: pd.DataFrame):
        """Process income data"""
        for _, row in df.iterrows():
            block = self.graph.nodes.match("CensusBlock", 
                                          id=row['block_id']).first()
            if block:
                income = Node("HouseholdIncome",
                            median=int(row['median_income']),
                            mean=int(row['mean_income']),
                            year=int(row['year']))
                self.graph.merge(income)
                
                rel = Relationship(block, "HAS_INCOME_DATA", income,
                                 year=int(row['year']))
                self.graph.merge(rel)
    
    def _process_geography(self, df: pd.DataFrame):
        """Process geographic relationships"""
        for _, row in df.iterrows():
            state = Node("State", 
                        name=row['state'],
                        fips=row['state_fips'])
            self.graph.merge(state, "State", "fips")
            
            county = Node("County",
                         name=row['county'],
                         fips=row['county_fips'])
            self.graph.merge(county, "County", "fips")
            
            tract = Node("CensusTract",
                        id=row['tract_id'],
                        urban=bool(row['is_urban']))
            self.graph.merge(tract, "CensusTract", "id")
            
            # Create relationships
            self.graph.merge(Relationship(state, "CONTAINS", county))
            self.graph.merge(Relationship(county, "CONTAINS", tract))

# Usage
importer = CensusDataImporter("bolt://localhost:7687", 
                             ("neo4j", "password"))
importer.import_from_excel("census_2020_data.xlsx")
```

### Weather Data Example

Pipeline for weather station data:

```python
import pandas as pd
from py2neo import Graph, Node, Relationship
from datetime import datetime

class WeatherDataImporter:
    def __init__(self, graph_uri: str, auth: tuple):
        self.graph = Graph(graph_uri, auth=auth)
    
    def import_from_csv(self, filepath: str):
        """Import weather data from CSV"""
        df = pd.read_csv(filepath)
        
        for _, row in df.iterrows():
            # Create or merge weather station
            station = Node("WeatherStation",
                          id=row['station_id'],
                          name=row['station_name'],
                          latitude=float(row['latitude']),
                          longitude=float(row['longitude']))
            self.graph.merge(station, "WeatherStation", "id")
            
            # Link to city if provided
            if pd.notna(row['city']):
                city = Node("City", name=row['city'])
                self.graph.merge(city, "City", "name")
                self.graph.merge(Relationship(city, "HAS_STATION", station))
            
            # Create temperature reading
            temp = Node("Temperature",
                       date=row['date'],
                       station_id=row['station_id'],
                       value=float(row['temperature']),
                       unit=row['unit'])
            self.graph.create(temp)
            
            # Create relationship
            rel = Relationship(station, "RECORDED", temp,
                             date=row['date'])
            self.graph.create(rel)
            
            # Create precipitation if available
            if pd.notna(row['precipitation']):
                precip = Node("Precipitation",
                            date=row['date'],
                            station_id=row['station_id'],
                            value=float(row['precipitation']),
                            unit="inches")
                self.graph.create(precip)
                
                rel = Relationship(station, "RECORDED", precip,
                                 date=row['date'])
                self.graph.create(rel)

# Usage
importer = WeatherDataImporter("bolt://localhost:7687",
                               ("neo4j", "password"))
importer.import_from_csv("weather_readings_2023.csv")
```

#### Batch Processing for Large Datasets

```python
from py2neo import Graph
import pandas as pd

def batch_import(graph: Graph, df: pd.DataFrame, batch_size: int = 1000):
    """Import data in batches for better performance"""
    total_rows = len(df)
    
    for i in range(0, total_rows, batch_size):
        batch = df.iloc[i:i+batch_size]
        
        # Build Cypher query for batch
        query = """
        UNWIND $rows AS row
        MERGE (s:WeatherStation {id: row.station_id})
        SET s.name = row.station_name,
            s.latitude = toFloat(row.latitude),
            s.longitude = toFloat(row.longitude)
        CREATE (t:Temperature {
          date: row.date,
          value: toFloat(row.temperature),
          unit: row.unit
        })
        CREATE (s)-[:RECORDED {date: row.date}]->(t)
        """
        
        # Convert batch to list of dicts
        rows = batch.to_dict('records')
        
        # Execute batch
        graph.run(query, rows=rows)
        
        print(f"Processed {min(i+batch_size, total_rows)}/{total_rows} rows")

# Usage
graph = Graph("bolt://localhost:7687", auth=("neo4j", "password"))
df = pd.read_csv("large_weather_dataset.csv")
batch_import(graph, df, batch_size=5000)
```

---

## Exposing APIs for Non-Technical Users

### REST API Approach

Using Flask and py2neo to create a REST API:

```python
from flask import Flask, request, jsonify
from py2neo import Graph
from typing import Dict, List, Any

app = Flask(__name__)
graph = Graph("bolt://localhost:7687", auth=("neo4j", "password"))

@app.route('/api/cities', methods=['GET'])
def get_cities():
    """Get all cities with population data"""
    query = """
    MATCH (c:City)
    RETURN c.name as name, c.population as population, c.state as state
    ORDER BY c.population DESC
    """
    results = graph.run(query).data()
    return jsonify(results)

@app.route('/api/city/<city_name>/weather', methods=['GET'])
def get_city_weather(city_name: str):
    """Get weather data for a specific city"""
    start_date = request.args.get('start_date', '2023-01-01')
    end_date = request.args.get('end_date', '2023-12-31')
    
    query = """
    MATCH (c:City {name: $city})-[:HAS_STATION]->(ws:WeatherStation)
          -[:RECORDED]->(t:Temperature)
    WHERE t.date >= date($start) AND t.date <= date($end)
    RETURN t.date as date, 
           avg(t.value) as avg_temp,
           min(t.value) as min_temp,
           max(t.value) as max_temp
    ORDER BY t.date
    """
    
    results = graph.run(query, 
                       city=city_name,
                       start=start_date,
                       end=end_date).data()
    
    return jsonify({
        'city': city_name,
        'start_date': start_date,
        'end_date': end_date,
        'data': results
    })

@app.route('/api/census/search', methods=['GET'])
def search_census_data():
    """Search census data with filters"""
    state = request.args.get('state')
    min_income = request.args.get('min_income', type=int, default=0)
    min_population = request.args.get('min_population', type=int, default=0)
    
    query = """
    MATCH (s:State)-[:CONTAINS]->(county:County)-[:CONTAINS]->(tract:CensusTract)
          -[:CONTAINS]->(block:CensusBlock)
    MATCH (block)-[:HAS_INCOME_DATA]->(income:HouseholdIncome)
    MATCH (block)-[:HAS_DEMOGRAPHIC]->(demo:Demographic)
    WHERE s.name = $state
      AND income.median >= $min_income
      AND demo.total_population >= $min_population
    RETURN county.name as county,
           tract.id as tract_id,
           block.id as block_id,
           income.median as median_income,
           demo.total_population as population
    ORDER BY income.median DESC
    LIMIT 100
    """
    
    results = graph.run(query,
                       state=state,
                       min_income=min_income,
                       min_population=min_population).data()
    
    return jsonify({
        'state': state,
        'filters': {
            'min_income': min_income,
            'min_population': min_population
        },
        'results': results,
        'count': len(results)
    })

@app.route('/api/correlate/weather-income', methods=['GET'])
def correlate_weather_income():
    """Correlate weather patterns with income levels"""
    query = """
    MATCH (city:City)-[:HAS_STATION]->(ws:WeatherStation)-[:RECORDED]->(t:Temperature)
    MATCH (city)<-[:LIVES_IN]-(cb:CensusBlock)-[:HAS_INCOME_DATA]->(income:HouseholdIncome)
    WITH city.name as city, 
         avg(t.value) as avg_temp,
         avg(income.median) as avg_income
    RETURN city, avg_temp, avg_income
    ORDER BY avg_income DESC
    """
    
    results = graph.run(query).data()
    return jsonify(results)

@app.route('/api/stats/summary', methods=['GET'])
def get_summary_stats():
    """Get summary statistics"""
    queries = {
        'total_cities': "MATCH (c:City) RETURN count(c) as count",
        'total_weather_stations': "MATCH (ws:WeatherStation) RETURN count(ws) as count",
        'total_census_blocks': "MATCH (cb:CensusBlock) RETURN count(cb) as count",
        'total_temperature_readings': "MATCH (t:Temperature) RETURN count(t) as count"
    }
    
    stats = {}
    for key, query in queries.items():
        result = graph.run(query).data()
        stats[key] = result[0]['count'] if result else 0
    
    return jsonify(stats)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

#### API Documentation Example

```yaml
# OpenAPI/Swagger documentation
openapi: 3.0.0
info:
  title: Census and Weather Graph Database API
  version: 1.0.0
  description: API for querying census and weather data from a graph database

paths:
  /api/cities:
    get:
      summary: Get all cities
      responses:
        '200':
          description: List of cities with population data
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    name:
                      type: string
                    population:
                      type: integer
                    state:
                      type: string

  /api/city/{city_name}/weather:
    get:
      summary: Get weather data for a city
      parameters:
        - name: city_name
          in: path
          required: true
          schema:
            type: string
        - name: start_date
          in: query
          schema:
            type: string
            format: date
        - name: end_date
          in: query
          schema:
            type: string
            format: date
      responses:
        '200':
          description: Weather data for the specified city and date range
```

### GraphQL API Approach

Using graphene and py2neo:

```python
import graphene
from graphene import ObjectType, String, Int, Float, List, Field
from py2neo import Graph

graph = Graph("bolt://localhost:7687", auth=("neo4j", "password"))

# Define GraphQL types
class City(ObjectType):
    name = String()
    population = Int()
    state = String()
    
class WeatherReading(ObjectType):
    date = String()
    temperature = Float()
    station_id = String()
    
class CensusBlock(ObjectType):
    id = String()
    population = Int()
    median_income = Int()
    tract_id = String()

# Define queries
class Query(ObjectType):
    cities = List(City, state=String())
    weather = List(WeatherReading, 
                   city=String(required=True),
                   start_date=String(),
                   end_date=String())
    census_blocks = List(CensusBlock,
                        state=String(),
                        min_income=Int())
    
    def resolve_cities(self, info, state=None):
        query = "MATCH (c:City) "
        if state:
            query += "WHERE c.state = $state "
        query += "RETURN c.name as name, c.population as population, c.state as state"
        
        results = graph.run(query, state=state).data()
        return [City(**r) for r in results]
    
    def resolve_weather(self, info, city, start_date=None, end_date=None):
        query = """
        MATCH (c:City {name: $city})-[:HAS_STATION]->(ws:WeatherStation)
              -[:RECORDED]->(t:Temperature)
        """
        if start_date and end_date:
            query += "WHERE t.date >= date($start) AND t.date <= date($end) "
        query += "RETURN t.date as date, t.value as temperature, ws.id as station_id"
        
        results = graph.run(query, 
                          city=city,
                          start=start_date,
                          end=end_date).data()
        return [WeatherReading(**r) for r in results]
    
    def resolve_census_blocks(self, info, state=None, min_income=None):
        query = """
        MATCH (block:CensusBlock)-[:HAS_INCOME_DATA]->(income:HouseholdIncome)
        MATCH (block)-[:HAS_DEMOGRAPHIC]->(demo:Demographic)
        """
        conditions = []
        if state:
            query += "MATCH (s:State {name: $state})-[:CONTAINS*]->(block) "
        if min_income:
            conditions.append("income.median >= $min_income")
        
        if conditions:
            query += "WHERE " + " AND ".join(conditions) + " "
        
        query += """
        RETURN block.id as id,
               demo.total_population as population,
               income.median as median_income,
               block.tract_id as tract_id
        """
        
        results = graph.run(query, state=state, min_income=min_income).data()
        return [CensusBlock(**r) for r in results]

# Create schema
schema = graphene.Schema(query=Query)

# Example usage with Flask
from flask import Flask
from flask_graphql import GraphQLView

app = Flask(__name__)
app.add_url_rule('/graphql',
                 view_func=GraphQLView.as_view('graphql',
                                              schema=schema,
                                              graphiql=True))

if __name__ == '__main__':
    app.run(debug=True)
```

#### Example GraphQL Queries

```graphql
# Get all cities in Ohio
query {
  cities(state: "Ohio") {
    name
    population
  }
}

# Get weather data for Columbus
query {
  weather(city: "Columbus", startDate: "2023-01-01", endDate: "2023-12-31") {
    date
    temperature
    stationId
  }
}

# Get high-income census blocks
query {
  censusBlocks(state: "Ohio", minIncome: 75000) {
    id
    population
    medianIncome
    tractId
  }
}

# Complex query combining multiple data sources
query {
  cities(state: "Ohio") {
    name
    population
  }
  censusBlocks(state: "Ohio", minIncome: 60000) {
    id
    medianIncome
  }
}
```

---

## Practical Examples

### Census Data Knowledge Graph

Complete example of building a census data knowledge graph:

#### Data Model
```
(State)-[:CONTAINS]->(County)-[:CONTAINS]->(CensusTract)-[:CONTAINS]->(CensusBlock)
(CensusBlock)-[:HAS_DEMOGRAPHIC]->(Demographic)
(CensusBlock)-[:HAS_INCOME_DATA]->(HouseholdIncome)
(CensusBlock)-[:HAS_HOUSING_DATA]->(Housing)
(CensusBlock)-[:HAS_EDUCATION_DATA]->(Education)
```

#### Sample Data Creation
```cypher
// Create geographic hierarchy
CREATE (ohio:State {name: "Ohio", fips: "39", population: 11799448})
CREATE (franklin:County {name: "Franklin", fips: "39049", population: 1323807})
CREATE (tract1:CensusTract {id: "39049001100", urban: true})
CREATE (block1:CensusBlock {id: "390490011001", land_area: 0.25})

// Create demographic data
CREATE (demo1:Demographic {
  year: 2020,
  total_population: 1245,
  male: 598,
  female: 647,
  median_age: 34.5,
  under_18: 287,
  over_65: 142
})

// Create income data
CREATE (income1:HouseholdIncome {
  year: 2020,
  median: 67500,
  mean: 78200,
  below_poverty: 89,
  households: 512
})

// Create relationships
CREATE (ohio)-[:CONTAINS]->(franklin)
CREATE (franklin)-[:CONTAINS]->(tract1)
CREATE (tract1)-[:CONTAINS]->(block1)
CREATE (block1)-[:HAS_DEMOGRAPHIC]->(demo1)
CREATE (block1)-[:HAS_INCOME_DATA]->(income1)
```

#### Analysis Queries
```cypher
// Find income inequality within a county
MATCH (county:County {name: "Franklin"})-[:CONTAINS*]->(block:CensusBlock)
      -[:HAS_INCOME_DATA]->(income:HouseholdIncome)
WHERE income.year = 2020
WITH county,
     max(income.median) as max_income,
     min(income.median) as min_income,
     avg(income.median) as avg_income,
     stdev(income.median) as std_dev
RETURN county.name,
       max_income,
       min_income,
       avg_income,
       std_dev,
       (max_income - min_income) as income_range

// Population density by tract
MATCH (tract:CensusTract)-[:CONTAINS]->(block:CensusBlock)
      -[:HAS_DEMOGRAPHIC]->(demo:Demographic)
WHERE demo.year = 2020
WITH tract,
     sum(demo.total_population) as total_pop,
     sum(block.land_area) as total_area
RETURN tract.id,
       total_pop,
       total_area,
       (total_pop / total_area) as population_density
ORDER BY population_density DESC
LIMIT 10

// Age distribution by urban/rural classification
MATCH (tract:CensusTract)-[:CONTAINS]->(block:CensusBlock)
      -[:HAS_DEMOGRAPHIC]->(demo:Demographic)
WHERE demo.year = 2020
RETURN tract.urban as is_urban,
       sum(demo.under_18) as youth,
       sum(demo.total_population - demo.under_18 - demo.over_65) as working_age,
       sum(demo.over_65) as seniors,
       sum(demo.total_population) as total
```

### Weather Data Knowledge Graph

Complete example of building a weather data knowledge graph:

#### Data Model
```
(State)-[:HAS_CITY]->(City)-[:HAS_STATION]->(WeatherStation)
(WeatherStation)-[:RECORDED]->(Temperature)
(WeatherStation)-[:RECORDED]->(Precipitation)
(WeatherStation)-[:RECORDED]->(WindSpeed)
(City)-[:NEIGHBOR_OF]->(City)
```

#### Sample Data Creation
```cypher
// Create weather stations
CREATE (kolu:WeatherStation {
  id: "KOLU",
  name: "Columbus Airport",
  latitude: 40.0,
  longitude: -82.89,
  elevation: 815
})

CREATE (kcmh:WeatherStation {
  id: "KCMH",
  name: "Port Columbus International",
  latitude: 39.998,
  longitude: -82.892,
  elevation: 812
})

// Create temperature readings
CREATE (t1:Temperature {
  date: date('2023-06-15'),
  value: 75.5,
  unit: "F",
  time_of_day: "afternoon"
})

CREATE (t2:Temperature {
  date: date('2023-06-16'),
  value: 78.2,
  unit: "F",
  time_of_day: "afternoon"
})

// Create precipitation readings
CREATE (p1:Precipitation {
  date: date('2023-06-15'),
  value: 0.25,
  unit: "inches",
  type: "rain"
})

// Create relationships
CREATE (columbus:City {name: "Columbus", state: "Ohio"})
CREATE (columbus)-[:HAS_STATION]->(kolu)
CREATE (columbus)-[:HAS_STATION]->(kcmh)
CREATE (kolu)-[:RECORDED {date: date('2023-06-15')}]->(t1)
CREATE (kolu)-[:RECORDED {date: date('2023-06-16')}]->(t2)
CREATE (kolu)-[:RECORDED {date: date('2023-06-15')}]->(p1)
```

#### Analysis Queries
```cypher
// Monthly temperature averages
MATCH (ws:WeatherStation)-[:RECORDED]->(t:Temperature)
WHERE t.date >= date('2023-01-01') AND t.date <= date('2023-12-31')
WITH ws, 
     t.date.year as year,
     t.date.month as month,
     avg(t.value) as avg_temp
RETURN ws.name, year, month, avg_temp
ORDER BY year, month

// Rainy days by station
MATCH (ws:WeatherStation)-[:RECORDED]->(p:Precipitation)
WHERE p.value > 0 AND p.date >= date('2023-01-01')
WITH ws, count(p) as rainy_days, sum(p.value) as total_precip
RETURN ws.name, rainy_days, total_precip
ORDER BY rainy_days DESC

// Temperature correlation between nearby stations
MATCH (ws1:WeatherStation)-[:RECORDED]->(t1:Temperature),
      (ws2:WeatherStation)-[:RECORDED]->(t2:Temperature)
WHERE ws1.id < ws2.id
  AND t1.date = t2.date
  AND t1.time_of_day = t2.time_of_day
WITH ws1, ws2, 
     count(*) as readings,
     avg(abs(t1.value - t2.value)) as avg_diff
WHERE readings > 100
RETURN ws1.name, ws2.name, readings, avg_diff
ORDER BY avg_diff

// Extreme weather events
MATCH (ws:WeatherStation)-[:RECORDED]->(t:Temperature)
WHERE t.value > 95 OR t.value < 10
MATCH (ws)-[:RECORDED]->(p:Precipitation)
WHERE p.date = t.date AND p.value > 1.0
RETURN t.date, ws.name, t.value as temp, p.value as precip
ORDER BY t.date
```

#### Combining Census and Weather Data
```cypher
// Correlate high temperatures with population density
MATCH (city:City)-[:HAS_STATION]->(ws:WeatherStation)-[:RECORDED]->(t:Temperature)
MATCH (city)<-[:IN_CITY]-(cb:CensusBlock)-[:HAS_DEMOGRAPHIC]->(demo:Demographic)
WHERE t.date >= date('2023-06-01') AND t.date <= date('2023-08-31')
WITH city,
     avg(t.value) as avg_summer_temp,
     sum(demo.total_population) / sum(cb.land_area) as pop_density
RETURN city.name, avg_summer_temp, pop_density
ORDER BY avg_summer_temp DESC

// Income levels and weather station proximity
MATCH (ws:WeatherStation)<-[:NEAR {distance: "<5km"}]-(cb:CensusBlock)
      -[:HAS_INCOME_DATA]->(income:HouseholdIncome)
MATCH (ws)-[:RECORDED]->(t:Temperature)
WHERE t.date >= date('2023-01-01')
WITH cb, income, avg(t.value) as avg_temp
RETURN avg_temp, avg(income.median) as avg_income
```

---

## Best Practices

### 1. Data Modeling
- **Start simple**: Begin with core entities and relationships, add complexity as needed
- **Use meaningful labels**: Choose descriptive names for nodes and relationships
- **Normalize appropriately**: Balance between denormalization for performance and normalization for maintainability
- **Index frequently queried properties**: Create indexes on properties used in WHERE clauses

### 2. Performance Optimization
- **Use MERGE wisely**: MERGE can be expensive; use CREATE when you know data doesn't exist
- **Batch operations**: Process large imports in batches (1000-10000 rows)
- **Profile queries**: Use PROFILE and EXPLAIN to understand query performance
- **Limit early**: Apply LIMIT clauses early in query processing when possible

### 3. Query Best Practices
```cypher
// Good: Use parameters
MATCH (p:Person {id: $person_id})
RETURN p

// Bad: String concatenation (SQL injection risk)
// MATCH (p:Person {id: '" + userId + "'}) RETURN p

// Good: Specific pattern matching
MATCH (p:Person)-[:LIVES_IN]->(c:City {name: "Columbus"})
RETURN p

// Bad: Vague pattern
MATCH (p:Person)-[r]->(c)
WHERE c.name = "Columbus"
RETURN p
```

### 4. Data Quality
- **Validate on import**: Check data types and required fields
- **Use constraints**: Define uniqueness and existence constraints
- **Monitor data growth**: Track node and relationship counts
- **Regular backups**: Implement automated backup strategies

### 5. Security
- **Use authentication**: Always enable authentication in production
- **Principle of least privilege**: Create role-based access control
- **Encrypt connections**: Use SSL/TLS for network communication
- **Sanitize inputs**: Prevent injection attacks in API layer

### 6. Documentation
- **Document your schema**: Maintain clear documentation of node labels and relationship types
- **Version your model**: Track changes to your graph schema over time
- **Example queries**: Provide sample queries for common use cases
- **API documentation**: Use OpenAPI/Swagger for REST APIs

---

## Resources

### Graph Databases
- **Neo4j**: https://neo4j.com/
  - Documentation: https://neo4j.com/docs/
  - Cypher Manual: https://neo4j.com/docs/cypher-manual/
  - Online Sandbox: https://neo4j.com/sandbox/
  
- **Amazon Neptune**: https://aws.amazon.com/neptune/
  - Documentation: https://docs.aws.amazon.com/neptune/
  
- **Apache Jena**: https://jena.apache.org/
  - Documentation: https://jena.apache.org/documentation/

### Query Languages
- **Cypher**: https://opencypher.org/
- **SPARQL**: https://www.w3.org/TR/sparql11-query/
- **Gremlin**: https://tinkerpop.apache.org/gremlin.html

### Python Libraries
- **py2neo**: https://py2neo.org/
- **neo4j-driver**: https://neo4j.com/docs/python-manual/
- **rdflib**: https://rdflib.readthedocs.io/

### Data Sources
- **US Census Bureau**: https://www.census.gov/data.html
  - API: https://www.census.gov/data/developers.html
- **NOAA Weather Data**: https://www.ncdc.noaa.gov/
  - API: https://www.ncdc.noaa.gov/cdo-web/webservices/v2

### Learning Resources
- **Graph Academy (Neo4j)**: https://graphacademy.neo4j.com/
- **Knowledge Graphs Book**: "Knowledge Graphs" by Aidan Hogan et al.
- **Graph Databases Book**: "Graph Databases" by Ian Robinson, Jim Webber, and Emil Eifrem

### Community
- **Neo4j Community Forum**: https://community.neo4j.com/
- **Stack Overflow**: Tag [neo4j], [graph-databases], [sparql]
- **GitHub Topics**: #knowledge-graph, #graph-database

---

## Getting Started

To get started with your own knowledge graph:

1. **Install Neo4j** (recommended for beginners):
   ```bash
   # Using Docker
   docker run -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/password neo4j:latest
   ```

2. **Install Python dependencies**:
   ```bash
   pip install py2neo pandas openpyxl
   ```

3. **Access Neo4j Browser**: Open http://localhost:7474

4. **Create your first nodes**:
   ```cypher
   CREATE (alice:Person {name: "Alice", age: 30})
   CREATE (bob:Person {name: "Bob", age: 35})
   CREATE (alice)-[:KNOWS {since: 2020}]->(bob)
   ```

5. **Query your data**:
   ```cypher
   MATCH (p:Person)-[:KNOWS]->(friend)
   RETURN p.name, friend.name
   ```

Now you're ready to build your own knowledge graphs with census data, weather data, or any other domain!

---

## Contributing

Contributions to improve this guide are welcome! Please submit issues or pull requests on GitHub.

## License

This project is licensed under the MIT License - see the LICENSE file for details.