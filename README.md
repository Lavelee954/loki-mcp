# Loki MCP Server

A Go-based server implementation for the Model Context Protocol (MCP) with Grafana Loki integration.

## Getting Started

### Prerequisites

- Go 1.16 or higher

### Building and Running

Build and run the server:

```bash
# Build the server
go build -o loki-mcp-server ./cmd/server

# Run the server
./loki-mcp-server
```

Or run directly with Go:

```bash
go run ./cmd/server
```

By default, the server runs on port 8080. You can change this by setting the PORT environment variable:

```bash
PORT=3000 go run ./cmd/server
```

## Project Structure

```
.
├── cmd/
│   ├── server/       # MCP server implementation
│   └── client/       # Client for testing the MCP server
├── internal/
│   ├── handlers/     # HTTP handlers
│   └── models/       # Data models
├── pkg/
│   └── utils/        # Utility functions and shared code
└── go.mod            # Go module definition
```

## MCP Server

The Loki MCP Server implements the Model Context Protocol (MCP) and provides the following tools:

### Loki Query Tool

The `loki_query` tool allows you to query Grafana Loki log data:

- Required parameters:
  - `query`: LogQL query string

- Optional parameters:
  - `url`: The Loki server URL (default: from LOKI_URL environment variable or http://localhost:3100)
  - `start`: Start time for the query (default: 1h ago)
  - `end`: End time for the query (default: now)
  - `limit`: Maximum number of entries to return (default: 100)

#### Environment Variables

The Loki query tool supports the following environment variables:

- `LOKI_URL`: Default Loki server URL to use if not specified in the request

### Testing the MCP Server

You can test the MCP server using the provided client:

```bash
# Build the client
go build -o loki-mcp-client ./cmd/client

# Loki query examples:
./loki-mcp-client loki_query "{job=\"varlogs\"}"
./loki-mcp-client loki_query "http://localhost:3100" "{job=\"varlogs\"}"
./loki-mcp-client loki_query "{job=\"varlogs\"}" "-1h" "now" 100

# Using environment variable:
export LOKI_URL="http://localhost:3100"
./loki-mcp-client loki_query "{job=\"varlogs\"}"
```

## Docker Support

You can build and run the MCP server using Docker:

```bash
# Build the Docker image
docker build -t loki-mcp-server .

# Run the server
docker run --rm -i loki-mcp-server
```

Alternatively, you can use Docker Compose:

```bash
# Build and run with Docker Compose
docker-compose up --build
```

### Local Testing with Loki

The project includes a complete Docker Compose setup to test Loki queries locally:

1. Start the Docker Compose environment:
   ```bash
   docker-compose up -d
   ```

   This will start:
   - A Loki server on port 3100
   - A Grafana instance on port 3000 (pre-configured with Loki as a data source)
   - A log generator container that sends sample logs to Loki
   - The Loki MCP server

2. Use the provided test script to query logs:
   ```bash
   # Make it executable
   chmod +x test-loki-query.sh
   
   # Run with default parameters (queries last 15 minutes of logs)
   ./test-loki-query.sh
   
   # Query for error logs
   ./test-loki-query.sh '{job="varlogs"} |= "ERROR"'
   
   # Specify a custom time range and limit
   ./test-loki-query.sh '{job="varlogs"}' '-1h' 'now' 50
   ```

3. Access the Grafana UI at http://localhost:3000 to explore logs visually.

## Architecture

The Loki MCP Server uses a modular architecture:

- **Server**: The main MCP server implementation in `cmd/server/main.go`
- **Client**: A test client in `cmd/client/main.go` for interacting with the MCP server
- **Handlers**: Individual tool handlers in `internal/handlers/`
  - `calculator.go`: Basic arithmetic operations
  - `loki.go`: Grafana Loki query functionality

## Using with Claude Desktop

You can use this MCP server with Claude Desktop to add calculator and Loki query tools. Follow these steps:

### Option 1: Using the Compiled Binary

1. Build the server:
```bash
go build -o loki-mcp-server ./cmd/server
```

2. Add the configuration to your Claude Desktop configuration file using `claude_desktop_config_binary.json`.

### Option 2: Using Go Run with a Shell Script

1. Make the script executable:
```bash
chmod +x run-mcp-server.sh
```

2. Add the configuration to your Claude Desktop configuration file using `claude_desktop_config_script.json`.

### Option 3: Using Docker (Recommended)

1. Build the Docker image:
```bash
docker build -t loki-mcp-server .
```

2. Add the configuration to your Claude Desktop configuration file using `claude_desktop_config_docker.json`.

### Configuration Details

The Claude Desktop configuration file is located at:
- On macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- On Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- On Linux: `~/.config/Claude/claude_desktop_config.json`

You can use one of the example configurations provided in this repository:
- `claude_desktop_config.json`: Generic template
- `claude_desktop_config_example.json`: Example using `go run` with the current path
- `claude_desktop_config_binary.json`: Example using the compiled binary
- `claude_desktop_config_script.json`: Example using a shell script (recommended for `go run`)
- `claude_desktop_config_docker.json`: Example using Docker (most reliable)

**Notes**:
- When using `go run` with Claude Desktop, you may need to set several environment variables in both the script and the configuration file:
  - `HOME`: The user's home directory
  - `GOPATH`: The Go workspace directory
  - `GOMODCACHE`: The Go module cache directory
  - `GOCACHE`: The Go build cache directory
  
  These are required to ensure Go can find its modules and build cache when run from Claude Desktop.

- Using Docker is the most reliable approach as it packages all dependencies and environment variables in a container.

Or create your own configuration:

```json
{
  "mcpServers": {
    "lokiserver": {
      "command": "path/to/loki-mcp-server",
      "args": [],
      "env": {
        "LOKI_URL": "http://localhost:3100"
      },
      "disabled": false,
      "autoApprove": ["loki_query"]
    }
  }
}
```

Make sure to replace `path/to/loki-mcp-server` with the absolute path to the built binary or source code.

4. Restart Claude Desktop.

5. You can now use the tools in Claude:
   - Loki query examples:
     - "Query Loki for logs with the query {job=\"varlogs\"}"
     - "Find error logs from the last hour in Loki using query {job=\"varlogs\"} |= \"ERROR\""
     - "Show me the most recent 50 logs from Loki with job=varlogs"

## License

This project is licensed under the MIT License - see the LICENSE file for details.
