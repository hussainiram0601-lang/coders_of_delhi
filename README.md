# CodeBook: Pure Python Social Network Data Pipeline & Recommendation Engine

A lightweight, dependency-free Python data pipeline for loading, cleaning, and processing structured user connection data from **CodeBook** (a social network platform for coders). Features an algorithmic recommendation system for suggesting connections based on mutual friend frequency.

---

## 📌 Project Overview

This project demonstrates how to handle raw data dumps, perform structured data validation, and build core graph-based recommendation systems using **Pure Python** (built-in modules only—no heavy external dependencies like Pandas or NumPy).

### Core Features:
- 📥 **JSON Data Ingestion:** Safely load user profiles, friend connections, and page metadata.
- 🧹 **Data Cleaning & Deduplication:** 
  - Filters out incomplete user profiles (missing names).
  - Deduplicates nested lists (friend IDs).
  - Removes inactive accounts (no friends and no liked pages).
  - Deduplicates page entries by primary ID keys.
- 🤝 **"People You May Know" Recommendation Algorithm:** Evaluates second-degree connections (friends-of-friends) and ranks non-friend recommendations based on mutual connection count.

---

## 📂 Repository Structure

```
.
├── notebook.ipynb            # Interactive notebook containing pipeline code and tasks
├── codersofdelhi.json        # Raw JSON data dump of user and page entities
├── cleaned_codebook_data.json# Formatted & clean JSON output after validation
└── README.md                 # Project documentation
```

---

## 🛠️ Data Pipeline & Logic

### 1. Data Cleaning (`clean_data`)
The cleaning pipeline resolves inconsistencies commonly found in raw JSON exports:
```python
# 1. Missing value handling
data["users"] = [u for u in data["users"] if u["name"].strip()]

# 2. Friend list deduplication
for u in data["users"]:
    u["friends"] = list(set(u["friends"]))

# 3. Inactive account purging
data["users"] = [u for u in data["users"] if u["friends"] or u["liked_pages"]]

# 4. Page deduplication by ID key
unique_pages = {page["id"]: page for page in data["pages"]}
data["pages"] = list(unique_pages.values())
```

### 2. Recommendation Algorithm (`people_you_may_know`)
The engine discovers candidates using a graph traversal logic:
1. Maps direct connections for a target `user_id`.
2. Traverses all second-degree connections (friends of direct friends).
3. Excludes the primary user and existing direct friends.
4. Ranks candidates dynamically based on mutual connection frequency.

---

## 🚀 Getting Started

### Prerequisites
- **Python 3.8+** (Uses standard libraries `json`)

### Running the Project
1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/codebook-recommendation-engine.git
   cd codebook-recommendation-engine
   ```

2. Run the notebook:
   ```bash
   jupyter notebook notebook.ipynb
   ```

---

## 📊 Sample Input & Output

### Input Format (`codersofdelhi.json`):
```json
{
    "users": [
        {"id": 1, "name": "Amit", "friends": [2, 3], "liked_pages": [101]},
        {"id": 2, "name": "Priya", "friends": [1, 4], "liked_pages": [102]},
        {"id": 3, "name": "Rahul", "friends": [1], "liked_pages": [101, 103]},
        {"id": 4, "name": "Sara", "friends": [2], "liked_pages": [104]}
    ],
    "pages": [
        {"id": 101, "name": "Python Developers"}
    ]
}
```

### Recommendation Output:
```python
user_id = 1
# User 1 (Amit) is friends with 2 (Priya) and 3 (Rahul).
# User 2 (Priya) is friends with 4 (Sara).
# System identifies User 4 as a 2nd-degree connection.

recommendations = people_you_may_know(1, data)
# Output: [4]
```
