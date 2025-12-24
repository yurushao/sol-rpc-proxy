# RPC Router

A high-performance HTTP router for Solana RPC requests with API key authentication, weighted load balancing, and method-based routing.

## Features

- **API Key Authentication**: Validates requests using query parameter `?api-key=`
- **Weighted Load Balancing**: Distribute requests across multiple backends with configurable weights
- **Method-Based Routing**: Route specific RPC methods to designated backends
- **Request Logging**: Logs request information including RPC method, path, client IP, and duration

## Configuration

The router uses a TOML configuration file specified via command-line argument.

### TOML Configuration

1. Copy the example configuration:
   ```bash
   cp config.example.toml config.toml
   ```

2. Edit `config.toml` with your settings:
   ```toml
   # Server port
   port = 28899

   # API keys for authentication
   api_keys = ["key1", "key2", "key3"]

   # Backend RPC endpoints with weights
   [[backends]]
   url = "https://api.mainnet-beta.solana.com"
   weight = 2

   [[backends]]
   url = "https://solana-api.com"
   weight = 1

   # Method-specific routing overrides (optional)
   [method_routes]
   getProgramAccountsV2 = "https://mainnet.helius-rpc.com/?api-key=<helius-api-key>"
   ```

### Weighted Load Balancing

Backends are selected randomly based on their configured weights:
- **Weight 2**: Gets 2x more requests than weight 1
- **Weight 3**: Gets 3x more requests than weight 1
- **Example**: Weights [2, 3, 1] result in distribution [33.3%, 50%, 16.7%]

### Method-Based Routing

Override the weighted selection for specific RPC methods:
- Define method → backend mappings in `[method_routes]`
- **Method names are case-sensitive** - must match exactly what's in the JSON-RPC `"method"` field
- Useful for routing expensive operations to specific providers

## Usage

1. Configure the router (see Configuration section above)

2. Run the router with default config file (`config.toml`):
   ```bash
   cargo run --release
   ```

3. Or specify a custom config file:
   ```bash
   cargo run --release -- --config /path/to/custom-config.toml
   # or short form:
   cargo run --release -- -c /path/to/custom-config.toml
   ```

4. Make requests with your API key:
   ```bash
   curl -X POST -H "Content-Type: application/json" \
     -d '{"jsonrpc":"2.0","id":1,"method":"getEpochInfo"}' \
     "http://localhost:28899?api-key=your-api-key"
   ```

5. Use with Solana CLI:
   ```bash
   solana -u "http://localhost:28899?api-key=your-api-key" epoch-info
   ```
