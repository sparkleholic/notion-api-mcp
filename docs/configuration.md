# Configuration Details

## Environment Variables

The server supports configuration through both a .env file and system environment variables. When both are present, system environment variables (including those set in the Claude Desktop MCP config) take precedence over .env file values.

Required:
- `NOTION_API_KEY`: Your Notion API integration token
  - Must start with "ntn_"
  - Get from https://www.notion.so/my-integrations

Optional (at least one is required):
- `NOTION_PARENT_PAGE_ID`: ID of a Notion page where you want to create new databases
  - Must be a page that has granted access to your integration
  - Required if you want to create new databases
  - Get from the page's URL

- `NOTION_DATABASE_ID`: ID of an existing database
  - Must be a database that has granted access to your integration
  - Required if you want to work with an existing database
  - Get from the database's URL

---

### Server Operation Mode

#### `MCP_SERVER_TYPE`
Specifies the communication protocol and operational mode for the MCP server.
-   **`stdio`** (Default): The server communicates over standard input/output channels. This is typically used when the server is managed as a child process by another application (e.g., Claude Desktop).
-   **`http-sse`**: The server operates as an HTTP server using Server-Sent Events (SSE) for communication. This mode allows the server to be accessed over a network.

#### HTTP-SSE Specific Variables
These variables are applicable only if `MCP_SERVER_TYPE` is set to `http-sse`.

##### `MCP_HTTP_HOST`
-   **Description**: Specifies the desired hostname or IP address for the HTTP-SSE server. The application reads this variable, and it's logged at startup. However, `FastMCP` might use its own defaults or other configuration mechanisms for actual binding when `transport='sse'` is used. Users should check server logs for the actual listening address and test if setting this variable influences the binding.
-   **Default**: `localhost`
-   **Usage**: Set this to `0.0.0.0` to make the server accessible from other machines on the network, or a specific IP address to bind to that interface.

##### `MCP_HTTP_PORT`
-   **Description**: Specifies the desired port number for the HTTP-SSE server. The application reads this variable, and it's logged at startup. Similar to `MCP_HTTP_HOST`, `FastMCP` might use its own defaults or other configuration mechanisms. Users should check server logs for the actual listening port and test if setting this variable influences the binding.
-   **Default**: `8080`
-   **Usage**: Ensure this port is not in use by another application.

**Note**: The `FastMCP` library is responsible for handling the HTTP-SSE transport. While `MCP_HTTP_HOST` and `MCP_HTTP_PORT` are read by this application, their effect on `FastMCP`'s default SSE behavior is pending further confirmation. The server logs will indicate the host/port values intended by these variables.

## Configuration Sources

You can provide these variables in two ways:

1. Environment File (.env):
```env
NOTION_API_KEY=ntn_your_integration_token_here
NOTION_PARENT_PAGE_ID=your_page_id_here
NOTION_DATABASE_ID=your_database_id_here
```

2. Claude Desktop MCP Config:
```json
{
  "mcpServers": {
    "notion-api": {
      "command": "/path/to/your/.venv/bin/python",
      "args": ["-m", "notion_api_mcp"],
      "env": {
        "NOTION_API_KEY": "ntn_your_integration_token_here",
        "NOTION_PARENT_PAGE_ID": "your_page_id_here",
        "NOTION_DATABASE_ID": "your_database_id_here"
      }
    }
  }
}
```

The server will:
1. First load any values from your .env file
2. Then apply any system environment variables (including those from MCP config)
3. System environment variables take precedence over .env values

This means you can:
- Use .env for local development and testing
- Override values via MCP config for production use
- Mix and match sources (e.g., some values in .env, others in MCP config)