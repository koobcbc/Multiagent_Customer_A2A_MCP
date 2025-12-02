# Multi-Agent Customer Service System with A2A and MCP

A multi-agent customer service system that demonstrates Agent-to-Agent (A2A) communication and Model Context Protocol (MCP) integration. The system consists of specialized agents that coordinate to handle customer queries, manage customer data, and process support tickets.

## Overview

This project implements a three-agent system:
- **Router Agent (Orchestrator)**: Receives queries, analyzes intent, and routes to specialist agents
- **Customer Data Agent**: Manages customer database operations via MCP
- **Support Agent**: Handles customer support queries and ticket management

## Architecture

- **A2A Communication**: Agents communicate using the Agent-to-Agent protocol
- **MCP Integration**: Agents access customer data through a Model Context Protocol server
- **Database**: SQLite database storing customers and support tickets

## Prerequisites

- Python 3.10 or higher
- Google Cloud Project with API access
- Google API Key (for Gemini models)
- (Optional) ngrok account for public MCP server access

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd assignment5
```

### 2. Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install --upgrade google-genai google-adk==1.9.0 a2a-sdk==0.3.0 python-dotenv aiohttp uvicorn requests mermaid-python nest-asyncio flask flask-cors termcolor pyngrok
```

Or install from requirements file:

```bash
pip install -r requirements.txt
```

### 4. Set Up Environment Variables

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_google_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

**For Google Colab:**
- Use the Secrets feature (🔑 icon in left sidebar) to store:
  - `GOOGLE_API_KEY`: Your Google API key
  - `NGROK_AUTHTOKEN`: Your ngrok authtoken (optional, for public MCP server)

### 5. Initialize Database

Run the database setup script:

```bash
python database_setup.py
```

This will:
- Create the `support.db` SQLite database
- Create `customers` and `tickets` tables
- Optionally insert sample data
- Run sample queries to verify setup

Follow the prompts to:
- Insert sample data (recommended: 'y')
- Run sample queries (recommended: 'y')

## Usage

### Option 1: Google Colab (Recommended for Testing)

#### Step 1: Start MCP Server

1. Open `mcp_integration.ipynb` in Google Colab
2. Run all cells sequentially:
   - Cell 1: Install packages
   - Cell 2: Database setup (creates tables and sample data)
   - Cell 3: Define MCP tool functions
   - Cell 4: Build MCP server
   - Cell 5: Start MCP server
3. Note the MCP server URL:
   - Local: `http://127.0.0.1:5000/mcp`
   - ngrok: `https://your-ngrok-url.ngrok-free.dev/mcp` (if using ngrok)

#### Step 2: Start A2A Agents

1. Open `a2a.ipynb` in Google Colab
2. Update `MCP_SERVER_URL` in the notebook with your MCP server URL from Step 1
3. Run all cells sequentially:
   - Cell 1: Install packages
   - Cell 2: Compatibility workaround
   - Cell 3: Import libraries
   - Cell 4: Environment configuration
   - Cell 5: Authenticate (Colab only)
   - Cell 6: Setup logging
   - Cells 7-10: Create Customer Data Agent
   - Cells 11-14: Create Support Agent
   - Cells 15-16: Create Router Agent
   - Cell 17: Define server creation function
   - Cell 18: Start all A2A servers
   - Cell 19: Verify agent cards
   - Cell 20: Define A2ASimpleClient
   - Cell 21: Define ask_agent helper function
4. Wait for servers to start (check output for "✅ All agent servers started!")

#### Step 3: Test the System

Use the `ask_agent()` function to test queries:

```python
# Simple query
await ask_agent("Get customer information for ID 5")

# Coordinated query
await ask_agent("I'm customer 5 and need help upgrading my account")

# Complex query
await ask_agent("Show me all active customers who have open tickets")
```

### Option 2: Local Python Environment

#### Step 1: Start MCP Server

**Option A: Using Jupyter Notebook**
```bash
jupyter notebook mcp_integration.ipynb
```
Run all cells to start the MCP server on port 5000.

**Option B: Extract Flask App (Advanced)**
Extract the Flask app from the notebook and run:
```bash
python mcp_server.py
```

#### Step 2: Start A2A Agents

```bash
jupyter notebook a2a.ipynb
```

1. Update `MCP_SERVER_URL` to `http://localhost:5000/mcp`
2. Run all cells to start:
   - Customer Data Agent (port 10021)
   - Support Agent (port 10020)
   - Router Agent (port 10022)

#### Step 3: Test the System

Use the `ask_agent()` helper function or the `A2ASimpleClient` to send queries.

## Project Structure

```
assignment5/
├── README.md                 # This file
├── requirements.txt          # Python dependencies
├── database_setup.py        # Database initialization script
├── mcp_integration.ipynb    # MCP server implementation
├── a2a.ipynb                # A2A agent system implementation
├── support.db               # SQLite database (created after setup)
└── .env                     # Environment variables (create this)
```

## MCP Tools

The MCP server exposes the following tools:

1. **get_customer(customer_id)**: Retrieve customer by ID
2. **list_customers(status, limit)**: List customers with optional status filter
3. **update_customer(customer_id, data)**: Update customer information (data is a dict with name, email, phone, status)
4. **create_ticket(customer_id, issue, priority)**: Create a support ticket
5. **get_customer_history(customer_id)**: Get all tickets for a customer

## Configuration

### Port Assignments

- **MCP Server**: Port 5000 (default)
- **Customer Data Agent**: Port 10021
- **Support Agent**: Port 10020
- **Router Agent**: Port 10022

**Important**: Ensure these ports are consistent across:
- `AgentCard` URL definitions
- `RemoteA2aAgent` endpoints
- Server startup configurations

### MCP Server URL

Update the `MCP_SERVER_URL` variable in the A2A notebook:
- Local: `http://localhost:5000/mcp`
- ngrok: `https://your-ngrok-url.ngrok-free.dev/mcp`

## Testing

### Test Scenarios

1. **Simple Query**: "Get customer information for ID 5"
   - Single agent, straightforward MCP call

2. **Coordinated Query**: "I'm customer 5 and need help upgrading my account"
   - Multiple agents coordinate: data fetch + support response

3. **Complex Query**: "Show me all active customers who have open tickets"
   - Requires negotiation between data and support agents

4. **Escalation**: "I've been charged twice, please refund immediately!"
   - Router identifies urgency and routes appropriately

5. **Multi-Intent**: "Update my email to new@email.com and show my ticket history"
   - Parallel task execution and coordination

## Troubleshooting

### Common Issues

**1. Port Already in Use**
```bash
# Find and kill process using the port (macOS/Linux)
lsof -ti:5000 | xargs kill -9  # For port 5000
lsof -ti:10020 | xargs kill -9  # For Support Agent
lsof -ti:10021 | xargs kill -9  # For Customer Data Agent
lsof -ti:10022 | xargs kill -9  # For Router Agent

# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F
```

**2. Database Connection Error**
- Ensure `support.db` exists (run `database_setup.py`)
- Check that `DB_PATH` is correctly set in the MCP integration notebook
- Verify the database file is in the correct location

**3. MCP Tool Execution Errors**
- Verify function signatures match MCP tool definitions exactly
- Check that `update_customer` uses `data` parameter (dict), not individual parameters
- Ensure `list_customers` includes `limit` parameter
- Verify `DB_PATH` is defined in the functions cell

**4. Agent Response Errors (`NoneType` has no attribute 'parts')**
- The `ask_agent()` function includes defensive checks for None content
- If errors persist, check agent coordination and MCP server connectivity
- Verify all agents are running and accessible

**5. 404 Errors on Agent Endpoints**
- Verify port numbers match across all configurations:
  - `AgentCard` URLs
  - `RemoteA2aAgent` endpoints
  - Server startup ports
- Check that servers have started successfully
- Wait a few seconds after server startup before testing
- Verify server status using the health check endpoints

**6. Google API Key Issues**
- Verify `GOOGLE_API_KEY` is set correctly in `.env` or Colab secrets
- Check API key has proper permissions for Gemini models
- Ensure you're not using Vertex AI if `GOOGLE_GENAI_USE_VERTEXAI=FALSE`
- Test API key with a simple Google GenAI call

**7. Compatibility Issues**
- Ensure you're using the exact versions: `google-adk==1.9.0` and `a2a-sdk==0.3.0`
- The compatibility workaround in the notebook must be run before importing ADK modules
- If issues persist, try restarting the runtime/kernel

### Debugging Tips

1. **Test Agents Individually**: Test each agent before testing coordination
2. **Check Logs**: Enable logging to see agent-to-agent communication
3. **Verify MCP Server**: Test MCP tools directly using the MCP Inspector:
   ```bash
   npx @modelcontextprotocol/inspector
   ```
   Then enter your MCP server URL
4. **Validate Configuration**: Double-check all port assignments and URLs
5. **Check Event Loop**: Ensure `nest_asyncio.apply()` is called before running async code in notebooks

## Dependencies

### Core Packages
- `google-genai`: Google's Generative AI SDK
- `google-adk==1.9.0`: Google Agent Development Kit
- `a2a-sdk==0.3.0`: Agent-to-Agent SDK

### Server & Protocol
- `flask`: MCP server web framework
- `flask-cors`: CORS support for MCP server
- `uvicorn`: ASGI server for A2A agents
- `aiohttp`: Async HTTP client

### Utilities
- `python-dotenv`: Environment variable management
- `nest-asyncio`: Nested event loop support for notebooks
- `pyngrok`: Public tunnel for MCP server (optional)
- `termcolor`: Colored terminal output

## Known Issues & Workarounds

### Compatibility Workaround

For `google-adk==1.9.0` and `a2a-sdk==0.3.0` compatibility, a workaround is included in the A2A notebook. This patches the `A2ACardResolver` reference issue and should be removed when google-adk releases version >1.9.0.

The workaround code:
```python
import sys
from a2a.client import client as real_client_module
from a2a.client.card_resolver import A2ACardResolver

class PatchedClientModule:
    def __init__(self, real_module) -> None:
        for attr in dir(real_module):
            if not attr.startswith('_'):
                setattr(self, attr, getattr(real_module, attr))
        self.A2ACardResolver = A2ACardResolver

patched_module = PatchedClientModule(real_client_module)
sys.modules['a2a.client.client'] = patched_module
```

## Database Schema

### Customers Table
- `id`: INTEGER PRIMARY KEY
- `name`: TEXT NOT NULL
- `email`: TEXT
- `phone`: TEXT
- `status`: TEXT NOT NULL DEFAULT 'active' (values: 'active', 'disabled')
- `created_at`: TIMESTAMP
- `updated_at`: TIMESTAMP

### Tickets Table
- `id`: INTEGER PRIMARY KEY
- `customer_id`: INTEGER NOT NULL (FK to customers.id)
- `issue`: TEXT NOT NULL
- `status`: TEXT NOT NULL DEFAULT 'open' (values: 'open', 'in_progress', 'resolved')
- `priority`: TEXT NOT NULL DEFAULT 'medium' (values: 'low', 'medium', 'high')
- `created_at`: DATETIME

## License

[Add your license here]

## Author

[Your name]

## Acknowledgments

- Google ADK and A2A SDK teams
- Model Context Protocol specification

