# TaskPilot – MCP-Powered AI Task Management Assistant

An AI-powered task management assistant built using **Java, Spring Boot, Spring AI, and Model Context Protocol (MCP)**. It enables users to manage tasks through natural-language conversations and generate personalized daily schedules using an LLM.

## 🚀 Overview

TaskPilot integrates an LLM with an MCP-based task management server, allowing the AI assistant to interact with external task management capabilities through standardized tools.

Instead of manually managing tasks, users can:
- Create new tasks using natural-language instructions.
- Retrieve and view pending tasks.
- Mark tasks as completed.
- Generate prioritized daily schedules based on available time.
- Leverage predefined task management guidelines for context-aware recommendations.

## 🏗️ Architecture

The application follows a two-service architecture:

1. **Task Client (Port 8080):** Handles user requests, communicates with the LLM through Spring AI, and connects to the MCP server.
2. **Task Server (Port 8081):** Exposes task management functionality through MCP Tools, Resources, and Prompts.

### Architecture Flow

```text
              User
                |
                v
       +------------------+
       |   Task Client    |
       |  Spring Boot     |
       |   Port: 8080     |
       +------------------+
                |
                v
       +------------------+
       |   Spring AI      |
       |   ChatClient     |
       +------------------+
                |
          LLM (Groq)
                |
                v
       +------------------+
       |    MCP Client    |
       +------------------+
                |
       MCP Streamable HTTP
                |
                v
       +------------------+
       |   MCP Server     |
       |  Spring Boot     |
       |   Port: 8081     |
       +------------------+
                |
       +--------+---------+
       |        |         |
       v        v         v
     Tools   Resources  Prompts
       |        |         |
       v        v         v
   Task CRUD  Guidelines Daily Planning
```

## ✨ Key Features

### 1. MCP Tools – Task Management

Implemented three MCP tools using Spring AI annotations:

| Tool | Description |
|---|---|
| `create_task` | Creates a new task using its title. |
| `list_tasks` | Retrieves all pending tasks. |
| `complete_task` | Marks a task as completed using its exact title. |

The LLM can dynamically discover and invoke these tools based on user requests.

### 2. MCP Resources – Context Management

Implemented an MCP Resource:

- URI: `task://guidelines`
- Provides predefined task management and prioritization guidelines.
- Helps the AI assistant generate recommendations using consistent scheduling rules.

Guidelines include:
- Prioritizing deep work before 12 PM.
- Scheduling meetings after 2 PM.
- Completing urgent tasks on the same day.
- Limiting high-priority tasks to three per day.

### 3. MCP Prompts – Daily Task Planning

Implemented a reusable MCP Prompt:

- Name: `plan_day`
- Accepts available hours as an argument.
- Generates a structured prompt instructing the LLM to retrieve pending tasks through available MCP tools.
- Produces an ordered daily schedule based on the available time.

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Java 21 | Programming language |
| Spring Boot | Backend application framework |
| Spring AI 2.0.2 | LLM integration |
| Model Context Protocol (MCP) | Standardized AI-to-tool communication |
| Groq API | LLM inference |
| Maven | Dependency management and build automation |

## 📂 Project Structure

```text
AI-TaskManagement-Assistant/
│
├── taskClient/
│   ├── pom.xml
│   └── src/main/
│       ├── java/in/chahak/taskClient/
│       │   ├── ChatController.java
│       │   ├── ChatService.java
│       │   └── TaskClientApplication.java
│       └── resources/
│           └── application.properties
│
├── taskServer/
│   ├── pom.xml
│   └── src/main/
│       ├── java/in/chahak/taskServer/
│       │   ├── TaskTools.java
│       │   ├── TaskResources.java
│       │   ├── TaskPrompts.java
│       │   └── TaskServerApplication.java
│       └── resources/
│           └── application.properties
│
└── README.md
```

## ⚙️ Getting Started

### Prerequisites

- Java 21 or later
- Maven
- Groq API key

### 1. Clone the Repository

```bash
git clone https://github.com/Chahak02/AI-TaskManagement-Assistant.git

cd AI-TaskManagement-Assistant
```

### 2. Configure Environment Variables

Generate a Groq API key from [Groq Console](https://console.groq.com/).

Set the API key as an environment variable:

```bash
export GROQ_API_KEY="your_groq_api_key"
```

Configure the following property in `taskClient/src/main/resources/application.properties`:

```properties
spring.application.name=taskClient
server.port=8080

spring.ai.openai.api-key=${GROQ_API_KEY}
spring.ai.openai.base-url=https://api.groq.com/openai/v1
spring.ai.openai.chat.options.model=openai/gpt-oss-20b

spring.ai.mcp.client.streamable-http.connections.task-server.url=http://localhost:8081
```

**Important:** Never commit API keys or other credentials to GitHub.

### 3. Start the MCP Server

Open a terminal:

```bash
cd taskServer
./mvnw spring-boot:run
```

The MCP server starts on:

`http://localhost:8081`

### 4. Start the Task Client

Open another terminal from the project root:

```bash
cd taskClient
mvn spring-boot:run
```

The task client starts on:

`http://localhost:8080`

Both services must be running for the application to work.

## 🔌 API Endpoints

The Task Client exposes the following REST endpoints:

| Method | Endpoint | Description |
|---|---|---|
| GET | `/ask?message={message}` | Sends a natural-language request to the AI assistant. |
| GET | `/plan?hours={hours}` | Generates a daily task plan based on available hours. |

### Example Requests

**Create a task**

```bash
curl --get "http://localhost:8080/ask" \
  --data-urlencode "message=Create a task to complete my Java assignment"
```

**List pending tasks**

```bash
curl --get "http://localhost:8080/ask" \
  --data-urlencode "message=Show me all my pending tasks"
```

**Complete a task**

```bash
curl --get "http://localhost:8080/ask" \
  --data-urlencode "message=Mark Java assignment as completed"
```

**Generate a daily plan**

```bash
curl --get "http://localhost:8080/plan" \
  --data-urlencode "hours=5"
```

## 🧠 Understanding the MCP Integration

This project demonstrates three core MCP capabilities:

- **Tools:** Allow the LLM to perform operations such as creating, listing, and completing tasks.
- **Resources:** Provide contextual information that the AI assistant can retrieve when generating responses.
- **Prompts:** Provide reusable, parameterized instructions for specific workflows such as daily task planning.

Together, these capabilities demonstrate how an LLM can interact with external services through a standardized protocol instead of relying only on its internal knowledge.

## ⚠️ Current Limitations

- Tasks are stored in an in-memory `ArrayList`; data is lost when the MCP server restarts.
- Task completion currently relies on exact title matching.
- Authentication and authorization are not implemented.
- The application currently uses synchronous MCP client interactions.

## 🔮 Future Improvements

- Integrate MySQL or PostgreSQL for persistent task storage.
- Introduce unique task IDs and richer task attributes such as priority and deadlines.
- Implement user authentication and authorization.
- Add task deadlines, reminders, and advanced scheduling.
- Introduce automated integration testing and improved error handling.

## 👨‍💻 Author

Developed as a project exploring **Spring AI, Model Context Protocol (MCP), LLM integration, and AI-powered backend applications**.
