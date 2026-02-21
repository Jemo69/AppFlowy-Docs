# AppFlowy Cloud API Guide

This guide provides information on how to authenticate, re-authenticate, and perform common data operations using the AppFlowy Cloud REST API.

For a deeper dive into how data is organized, see the [Workspaces and Spaces Guide](WORKSPACES_AND_SPACES.md).

---

## 1. Authentication

AppFlowy uses GoTrue for identity management. You must first obtain a Bearer token to access protected data.

### 1.1 Login (Get Initial Token)
**Endpoint:** `POST https://beta.appflowy.cloud/gotrue/token?grant_type=password`

**Headers:**
- `Content-Type: application/json`

**Body:**
```json
{
  "email": "your_email@example.com",
  "password": "your_password"
}
```

**Example Curl:**
```bash
curl -X POST "https://beta.appflowy.cloud/gotrue/token?grant_type=password" \
     -H "Content-Type: application/json" \
     -d '{"email": "your_email@example.com", "password": "your_password"}'
```

**Response:**
```json
{
  "access_token": "eyJhbG...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "ref_token_123",
  "user": { ... }
}
```

### 1.2 Re-authentication (Refresh Token)
When the `access_token` expires, use the `refresh_token` to get a new one.

**Endpoint:** `POST https://beta.appflowy.cloud/gotrue/token?grant_type=refresh_token`

**Headers:**
- `Content-Type: application/json`

**Body:**
```json
{
  "refresh_token": "your_refresh_token_here"
}
```

---

## 2. Request Structure

For all endpoints under `/api/`, the following headers are required:

- `Authorization: Bearer <your_access_token>`
- `Content-Type: application/json` (for POST/PUT requests)

---

## 3. Workspaces

### 3.1 List All Workspaces
Retrieves a list of workspaces you belong to.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace`

**Example Curl:**
```bash
curl -H "Authorization: Bearer <token>" https://beta.appflowy.cloud/api/workspace
```

---

## 4. Notes, Pages, and Spaces (Folders)

In AppFlowy, the hierarchy is organized into **Workspaces**, which contain **Spaces**, which in turn contain **Pages** (or Notes) and **Databases**.

### 4.1 Identifying Spaces
Spaces are top-level organizational units within a workspace. You can identify them by fetching the workspace folder structure and looking for the `is_space` property.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/folder`

In the response, objects where `"is_space": true` are Spaces. They can contain children which are pages, databases, or sub-folders.

### 4.2 List All Notes/Pages in a Workspace
This endpoint returns the hierarchical folder structure of the workspace.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/folder`

**Query Parameters:**
- `depth` (optional): How deep to go into subfolders (default 1).

**Example Curl:**
```bash
curl -H "Authorization: Bearer <token>" \
     "https://beta.appflowy.cloud/api/workspace/{workspace_id}/folder?depth=2"
```

The response will contain a list of views with their `view_id`, `name`, and any `children` (sub-pages).

---

## 5. Databases

### 5.1 List Databases in a Workspace
**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database`

### 5.2 Get Database Fields
To know what data can be stored in a database, you need to retrieve its fields.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/fields`

---

## 6. Data Operations (Rows)

### 6.1 List Database Row IDs
Retrieves all row identifiers for a database.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row`

### 6.2 Fetch All Data (Row Details)
To get the actual content (cells) of the rows, you use the details endpoint. You can pass multiple IDs.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row/detail`

**Query Parameters:**
- `ids`: Comma-separated list of row UUIDs.
- `with_doc` (optional): Set to `true` to include the markdown document content.

**Example Curl:**
```bash
curl -H "Authorization: Bearer <token>" \
     "https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row/detail?ids=uuid1,uuid2&with_doc=true"
```

### 6.3 Create a New Row
**Endpoint:** `POST https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row`

**Body:**
```json
{
  "cells": {
    "Field_ID_or_Name": "Value",
    "Another_Field": 123
  },
  "document": "Optional Markdown content for the row"
}
```

### 6.4 Upsert a Row
Update an existing row or create it if it doesn't exist.

**Endpoint:** `PUT https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row`

**Body:**
```json
{
  "pre_hash": "optional_string",
  "cells": {
    "Field_ID": "New Value"
  },
  "document": "Updated markdown content"
}
```

---

## Summary of Common Endpoints

| Action | Method | URL |
| :--- | :--- | :--- |
| Login | POST | `/gotrue/token?grant_type=password` |
| Refresh Token | POST | `/gotrue/token?grant_type=refresh_token` |
| List Workspaces | GET | `/api/workspace` |
| List Notes/Pages | GET | `/api/workspace/{workspace_id}/folder` |
| List Databases | GET | `/api/workspace/{workspace_id}/database` |
| Get Row Details | GET | `/api/workspace/{workspace_id}/database/{database_id}/row/detail` |
| Create Row | POST | `/api/workspace/{workspace_id}/database/{database_id}/row` |

---

## 7. Current Limitations

Please note that the AppFlowy Cloud REST API is currently primarily focused on data retrieval and row-level operations.

### 7.1 Creating Databases, Fields, and Views
Currently, it is **not possible** to perform structural creations via the REST API. This includes:
- Creating a new **Database**.
- Adding or modifying **Fields** within a database.
- Creating new **Spaces** or **Pages**.

These structural changes are managed by the AppFlowy Native Application using its internal collaborative synchronization protocol (based on Yjs).

To create a new database in a specific space or add fields, you should use the AppFlowy desktop or mobile application. Once these structures are created, you can use the REST API to interact with the data (rows) within them.

### 7.2 Creating Workspaces
Similarly, creating new workspaces must be done through the AppFlowy application.
