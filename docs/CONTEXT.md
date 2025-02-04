# Technical Context

## TunnelConnectionListener

The `TunnelConnectionListener` is a core component that manages connections to a proxy server. It implements `IConnectionListener` and handles the creation, maintenance, and lifecycle of tunnel connections.

### Key Features

1. **Connection Management**
   - Enforces connection limits through a semaphore
   - Tracks active connections using a concurrent dictionary
   - Supports both WebSocket and HTTP/2 transport types

2. **Connection Handling**
   - Implements automatic retry logic for failed connections
   - Manages connection lifecycle and cleanup
   - Provides graceful shutdown capabilities

3. **Transport Configuration**
   - Uses optimized HTTP handler settings for long-lived connections
   - Supports multiple concurrent HTTP/2 connections
   - Configures infinite connection pooling timeouts

### Connection Acceptance

The listener implements a robust `AcceptAsync` flow that:
- Enforces connection limits
- Establishes new connections with retry capability
- Tracks connection lifetime
- Automatically releases resources when connections complete

## TunnelConnectionListenerFactory

The `TunnelConnectionListenerFactory` is a factory class that creates instances of `TunnelConnectionListener`. It implements `IConnectionListenerFactory` to integrate with ASP.NET Core's connection handling infrastructure.

### Responsibilities

- Creates new TunnelConnectionListener instances
- Manages tunnel options configuration
- Binds listeners to specific endpoints

## TunnelOptions

The `TunnelOptions` class provides configuration settings for the tunnel implementation.

### Configuration Properties

1. **MaxConnectionCount**
   - Controls the maximum number of concurrent connections
   - Defaults to 10 connections

2. **Transport**
   - Specifies the transport protocol to use
   - Supports two types:
     - WebSockets
     - HTTP/2 (default)

This configuration system allows for flexible deployment scenarios and can be adjusted based on specific performance and scaling requirements.

## HttpClientConnectionContext

The `HttpClientConnectionContext` class is a crucial component in the transport layer that handles HTTP client connections. It implements several key interfaces to provide a complete connection context:

- `ConnectionContext` - Base functionality for connection handling
- `IConnectionLifetimeFeature` - Manages connection lifecycle
- `IConnectionEndPointFeature` - Handles connection endpoints
- `IConnectionItemsFeature` - Manages connection metadata
- `IConnectionIdFeature` - Provides unique connection identification
- `IConnectionTransportFeature` - Handles transport-level operations
- `IDuplexPipe` - Enables bi-directional communication

### Key Responsibilities

1. **Connection Management**
   - Generates unique connection IDs
   - Maintains connection state and features
   - Handles connection lifecycle (establishment, abortion, disposal)

2. **Transport Layer Integration**
   - Implements duplex pipe functionality for bi-directional communication
   - Manages input/output streams through PipeReader and PipeWriter
   - Handles HTTP/2 connections via HttpMessageInvoker

3. **Connection Features**
   - Provides a feature collection for connection capabilities
   - Manages connection endpoints (local and remote)
   - Handles connection items for metadata storage

### Connection Establishment

The class provides a static `ConnectAsync` method that:
1. Creates an HTTP/2 POST request to the specified URI
2. Establishes a connection context
3. Sets up bi-directional communication channels
4. Handles the HTTP response and stream management

This implementation is particularly useful for scenarios requiring long-lived HTTP connections with bi-directional communication capabilities, such as tunneling or streaming applications.

## WebSocketConnectionContext

The `WebSocketConnectionContext` class extends `HttpConnection` to provide WebSocket-specific connection handling. It implements a specialized connection context for WebSocket-based communication.

### Key Features

1. **WebSocket Management**
   - Manages underlying WebSocket connections
   - Implements keep-alive functionality (5-second interval)
   - Handles WebSocket lifecycle (connection, abortion, disposal)

2. **Connection Configuration**
   - Skips negotiation phase for direct WebSocket connections
   - Uses binary transfer format for efficient data transmission
   - Configures custom WebSocket factory for connection creation

3. **Connection Control**
   - Provides cancellation support for connection termination
   - Implements graceful and forced abortion mechanisms
   - Manages connection disposal and cleanup

### Connection Establishment

The class provides a static `ConnectAsync` method that:
- Creates a new ClientWebSocket instance
- Configures WebSocket options and keep-alive settings
- Establishes the WebSocket connection
- Initializes the connection context

This implementation is particularly suited for scenarios requiring real-time, bi-directional communication over WebSocket protocols, providing a reliable and efficient transport mechanism.

## TrackLifetimeConnectionContext

The `TrackLifetimeConnectionContext` is a wrapper class that decorates an existing `ConnectionContext` to track its lifetime. This class serves a specific monitoring purpose in the transport layer.

### Purpose

- Acts as a decorator for an underlying connection context
- Primarily exists to monitor and track the lifetime of connections
- Provides execution state tracking through a TaskCompletionSource

### Implementation Details

1. **Connection Delegation**
   - Forwards all connection operations to the underlying connection
   - Maintains transparency by implementing the full ConnectionContext interface
   - Preserves all original connection features and capabilities

2. **Lifetime Tracking**
   - Exposes an `ExecutionTask` that completes when the connection is disposed
   - Enables external components to monitor connection lifetime
   - Helps in resource management and connection state observation

3. **Connection Properties**
   - Delegates all property access (ConnectionId, Features, Items, etc.)
   - Maintains connection endpoints (local and remote)
   - Preserves transport layer access

This implementation is particularly useful for scenarios where monitoring connection lifetime is crucial, such as connection pooling, resource cleanup, or diagnostic purposes.
