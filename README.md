# Automated-Data-Recovery

Your plan is well-structured! Here's a refined and detailed approach to accomplish the assignment using Google Colab, MongoDB Atlas, and Hugging Face Transformers:

### 1. Setting Up MongoDB Atlas

1. **Create a MongoDB Atlas Account**
   - Visit [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and sign up for an account.

2. **Create a Cluster and Database**
   - Follow the prompts to create a new cluster. Choose the free tier if you're testing.
   - Create a new database within your cluster, e.g., `store_database`.

3. **Whitelist Your IP Address**
   - Go to Network Access in your MongoDB Atlas dashboard and add your IP address.

4. **Create a Database User**
   - Go to Database Access and create a new user with read and write access to your database. Note the username and password.

5. **Get the Connection String**
   - Go to Database Connect, select “Connect your application,” and copy the connection string. Replace the placeholder values with your database user’s credentials.

### 2. Install Required Packages

Run this code in Google Colab to install the necessary libraries:

```bash
!pip install pymongo pandas transformers openai llama-index
```

### 3. Load CSV Data into MongoDB

Use this script to load your CSV data into MongoDB:

```python
import pandas as pd
from pymongo import MongoClient

# MongoDB Atlas connection
client = MongoClient('your_mongodb_connection_string')  # Replace with your actual connection string
db = client['store_database']
collection = db['products']

# Load CSV data into DataFrame
df = pd.read_csv('/path_to_your_file/products.csv')  # Provide the correct path to your CSV file

# Convert DataFrame to dictionary and insert into MongoDB
data = df.to_dict(orient='records')
collection.insert_many(data)

print("Data inserted successfully into MongoDB!")
```

### 4. Hugging Face Model for Query Generation

Load and use a Hugging Face model to generate MongoDB queries:

```python
from transformers import pipeline

# Load the pre-trained model
query_generator = pipeline("text2text-generation", model="google/flan-t5-base")

def generate_mongodb_query(user_input):
    prompt = f"Generate a MongoDB query based on this user input: {user_input}"
    query = query_generator(prompt)[0]['generated_text']
    return query

# Example usage
user_input = "Find all products with a price greater than $50"
generated_query = generate_mongodb_query(user_input)
print("Generated MongoDB Query:", generated_query)
```

### 5. Execute MongoDB Queries

Convert the generated query into a valid MongoDB query and execute it:

```python
def execute_mongodb_query(query):
    try:
        # Use eval to convert the string query to a Python dictionary
        query_dict = eval(query)
        result = collection.find(query_dict)
        return list(result)
    except Exception as e:
        print(f"Error executing query: {e}")
        return []

# Example usage
query = '{"Price": {"$gt": 50}}'  # Example query string
products = execute_mongodb_query(query)

# Convert result to DataFrame for display or saving
result_df = pd.DataFrame(products)
print(result_df)
```

### 6. Present or Save Data

You can save the data to a CSV file or present it:

```python
def save_to_csv(data, filename="output.csv"):
    df = pd.DataFrame(data)
    df.to_csv(filename, index=False)
    print(f"Data saved to {filename}")

# Example usage
save_to_csv(products, "products_over_50.csv")
```

### 7. Error Handling and User Interaction

Handle errors in user input:

```python
def get_user_input():
    try:
        column_name = input("Enter the column name to query:")
        if column_name not in df.columns:
            raise ValueError("Invalid column name!")
        return column_name
    except Exception as e:
        print(f"Error: {e}")
        return None
```

### 8. Test Case Example

Here’s a test case for querying products:

```python
# Define test case query
query = {"$and": [
    {"Rating": {"$lt": 4.5}},
    {"ReviewCount": {"$gt": 200}},
    {"Brand": {"$in": ["Nike", "Sony"]}}
]}
result = execute_mongodb_query(query)
print(result)
save_to_csv(result, "test_case1.csv")
```

### 9. Documentation

- **README.md**
  - **Setup Instructions**: Guide on setting up MongoDB Atlas and connecting it to your script.
  - **Code Explanation**: Detailed explanation of each function and the overall workflow.
  - **Error Handling**: How errors are managed and potential issues users might encounter.

### Code Comments

Ensure your code is well-commented:

```python
# Connect to MongoDB Atlas
client = MongoClient('your_mongodb_connection_string')  # Replace with actual connection string

# Select the database and collection
db = client['store_database']
collection = db['products']

# Load CSV data into DataFrame
df = pd.read_csv('/path_to_your_file/products.csv')  # Provide the path to your CSV file

# Insert data into MongoDB
data = df.to_dict(orient='records')  # Convert DataFrame to dictionary
collection.insert_many(data)  # Insert data into MongoDB collection
```

This setup ensures a smooth workflow from CSV data loading to query execution and data presentation. Let me know if you need any more details or adjustments!
