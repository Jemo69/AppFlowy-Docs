# AppFlowy Cloud API Guide

This guide provides information on how to authenticate, re-authenticate, and perform common data operations using the AppFlowy Cloud REST API.

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

## 4. Databases

### 4.1 List Databases in a Workspace
**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database`

### 4.2 Get Database Fields
To know what data can be stored in a database, you need to retrieve its fields.

**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/fields`

---

## 5. Data Operations (Rows)

### 5.1 List Database Row IDs
**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row`

### 5.2 Get Row Details
**Endpoint:** `GET https://beta.appflowy.cloud/api/workspace/{workspace_id}/database/{database_id}/row/detail?ids={row_id1},{row_id2}`

### 5.3 Create a New Row
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

### 5.4 Upsert a Row
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
| List Databases | GET | `/api/workspace/{workspace_id}/database` |
| Create Row | POST | `/api/workspace/{workspace_id}/database/{database_id}/row` |
| Get Row Details | GET | `/api/workspace/{workspace_id}/database/{database_id}/row/detail` |
