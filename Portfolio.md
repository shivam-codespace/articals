 

 
# 🚀 Portfolio & Blog Platform

A full-stack **portfolio + article/blog platform** built using **Spring Boot (Backend)** and **React (Frontend)**.  
The application allows managing articles stored in MongoDB while dynamically fetching and rendering **Markdown content directly from GitHub**.

This project is designed to demonstrate **real-world backend flow, clean API design, and modern frontend rendering**.

---

## 🧠 Project Overview

- Articles metadata (title, slug, description, GitHub raw URL) are stored in **MongoDB**
- Actual article content is written in **Markdown (.md)** and stored on **GitHub**
- Backend fetches Markdown from GitHub and serves it to frontend
- Frontend renders Markdown as a blog article using React

This approach keeps content **version-controlled**, **lightweight**, and **developer-friendly**.

---

## 🛠️ Tech Stack

### 🔹 Frontend
- **React.js**
- **React Router**
- **Axios / Fetch API**
- **React Markdown**
- **HTML5, CSS3**
- **SEO optimization (React Helmet)**

### 🔹 Backend
- **Spring Boot**
- **Spring Web**
- **Spring Data MongoDB**
- **MongoDB Atlas**
- **SLF4J Logging**
- **RESTful APIs**

### 🔹 Other Tools
- **Git & GitHub**
- **Markdown (.md)**
- **Postman**
- **VS Code**

---

## 🗂️ Backend Architecture & Flow

### 📌 Article Creation Flow

1. Client sends article details:
   - title
   - description
   - GitHub raw markdown URL
   - keywords
2. Backend:
   - Generates a **SEO-friendly slug**
   - Stores metadata in MongoDB
3. Markdown file remains in GitHub (not stored in DB)

---

### 📌 Fetch Article by Slug (Main Flow)

```

Frontend
↓
GET /api/articles/{slug}
↓
ArticleController
↓
ArticleService
↓
MongoDB → fetch article metadata
↓
GitHubMarkdownService → fetch .md content
↓
ArticleResponse (title + description + markdown content)
↓
Frontend renders markdown

```

✔ Backend controls GitHub access  
✔ Frontend never talks directly to GitHub  
✔ Clean separation of concerns  

---

## ✨ Key Features

- 📄 Dynamic blog/article system
- 🔗 GitHub-based Markdown content
- 🧩 SEO-friendly slugs
- ⚡ Fast content updates (no redeploy needed)
- 🗃️ MongoDB Atlas integration
- 📜 Markdown rendering on frontend
- 🪵 Structured logging with SLF4J
- 🧼 Clean REST API design

---

## 📦 API Endpoints

### 🔹 Create Article
```

POST /api/articles

```

### 🔹 Get All Articles
```

GET /api/articles

```

### 🔹 Get Article by Slug
```

GET /api/articles/{slug}

```

### 🔹 Delete Article
```

DELETE /api/articles/{id}

```

---

## 🧪 Logging & Debugging

- Centralized logging using **SLF4J**
- Logs:
  - API calls
  - Database interactions
  - GitHub markdown fetch status
- Helps in easy debugging and production monitoring

---

## 🎯 Why This Project Is Different

- No hardcoded blog content
- No CMS dependency
- GitHub acts as a **content source**
- Backend handles security & fetching
- Frontend focuses only on UI & rendering

This architecture mirrors **real production systems**.

---

## 📌 Future Enhancements

- ✍️ Admin dashboard
- 🔐 Authentication & authorization
- 🧠 Markdown caching
- 💬 Comments system
- 📊 Article analytics
- 🌙 Dark mode


## 🏗️ System Architecture (Textual Figure)

```text
┌──────────────┐
│   User       │
│ (Browser)    │
└──────┬───────┘
       │
       │ HTTP Requests
       ▼
┌────────────────────┐
│  React Frontend    │
│  (Portfolio UI)    │
└──────┬─────────────┘
       │ REST API Calls
       ▼
┌────────────────────┐
│  Spring Boot API   │
│  (Backend Layer)  │
└──────┬───────┬────┘
       │       │
       │       │ Fetch Markdown
       │       ▼
       │   ┌──────────────┐
       │   │   GitHub     │
       │   │  Markdown   │
       │   └──────────────┘
       │
       │ Fetch Metadata
       ▼
┌────────────────────┐
│  MongoDB Atlas     │
│ (Article Metadata)│
└────────────────────┘
```

---

## 🔄 Article Fetch Flow (Slug-Based)

```text
User clicks article
        │
        ▼
React Router (/articles/:slug)
        │
        ▼
GET /api/articles/{slug}
        │
        ▼
ArticleController
        │
        ▼
ArticleService
        │
        ├── Fetch article metadata from MongoDB
        │
        ├── Fetch .md file from GitHub (raw URL)
        │
        ▼
Combined Response (JSON)
        │
        ▼
React Markdown Renderer
        │
        ▼
Rendered Blog Page
```

---

## 🧩 Backend Internal Architecture

```text
Controller Layer
┌────────────────────┐
│ ArticleController  │
└─────────┬──────────┘
          │
          ▼
Service Layer
┌────────────────────┐
│ ArticleService     │
│ GitHubFetchService │
└─────────┬──────────┘
          │
          ▼
Repository Layer
┌────────────────────┐
│ ArticleRepository  │
│ (MongoDB)          │
└────────────────────┘
```

✔ Separation of concerns
✔ Easy to test
✔ Production-friendly structure

---

## 📦 Data Model Flow

```text
MongoDB Document
┌─────────────────────────────┐
│ id                          │
│ title                       │
│ slug                        │
│ description                 │
│ githubRawUrl                │
│ keywords                    │
│ createdAt                   │
└─────────────────────────────┘
              │
              ▼
GitHub Markdown File (.md)
┌─────────────────────────────┐
│ # Article Title             │
│ ## Sections                 │
│ - Code blocks               │
│ - Images                    │
│ - Lists                     │
└─────────────────────────────┘
```

---

## 🪵 Logging & Monitoring Flow

```text
Incoming Request
       │
       ▼
Controller Log
       │
       ▼
Service Log
       │
       ├── MongoDB Fetch Log
       ├── GitHub Fetch Log
       ▼
Response Sent Log
```

Example:

```text
INFO  Fetching article by slug
INFO  Markdown content fetched successfully
INFO  Article fetched successfully
```

---

## ⚡ Why This Architecture Works Well

```text
✔ Content versioned via GitHub
✔ Backend-controlled data access
✔ Frontend stays lightweight
✔ No redeployment for content updates
✔ Easy to scale & extend
 

## 👨‍💻 Author

**Shivam Sharma**  
MCA (2026) | Java Backend Developer  
Spring Boot • MongoDB • React  

---
## 🔗 GitHub Repository

```md
### Source Code Repository

The complete source code for this project is available on GitHub.  
It includes frontend (React), backend (Spring Boot), database configuration, and documentation.

🔗 GitHub(Frontend) Link: https://github.com/shivam-codespace/Personal-Portfolio.git
🔗 GitHub(Backend) Link: https://github.com/shivam-codespace/articals-service.git
```

---

## 📁 Project Code Availability

```md
### Project Code

All modules of the Integrated Eligibility System (IES) Application are maintained in a public GitHub repository.  
The repository contains:
- Backend source code (Spring Boot)
- Frontend source code (React)
- Database scripts
- API documentation
- README and project documentation

# Repository URL:

🔗 GitHub(Frontend) Link: https://github.com/shivam-codespace/Personal-Portfolio.git
🔗 GitHub(Backend) Link: https://github.com/shivam-codespace/articals-service.git

---

## 👨‍💻 Developer & Repository Information

```md
### Developer Information

**Developer:** Shivam Sharma  
**Degree:** MCA (Batch 2026)  
**University:** MMMUT, Gorakhpur  

The project is developed and maintained using GitHub for version control.

🔗 GitHub(Frontend) Link: https://github.com/shivam-codespace/Personal-Portfolio.git
🔗 GitHub(Backend) Link: https://github.com/shivam-codespace/articals-service.git
```

---

## 🚀 How to Access the Project

```md
### Accessing the Project Source Code

To explore or run this project locally, visit the GitHub repository using the link below:

🔗 GitHub(Frontend) Link: https://github.com/shivam-codespace/Personal-Portfolio.git
🔗 GitHub(Backend) Link: https://github.com/shivam-codespace/articals-service.git

Clone the repository:
```

```bash
git clone https://github.com/shivam-codespace/articals-service.git

git clone https://github.com/shivam-codespace/Personal-Portfolio.git
```

---

## 📌 Reference Section Entry (For Report / README)

```md
### References

- Project Source Code Repository: 
🔗 GitHub(Frontend) Link: https://github.com/shivam-codespace/Personal-Portfolio.git
🔗 GitHub(Backend) Link: https://github.com/shivam-codespace/articals-service.git
```

---

 
