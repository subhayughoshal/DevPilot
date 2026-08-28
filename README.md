# 🚀 DevPilot

### AI-Powered GitHub Repository Assistant

DevPilot is an AI-powered developer assistant that allows you to **connect your GitHub repositories, index their source code, and interact with the codebase through a conversational interface**.

Instead of manually searching through hundreds of files, developers can ask questions such as:

> *"Where is authentication implemented?"*
> *"Explain how the repository indexing works."*
> *"Which class handles GitHub API communication?"*
> *"How does the chat system retrieve relevant code?"*

DevPilot retrieves the most relevant pieces of the indexed codebase and uses them as context for the AI, providing answers along with **code citations**.

---

## ✨ Features

* 🔐 **GitHub OAuth Login**

  * Sign in using your GitHub account.
  * GitHub access tokens are securely stored in encrypted form.

* 📂 **GitHub Repository Integration**

  * Fetch repositories accessible to the authenticated user.
  * Supports public and private repositories according to the GitHub permissions granted.

* 🧠 **AI-Powered Code Understanding**

  * Indexes source files from a GitHub repository.
  * Splits source code into manageable chunks.
  * Generates vector representations for semantic search.

* 🔎 **RAG-Based Code Retrieval**

  * Finds code chunks relevant to the user's question.
  * Uses repository-specific filtering to prevent unrelated code from being included.

* 💬 **Conversational Code Assistant**

  * Ask questions about an indexed repository.
  * Chat sessions are stored for later access.
  * AI responses are streamed to the frontend using Server-Sent Events (SSE).

* 📌 **Code Citations**

  * Responses can include references to the source files used to generate the answer.

* 📊 **Indexing Progress**

  * Track repository indexing status and progress.
  * Displays files processed and code chunks generated.

* 🗄️ **PostgreSQL + pgvector**

  * PostgreSQL stores application data.
  * pgvector provides vector similarity search for the RAG pipeline.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │       GitHub         │
                         │  OAuth + Repository  │
                         │       Contents       │
                         └──────────┬───────────┘
                                    │
                                    ▼
┌──────────────────┐       ┌──────────────────────┐
│                  │       │   Spring Boot API    │
│   Next.js        │◄─────►│                      │
│   Frontend       │  HTTP │  Authentication      │
│                  │  SSE  │  Repository API      │
└────────┬─────────┘       │  Indexing            │
         │                 │  RAG / AI Chat       │
         │                 └──────────┬───────────┘
         │                            │
         │                            ▼
         │                  ┌──────────────────────┐
         │                  │     PostgreSQL       │
         │                  │                      │
         │                  │  Application Data    │
         │                  │  + pgvector          │
         │                  │  Vector Embeddings   │
         │                  └──────────┬───────────┘
         │                             │
         │                             ▼
         │                  ┌──────────────────────┐
         └─────────────────►│      AI Model        │
                            │    Spring AI         │
                            │       + LLM          │
                            └──────────────────────┘
```

### RAG Pipeline

DevPilot follows a Retrieval-Augmented Generation workflow:

```text
GitHub Repository
       │
       ▼
Fetch Repository Tree
       │
       ▼
Filter Source Files
       │
       ▼
Read File Contents
       │
       ▼
Split Code into Chunks
       │
       ▼
Generate Embeddings
       │
       ▼
Store in PostgreSQL + pgvector
       │
       ▼
       ┌──────────────────────┐
       │      User Question   │
       └──────────┬───────────┘
                  │
                  ▼
        Semantic Similarity Search
                  │
                  ▼
          Relevant Code Chunks
                  │
                  ▼
            Prompt + Context
                  │
                  ▼
               AI Model
                  │
                  ▼
          Streaming AI Response
                  │
                  ▼
          Answer + Citations
```

---

## 🛠️ Technology Stack

### Frontend

| Technology         | Purpose                 |
| ------------------ | ----------------------- |
| **Next.js 16**     | Frontend framework      |
| **React 19**       | UI development          |
| **TypeScript**     | Type-safe development   |
| **Tailwind CSS**   | Styling                 |
| **shadcn/ui**      | UI components           |
| **TanStack Query** | Server-state management |
| **SSE**            | Streaming AI responses  |

### Backend

| Technology          | Purpose                        |
| ------------------- | ------------------------------ |
| **Java 21**         | Backend language               |
| **Spring Boot 4**   | Backend framework              |
| **Spring Security** | Authentication & authorization |
| **Spring Data JPA** | Database persistence           |
| **Spring AI**       | AI integration                 |
| **OAuth 2.0**       | GitHub authentication          |
| **Maven**           | Dependency management          |

### Database & AI Infrastructure

| Technology          | Purpose                      |
| ------------------- | ---------------------------- |
| **PostgreSQL 16**   | Relational database          |
| **pgvector**        | Vector similarity search     |
| **Docker Compose**  | Local PostgreSQL environment |
| **GitHub REST API** | Repository access            |

---

# 🚀 Getting Started

## 1. Prerequisites

Make sure the following are installed:

* **Java 21**
* **Node.js 20+**
* **npm**
* **Docker Desktop**
* **Git**
* A **GitHub account**
* An **AI API key compatible with the current Spring AI configuration**

Verify the installations:

```bash
java -version
node -v
npm -v
docker --version
git --version
```

---

# 📥 2. Clone the Repository

```bash
git clone https://github.com/subhayughosha/DevPilot.git
cd DevPilot
```

---

# 🗄️ 3. Start PostgreSQL + pgvector

DevPilot includes a Docker Compose configuration for PostgreSQL with pgvector.

From the project root:

```bash
docker compose up -d
```

Check that the container is running:

```bash
docker ps
```

The database is configured as:

```text
Database: devpilot
Username: postgres
Password: postgres
Host: localhost
Port: 5433
```

The PostgreSQL container internally uses port `5432`, while port `5433` is exposed on the host to avoid conflicts with an existing PostgreSQL installation.

To stop the database:

```bash
docker compose down
```

---

# 🔑 4. Configure GitHub OAuth

DevPilot uses GitHub OAuth for authentication.

Create a GitHub OAuth application from your GitHub Developer Settings.

Use the following local configuration:

```text
Homepage URL:
http://localhost:3000

Authorization callback URL:
http://localhost:8080/login/oauth2/code/github
```

You will receive:

```text
GitHub Client ID
GitHub Client Secret
```

Keep the client secret private and **never commit it to GitHub**.

---

# ⚙️ 5. Configure the Backend

Create:

```text
backend/src/main/resources/application.properties
```

Example local configuration:

```properties
spring.application.name=devpilot

# --------------------------------------------------
# PostgreSQL
# --------------------------------------------------

spring.datasource.url=jdbc:postgresql://localhost:5433/devpilot
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false

# --------------------------------------------------
# GitHub OAuth
# --------------------------------------------------

spring.security.oauth2.client.registration.github.client-id=YOUR_GITHUB_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_GITHUB_CLIENT_SECRET
spring.security.oauth2.client.registration.github.scope=read:user,user:email,repo

# --------------------------------------------------
# DevPilot Application Configuration
# --------------------------------------------------

app.frontend-url=http://localhost:3000
app.cors.allowed-origins=http://localhost:3000

# Used to encrypt GitHub access tokens
app.token-encryptor-password=CHANGE_THIS_TO_A_STRONG_SECRET
app.token-encryptor-salt=CHANGE_THIS_TO_AN_8_CHAR_SALT

# GitHub API indexing configuration
app.github.api-delay-ms=50

# Code indexing configuration
app.indexing.chunk-size=800
app.indexing.max-file-bytes=102400

# --------------------------------------------------
# Spring AI
# --------------------------------------------------

spring.ai.openai.api-key=YOUR_AI_API_KEY
```

### ⚠️ Important

Do **not** commit `application.properties` if it contains real credentials.

Add it to `.gitignore`:

```gitignore
backend/src/main/resources/application.properties
```

A safer approach is to use environment variables for production deployments.

---

# 🤖 6. AI API Configuration

The current DevPilot implementation uses **Spring AI's OpenAI integration** for the chat generation and embedding workflow.

Therefore, the current version requires an API key compatible with the configured Spring AI OpenAI provider.

Set:

```properties
spring.ai.openai.api-key=YOUR_AI_API_KEY
```

> **Note:** The application architecture can be adapted to other providers such as Gemini, Groq, Ollama, or other OpenAI-compatible APIs. However, the repository's current implementation is configured around Spring AI's OpenAI starter, so switching providers requires corresponding configuration/code changes.

---

# ☕ 7. Run the Backend

Open a terminal inside the backend directory:

```bash
cd backend
```

### Windows

```powershell
.\mvnw.cmd spring-boot:run
```

### Linux / macOS

```bash
./mvnw spring-boot:run
```

The backend should start on:

```text
http://localhost:8080
```

---

# 💻 8. Run the Frontend

Open another terminal:

```bash
cd client
```

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

The frontend will be available at:

```text
http://localhost:3000
```

If the backend is running on a different URL, create a `.env.local` file inside `client`:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080
```

---

# 🔄 9. Using DevPilot

Once both the frontend and backend are running:

### Step 1 — Login

Open:

```text
http://localhost:3000
```

Sign in using **GitHub OAuth**.

### Step 2 — Select a Repository

DevPilot retrieves the repositories accessible through your GitHub account.

Select the repository you want to analyze.

### Step 3 — Index the Repository

Start repository indexing.

DevPilot will:

1. Retrieve the repository tree from GitHub.
2. Identify indexable files.
3. Download the source code.
4. Split files into code chunks.
5. Generate vector representations.
6. Store the vectors in PostgreSQL/pgvector.
7. Mark the repository as ready.

### Step 4 — Start a Chat

Once indexing is complete, create a chat session and ask questions about the codebase.

For example:

```text
How does authentication work in this project?
```

or:

```text
Where is GitHub OAuth implemented?
```

or:

```text
Explain the repository indexing pipeline.
```

DevPilot retrieves relevant code before sending the context to the AI model.

### Step 5 — Explore Citations

AI responses can contain citations pointing back to the relevant source files used during retrieval.

---

# 📁 Project Structure

```text
DevPilot/
│
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   └── java/
│   │   │       └── devPilot/
│   │   │           └── backend/
│   │   │               ├── config/
│   │   │               ├── controllers/
│   │   │               ├── dto/
│   │   │               ├── entity/
│   │   │               ├── exceptions/
│   │   │               ├── repository/
│   │   │               ├── security/
│   │   │               └── services/
│   │   │                   ├── ai/
│   │   │                   ├── github/
│   │   │                   └── indexing/
│   │   │
│   │   └── test/
│   │
│   ├── pom.xml
│   ├── mvnw
│   └── mvnw.cmd
│
├── client/
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   ├── public/
│   ├── package.json
│   └── ...
│
├── docker/
│   └── postgres/
│       └── init-extensions.sql
│
├── docker-compose.yml
└── README.md
```

---

# 🔐 Security Considerations

DevPilot handles GitHub authentication and repository access, so credentials must be handled carefully.

The application includes token encryption for stored GitHub access tokens.

For local development:

* Never commit GitHub client secrets.
* Never commit AI API keys.
* Never commit production database credentials.
* Keep `application.properties` and `.env` files out of version control.
* Use environment variables or a secrets manager for production deployments.

---

# 🧪 Building the Project

### Backend

```bash
cd backend
```

Windows:

```powershell
.\mvnw.cmd clean package
```

Linux/macOS:

```bash
./mvnw clean package
```

### Frontend

```bash
cd client
npm run build
```

---

# 🐳 PostgreSQL Management

Start PostgreSQL:

```bash
docker compose up -d
```

View logs:

```bash
docker compose logs -f postgres
```

Stop PostgreSQL:

```bash
docker compose down
```

Stop and remove the database volume:

```bash
docker compose down -v
```

> ⚠️ `docker compose down -v` permanently removes the local PostgreSQL data stored in the Docker volume.

---

# 🧠 How the AI System Works

DevPilot does not simply send the user's question directly to the AI model.

Instead, it follows a RAG pipeline:

```text
User Question
      │
      ▼
Semantic Search
      │
      ▼
Relevant Code Chunks
      │
      ▼
Repository Context
      │
      ▼
Prompt Construction
      │
      ▼
AI Model
      │
      ▼
Streaming Response
      │
      ▼
Answer + Citations
```

This approach allows the assistant to answer questions using the **actual code contained in the connected repository**, rather than relying solely on the model's general knowledge.

---

# 🔌 API Overview

The backend exposes REST endpoints for major application operations.

### Authentication

```text
/api/auth/me
/api/auth/logout
```

### Repositories

```text
/api/repos
/api/repos/{id}
/api/repos/{id}/index
/api/repos/{id}/status
```

### Chat

```text
/api/chat/sessions
/api/chat/sessions/{sessionId}
/api/chat/sessions/{sessionId}/messages
```

The chat response is streamed to the frontend using **Server-Sent Events (SSE)**.

---

# 🛣️ Future Improvements

Some potential extensions for DevPilot include:

* 🌐 Support for multiple AI providers
* 🤖 Local AI support using Ollama
* 📝 Automatic code documentation
* 🐛 AI-powered bug detection
* 🔍 Advanced code search
* 🧪 Test-case generation
* 📊 Repository analytics
* 🔀 Pull-request analysis
* 💡 Code improvement suggestions
* 🗂️ Multi-repository conversations
* 👥 Team/workspace support
* ☁️ Cloud deployment
* 🔐 More granular GitHub permissions
* 📚 Support for additional programming languages

---

# 🤝 Contributing

Contributions, suggestions, and improvements are welcome.

1. Fork the repository.
2. Create a feature branch.

```bash
git checkout -b feature/your-feature
```

3. Commit your changes.

```bash
git commit -m "Add your feature"
```

4. Push the branch.

```bash
git push origin feature/your-feature
```

5. Open a Pull Request.

---

# 📄 License

This project is currently intended for educational and development purposes.

A formal open-source license can be added when the project is ready for public distribution.

---

<div align="center">

### 🚀 DevPilot

**Understand your codebase. Ask questions. Build faster.**

</div>
