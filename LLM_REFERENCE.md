# Twenty CRM API - LLM Reference Guide

> **Purpose:** Complete technical reference for AI agents automating Twenty CRM
> **Last Updated:** 2025-10-25
> **Status:** Production-ready, tested, verified

---

## CRITICAL FACTS

### What Works (100% Verified)
- ✅ Create Objects: GraphQL `/metadata` endpoint, `createOneObject` mutation
- ✅ Create Fields: GraphQL `/metadata` OR REST `/rest/metadata/fields`
- ✅ Create Relations: REST `/rest/metadata/fields` with `relationCreationPayload`
- ✅ Create Views: GraphQL `/metadata` endpoint, `createCoreView` mutation
- ✅ Data CRUD: GraphQL `/graphql` endpoint
- ✅ Search & Filter: GraphQL `/graphql` with advanced operators
- ✅ All operations work with API Keys (Bearer token)

### Authentication
```python
HEADERS = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
```

API Key format: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
Get from: Settings → APIs & Webhooks

---

## ENDPOINTS

| Endpoint | Method | Purpose | Auth |
|----------|--------|---------|------|
| `/metadata` | POST | GraphQL metadata operations (objects, fields, views) | API Key ✅ |
| `/rest/metadata/objects` | GET | List all objects | API Key ✅ |
| `/rest/metadata/fields` | POST | Create field | API Key ✅ |
| `/rest/metadata/fields/{id}` | PATCH | Update field | API Key ✅ |
| `/rest/metadata/fields/{id}` | DELETE | Delete field | API Key ✅ |
| `/graphql` | POST | Data operations (CRUD, Search, Filter) | API Key ✅ |

---

## TABLE OF CONTENTS

1. **OBJECTS** - Create and manage object schemas
2. **FIELDS** - Add fields to objects
3. **RELATIONS** - Link objects together
4. **VIEWS** - Create different data views
5. **DATA OPERATIONS** - CRUD operations on records
6. **SEARCH & FILTERING** - Query and filter data
7. **BATCH OPERATIONS** - Bulk create/update
8. **COMPLETE WORKFLOWS** - End-to-end examples

---

## OBJECTS

### Create Object (GraphQL)
```python
mutation = """
mutation {
  createOneObject(input: {
    object: {
      nameSingular: "task"
      namePlural: "tasks"
      labelSingular: "Task"
      labelPlural: "Tasks"
      description: "Task management"
      icon: "IconCheckbox"
    }
  }) {
    id
    nameSingular
    namePlural
    isCustom
  }
}
"""
requests.post(f"{API_URL}/metadata", headers=HEADERS, json={"query": mutation})
```

### Naming Rules
- `nameSingular`: camelCase (e.g., `task`, `projectMilestone`)
- `namePlural`: camelCase (e.g., `tasks`, `projectMilestones`)
- `labelSingular`: Title Case (e.g., `Task`, `Project Milestone`)
- `labelPlural`: Title Case (e.g., `Tasks`, `Project Milestones`)

### Get Object ID
```python
response = requests.get(f"{API_URL}/rest/metadata/objects", headers=HEADERS)
objects = response.json()["data"]["objects"]
obj = next((o for o in objects if o["nameSingular"] == "project"), None)
object_id = obj["id"]
```

---

## FIELDS

### Field Types
| Type | Use | Options Required | Example |
|------|-----|------------------|---------|  
| TEXT | Single line text | No | Name, Email |
| RICH_TEXT | Multi-line formatted | No | Description |
| NUMBER | Numeric values | No | Hours, Amount |
| DATE | Date only | No | Deadline |
| DATE_TIME | Date + time | No | Created At |
| BOOLEAN | True/False | No | Is Active |
| SELECT | Single choice | **Yes** | Status, Priority |
| MULTI_SELECT | Multiple choice | **Yes** | Tags, Tech Stack |
| LINKS | URLs | No | Repository URL |
| RELATION | Link to object | **Special** | Project → Client |

### Create Field (REST API)
```python
field_data = {
    "name": "projectName",
    "label": "Project Name",
    "type": "TEXT",
    "icon": "IconTag",
    "objectMetadataId": object_id
}
requests.post(f"{API_URL}/rest/metadata/fields", headers=HEADERS, json=field_data)
```

### Create SELECT Field
```python
field_data = {
    "name": "status",
    "label": "Status",
    "type": "SELECT",
    "icon": "IconFlag",
    "objectMetadataId": object_id,
    "options": [
        {"label": "To Do", "value": "TODO", "color": "blue", "position": 0},
        {"label": "In Progress", "value": "IN_PROGRESS", "color": "yellow", "position": 1},
        {"label": "Done", "value": "DONE", "color": "green", "position": 2}
    ]
}
```

### Available Colors
`blue`, `green`, `yellow`, `orange`, `red`, `purple`, `pink`, `gray`, `teal`

---

## RELATIONS

### Create Relation (REST API)
```python
field_data = {
    "name": "projectRelation",
    "label": "Project",
    "type": "RELATION",
    "icon": "IconLink",
    "objectMetadataId": from_object_id,
    "relationCreationPayload": {
        "type": "MANY_TO_ONE",
        "targetObjectMetadataId": to_object_id,
        "targetFieldLabel": "Features",  # Reverse relation label (plural)
        "targetFieldIcon": "IconLink"
    }
}
requests.post(f"{API_URL}/rest/metadata/fields", headers=HEADERS, json=field_data)
```

### Relation Types
- `MANY_TO_ONE`: Many Features → One Project
- `ONE_TO_MANY`: Automatically created as reverse

### Important
- Forward relation: singular label (e.g., "Project")
- Reverse relation: plural label (e.g., "Features")
- Unique field names required

---

## VIEWS

### Create View (GraphQL)
```python
mutation = """
mutation {
  createCoreView(input: {
    objectMetadataId: "%s"
    name: "All Projects"
    type: "table"
    icon: "IconTable"
  }) {
    id
    name
    type
  }
}
""" % object_id
requests.post(f"{API_URL}/metadata", headers=HEADERS, json={"query": mutation})
```

### View Types
- `table`: Standard table view (best for detailed data)
- `kanban`: Kanban board (best for workflows)
- `calendar`: Calendar view (best for date-based data)
- `gallery`: Gallery/card view (best for visual data)

---

## DATA OPERATIONS (CRUD)

### Create Record
```python
# Example: Create a person record
mutation = """
mutation {
  createPerson(data: {
    name: {firstName: "John", lastName: "Doe"}
    email: "john@example.com"
    phone: "+1234567890"
  }) {
    id
    name { firstName lastName }
    email
  }
}
"""
response = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
record_id = response.json()["data"]["createPerson"]["id"]
```

### Read Records
```python
# Get all records
query = """
query {
  people {
    edges {
      node {
        id
        name { firstName lastName }
        email
        phone
      }
    }
  }
}
"""
response = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": query})
records = response.json()["data"]["people"]["edges"]

# Get single record by ID
query = 'query { person(id: "%s") { id name { firstName lastName } email } }' % record_id
```

### Update Record
```python
mutation = """
mutation {
  updatePerson(id: "%s", data: {email: "new@example.com", phone: "+9876543210"}) {
    id email phone
  }
}
""" % record_id
response = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
```

### Delete Record
```python
mutation = 'mutation { deletePerson(id: "%s") { id } }' % record_id
response = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
```

### Custom Object CRUD
```python
# For custom objects, use object name (plural, camelCase)
# Example: "project" object

# Create
mutation = 'mutation { createProject(data: {projectName: "Website", status: "ACTIVE"}) { id projectName } }'

# Read all
query = 'query { projects { edges { node { id projectName status } } } }'

# Update
mutation = 'mutation { updateProject(id: "%s", data: {status: "COMPLETED"}) { id status } }' % project_id

# Delete
mutation = 'mutation { deleteProject(id: "%s") { id } }' % project_id
```

---

## SEARCH & FILTERING

### Basic Search
```python
query = """
query {
  people(filter: {name: {firstName: {eq: "John"}}}) {
    edges { node { id name { firstName lastName } email } }
  }
}
"""
```

### Advanced Filtering
```python
# AND condition
query = """
query {
  projects(filter: {
    and: [
      {status: {eq: "ACTIVE"}}
      {deadline: {gte: "2025-01-01"}}
    ]
  }) {
    edges { node { id projectName status deadline } }
  }
}
"""

# OR condition
query = """
query {
  projects(filter: {
    or: [
      {status: {eq: "ACTIVE"}}
      {status: {eq: "IN_PROGRESS"}}
    ]
  }) {
    edges { node { id projectName status } }
  }
}
"""
```

### Filter Operators
| Operator | Description | Example |
|----------|-------------|---------|  
| `eq` | Equals | `{status: {eq: "ACTIVE"}}` |
| `neq` | Not equals | `{status: {neq: "COMPLETED"}}` |
| `gt` | Greater than | `{amount: {gt: 1000}}` |
| `gte` | Greater than or equal | `{deadline: {gte: "2025-01-01"}}` |
| `lt` | Less than | `{amount: {lt: 5000}}` |
| `lte` | Less than or equal | `{deadline: {lte: "2025-12-31"}}` |
| `in` | In array | `{status: {in: ["ACTIVE", "PENDING"]}}` |
| `contains` | Contains text | `{name: {contains: "John"}}` |
| `startsWith` | Starts with | `{email: {startsWith: "john"}}` |
| `endsWith` | Ends with | `{email: {endsWith: "@example.com"}}` |

### Pagination
```python
# First page
query = """
query {
  projects(first: 10) {
    edges { node { id projectName } cursor }
    pageInfo { hasNextPage endCursor }
  }
}
"""
data = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": query}).json()["data"]["projects"]

# Next page
if data["pageInfo"]["hasNextPage"]:
    query = 'query { projects(first: 10, after: "%s") { edges { node { id } } pageInfo { hasNextPage endCursor } } }' % data["pageInfo"]["endCursor"]
```

### Sorting
```python
# Single field
query = 'query { projects(orderBy: {createdAt: DESC}) { edges { node { id projectName createdAt } } } }'

# Multiple fields
query = 'query { projects(orderBy: [{status: ASC}, {deadline: DESC}]) { edges { node { id projectName status deadline } } } }'
```

---

## BATCH OPERATIONS

### Create Multiple Records
```python
records = [
    {"projectName": "Project A", "status": "ACTIVE"},
    {"projectName": "Project B", "status": "PENDING"}
]

for record in records:
    mutation = 'mutation { createProject(data: {projectName: "%s", status: "%s"}) { id } }' % (record["projectName"], record["status"])
    requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
    time.sleep(0.1)
```

### Bulk Update
```python
# Get records
query = 'query { projects(filter: {status: {eq: "PENDING"}}) { edges { node { id } } } }'
records = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": query}).json()["data"]["projects"]["edges"]

# Update each
for record in records:
    mutation = 'mutation { updateProject(id: "%s", data: {status: "ACTIVE"}) { id } }' % record["node"]["id"]
    requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
    time.sleep(0.1)
```

---

## RELATIONS IN DATA

### Create Record with Relation
```python
mutation = 'mutation { createTask(data: {title: "Design", status: "TODO", projectRelationId: "%s"}) { id title projectRelation { id projectName } } }' % project_id
```

### Query with Relations
```python
query = 'query { project(id: "%s") { id projectName tasks { edges { node { id title status } } } } }' % project_id
```

### Update Relation
```python
mutation = 'mutation { updateTask(id: "%s", data: {projectRelationId: "%s"}) { id projectRelation { id projectName } } }' % (task_id, new_project_id)
```

---

## COMPLETE WORKFLOW

```python
#!/usr/bin/env python3
import requests
import time

API_URL = "https://your-instance.com"
API_KEY = "your-api-key"
HEADERS = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

def create_project(name, status):
    mutation = f'mutation {{ createProject(data: {{projectName: "{name}", status: "{status}"}}) {{ id projectName }} }}'
    r = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation})
    return r.json()["data"]["createProject"]["id"]

def search_projects(status):
    query = f'query {{ projects(filter: {{status: {{eq: "{status}"}}}}) {{ edges {{ node {{ id projectName status }} }} }} }}'
    r = requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": query})
    return r.json()["data"]["projects"]["edges"]

def update_status(project_id, new_status):
    mutation = f'mutation {{ updateProject(id: "{project_id}", data: {{status: "{new_status}"}}) {{ id status }} }}'
    return requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation}).json()

def delete_project(project_id):
    mutation = f'mutation {{ deleteProject(id: "{project_id}") {{ id }} }}'
    return requests.post(f"{API_URL}/graphql", headers=HEADERS, json={"query": mutation}).json()

# Usage
project_id = create_project("Website", "ACTIVE")
print(f"✅ Created: {project_id}")

active = search_projects("ACTIVE")
print(f"📊 Found {len(active)} active projects")

update_status(project_id, "COMPLETED")
print("✅ Updated")
```

---

## QUICK REFERENCE

### Setup
```python
import requests
API_URL = "https://your-instance.com"
API_KEY = "your-api-key"
HEADERS = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}
```

### Create Object
```python
mutation = 'mutation { createOneObject(input: {object: {nameSingular: "x", namePlural: "xs", labelSingular: "X", labelPlural: "Xs", icon: "IconTag"}}) { id } }'
requests.post(f"{API_URL}/metadata", headers=HEADERS, json={"query": mutation})
```

### Create Field
```python
requests.post(f"{API_URL}/rest/metadata/fields", headers=HEADERS, json={
    "name": "fieldName", "label": "Label", "type": "TEXT", "icon": "IconTag", "objectMetadataId": obj_id
})
```

### Create View
```python
mutation = 'mutation { createCoreView(input: {objectMetadataId: "%s", name: "All", type: "table", icon: "IconTable"}) { id } }' % obj_id
requests.post(f"{API_URL}/metadata", headers=HEADERS, json={"query": mutation})
```

### CRUD Data
```python
# Create
mutation = 'mutation { createProject(data: {projectName: "X"}) { id } }'

# Read
query = 'query { projects { edges { node { id projectName } } } }'

# Update
mutation = 'mutation { updateProject(id: "%s", data: {projectName: "Y"}) { id } }' % id

# Delete
mutation = 'mutation { deleteProject(id: "%s") { id } }' % id
```

### Search
```python
query = 'query { projects(filter: {status: {eq: "ACTIVE"}}) { edges { node { id projectName } } } }'
```

---

**END OF LLM REFERENCE**

This document contains everything needed to automate Twenty CRM including setup and data management. No additional context required.
