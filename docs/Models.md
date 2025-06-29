## 🧑‍💻 Users

Represents registered users of the platform.

```sql
Table: users
```

| Column         | Type         | Constraints                                 |
| -------------- | ------------ | ------------------------------------------- |
| id             | UUID         | `PRIMARY KEY`, default: `gen_random_uuid()` |
| email          | VARCHAR(255) | `UNIQUE`, `NOT NULL`                        |
| password\_hash | VARCHAR(255) | `NOT NULL`                                  |
| first\_name    | VARCHAR(100) | Optional                                    |
| last\_name     | VARCHAR(100) | Optional                                    |
| is\_active     | BOOLEAN      | Default: `true`                             |
| is\_verified   | BOOLEAN      | Default: `false`                            |
| plan\_type     | VARCHAR(20)  | Default: `'free'`                           |
| created\_at    | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                |
| updated\_at    | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                |
| last\_login    | TIMESTAMP    | Optional                                    |

---

## 📁 Collections

Collections of API endpoints grouped by users.

```sql
Table: collections
```

| Column      | Type         | Constraints                                       |
| ----------- | ------------ | ------------------------------------------------- |
| id          | UUID         | `PRIMARY KEY`, default: `gen_random_uuid()`       |
| user\_id    | UUID         | `NOT NULL`, `FOREIGN KEY` → `users(id)` (CASCADE) |
| name        | VARCHAR(100) | `NOT NULL`                                        |
| description | TEXT         | Optional                                          |
| is\_public  | BOOLEAN      | Default: `false`                                  |
| tags        | JSONB        | Default: `'[]'`                                   |
| created\_at | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                      |
| updated\_at | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                      |

**Constraints:**

* Unique collection names per user: `UNIQUE(user_id, name)`

---

## 🔗 Endpoints

Describes an individual API request configuration.

```sql
Table: endpoints
```

| Column               | Type         | Constraints                                                       |
| -------------------- | ------------ | ----------------------------------------------------------------- |
| id                   | UUID         | `PRIMARY KEY`, default: `gen_random_uuid()`                       |
| collection\_id       | UUID         | `NOT NULL`, `FOREIGN KEY` → `collections(id)` (CASCADE)           |
| name                 | VARCHAR(100) | `NOT NULL`                                                        |
| method               | VARCHAR(10)  | `NOT NULL`, CHECK: `GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS` |
| url                  | TEXT         | `NOT NULL`                                                        |
| headers              | JSONB        | Default: `'{}'`                                                   |
| query\_params        | JSONB        | Default: `'{}'`                                                   |
| body                 | JSONB        | Optional                                                          |
| timeout\_seconds     | INTEGER      | Default: `10`, CHECK: `BETWEEN 1 AND 30`                          |
| auth\_config         | JSONB        | Default: `'{}'`                                                   |
| pre\_request\_script | TEXT         | Optional                                                          |
| test\_script         | TEXT         | Optional                                                          |
| created\_at          | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                                      |
| updated\_at          | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                                      |

---

## 📜 Request Logs

Stores information about past requests and responses for auditing and replay.

```sql
Table: request_logs
```

| Column                | Type      | Constraints                                       |
| --------------------- | --------- | ------------------------------------------------- |
| id                    | UUID      | `PRIMARY KEY`, default: `gen_random_uuid()`       |
| user\_id              | UUID      | `NOT NULL`, `FOREIGN KEY` → `users(id)` (CASCADE) |
| endpoint\_id          | UUID      | `FOREIGN KEY` → `endpoints(id)` (SET NULL)        |
| collection\_id        | UUID      | `FOREIGN KEY` → `collections(id)` (SET NULL)      |
| request\_snapshot     | JSONB     | `NOT NULL`                                        |
| response\_status      | INTEGER   | Optional                                          |
| response\_headers     | JSONB     | Optional                                          |
| response\_body        | TEXT      | Optional                                          |
| response\_size\_bytes | INTEGER   | Optional                                          |
| duration\_ms          | INTEGER   | Optional                                          |
| error\_message        | TEXT      | Optional                                          |
| ip\_address           | INET      | Optional                                          |
| user\_agent           | TEXT      | Optional                                          |
| created\_at           | TIMESTAMP | Default: `CURRENT_TIMESTAMP`                      |

**Indexes:**

* `(user_id, created_at DESC)`
* `(endpoint_id, created_at DESC)`

---

## 👥 Teams & Sharing

Advanced multi-user and team collaboration support.

### Teams

```sql
Table: teams
```

| Column      | Type         | Constraints                                 |
| ----------- | ------------ | ------------------------------------------- |
| id          | UUID         | `PRIMARY KEY`, default: `gen_random_uuid()` |
| name        | VARCHAR(100) | `NOT NULL`                                  |
| owner\_id   | UUID         | `NOT NULL`, `FOREIGN KEY` → `users(id)`     |
| created\_at | TIMESTAMP    | Default: `CURRENT_TIMESTAMP`                |

---

### Team Members

```sql
Table: team_members
```

| Column     | Type        | Constraints                                        |
| ---------- | ----------- | -------------------------------------------------- |
| team\_id   | UUID        | `FOREIGN KEY` → `teams(id)` (CASCADE)              |
| user\_id   | UUID        | `FOREIGN KEY` → `users(id)` (CASCADE)              |
| role       | VARCHAR(20) | Default: `'member'`, CHECK: `owner, admin, member` |
| joined\_at | TIMESTAMP   | Default: `CURRENT_TIMESTAMP`                       |

**Primary Key:** `(team_id, user_id)`

---

### Collection Shares

```sql
Table: collection_shares
```

| Column                 | Type        | Constraints                                             |
| ---------------------- | ----------- | ------------------------------------------------------- |
| id                     | UUID        | `PRIMARY KEY`, default: `gen_random_uuid()`             |
| collection\_id         | UUID        | `NOT NULL`, `FOREIGN KEY` → `collections(id)` (CASCADE) |
| shared\_with\_user\_id | UUID        | `FOREIGN KEY` → `users(id)` (CASCADE)                   |
| shared\_with\_team\_id | UUID        | `FOREIGN KEY` → `teams(id)` (CASCADE)                   |
| permission             | VARCHAR(20) | Default: `'read'`, CHECK: `read, write`                 |
| created\_at            | TIMESTAMP   | Default: `CURRENT_TIMESTAMP`                            |

**Check Constraint:** Exactly one of `shared_with_user_id` or `shared_with_team_id` must be set (XOR).
