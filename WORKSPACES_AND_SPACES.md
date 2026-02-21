# Understanding Workspaces and Spaces in AppFlowy

This document explains the conceptual and technical differences between **Workspaces** and **Spaces** in the AppFlowy ecosystem.

---

## 1. Workspaces

A **Workspace** is the highest level of organization in AppFlowy. It acts as a primary container for all your data, collaborative settings, and members.

- **Purpose:** Typically used to separate large domains of work (e.g., "Personal", "Company Name", "Freelance Clients").
- **Ownership:** A workspace has an owner and can have multiple members with different roles.
- **Isolation:** Data is generally isolated between different workspaces.
- **Identification:** Each workspace has a unique `workspace_id` (UUID).

### API Interaction
- **List Workspaces:** `GET /api/workspace`
- **Result:** Returns metadata about the workspace, including the `workspace_id`, name, and owner information.

---

## 2. Spaces

A **Space** is a sub-container within a Workspace. It is used to organize related pages, databases, and sub-folders.

- **Purpose:** Used to categorize content within a workspace (e.g., "Engineering", "Marketing", "HR").
- **Hierarchy:** Spaces exist directly under a Workspace. They can contain **Pages** (Notes), **Databases**, and other sub-folders.
- **Identification:** In the API, Spaces are identified by a `view_id` where the property `is_space` is set to `true`.

### API Interaction
- **Find Spaces:** Fetch the folder structure using `GET /api/workspace/{workspace_id}/folder`.
- **Logic:** Iterate through the top-level items in the `data.children` array. If an item has `"is_space": true`, it is a Space.

---

## 3. Hierarchy Diagram

```text
Workspace (Highest Level)
 └── Space A (is_space: true)
     ├── Page 1 (Note)
     ├── Database 1
     └── Sub-folder
         └── Page 2
 └── Space B (is_space: true)
     ├── Page 3
     └── Database 2
```

---

## 4. Key Differences

| Feature | Workspace | Space |
| :--- | :--- | :--- |
| **Level** | Root / Top-level | Second-level (under Workspace) |
| **Members** | Managed at this level | Inherits from Workspace |
| **API ID** | `workspace_id` | `view_id` |
| **API Identification** | Dedicated Endpoint | Property `is_space: true` in Folder view |

---

## 5. Management and Creation

### Via the Native Application
- **Workspaces:** Can be created, deleted, and switched within the AppFlowy Desktop/Mobile application.
- **Spaces:** Can be created within a workspace to organize your sidebar.

### Via the REST API (Current State)
- **Retrieval:** You can list workspaces and their internal space/folder structures.
- **Creation:** **Structural creation (creating new Workspaces or Spaces) is currently not supported via the REST API.**
- **Why?** Structural changes rely on AppFlowy's collaborative synchronization protocol (Yjs), which ensures real-time consistency across all clients. The REST API is currently optimized for data interaction (reading/writing rows) rather than schema/structural modification.

---

## 6. How to find a Space ID for API use

1. Call `GET /api/workspace` to get your `workspace_id`.
2. Call `GET /api/workspace/{workspace_id}/folder?depth=1`.
3. Locate the item in the `children` list where `"name": "Your Space Name"` and `"is_space": true`.
4. The `view_id` of that item is the ID for that Space.
