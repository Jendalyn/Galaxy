# 🚀 Galaxy
**The Ultra High-Performance BSV Node**

![Rust](https://img.shields.io/badge/Rust-1.88+-orange?logo=rust)
![Build Status](https://github.com/murphsicles/Galaxy/actions/workflows/ci.yml/badge.svg)
![Dependencies](https://img.shields.io/badge/dependencies-up%20to%20date-green)
![License](https://img.shields.io/badge/license-Open%20BSV-blue)

**Galaxy** is an ultra high-performance Bitcoin SV (BSV) node built in Rust, designed to unboundedly scale to asynchronously parallel process over **100,000,000 transactions per second (TPS)** per shard. It leverages a microservices architecture with gRPC for ultra-fast, asynchronous communication, Tiger Beetle DB for scalable UTXO storage, and Rust SV for BSV-specific libraries to power OP_RETURN data, private blockchain overlays, and Simplified Payment Verification (SPV) proofs.

## 🌟 Features

- **High Throughput**: Targets over 100,000,000 TPS with async gRPC, batching, and sharding.
- **BSV-Specific**:
  - Supports unbounded block creation.
  - Handles OP_RETURN data for enterprise applications.
  - Manages private blockchain overlays with persistent storage (sled).
  - Provides SPV proofs with caching and streaming.
  - Supports mining with dynamic difficulty and transaction selection.
- **Security**:
  - Token-based authentication with JWTs.
  - Role-based access control (RBAC) for miners and clients.
  - Rate limiting to prevent abuse (1000 req/s per service).
- **Microservices Architecture**:
  - `network_service`: P2P networking with peer pool management.
  - `transaction_service`: Transaction validation, queuing, and indexing.
  - `block_service`: Block validation, assembly, and indexing with merkle root computation.
  - `storage_service`: UTXO management with Tiger Beetle integration.
  - `consensus_service`: Enforces BSV consensus rules.
  - `overlay_service`: Manages private blockchains with streaming, storage, and indexing.
  - `validation_service`: Generates/verifies SPV proofs with caching and streaming.
  - `mining_service`: Supports block mining with streaming and transaction selection.
  - `auth_service`: Handles authentication and authorization.
  - `alert_service`: Monitors network health and sends notifications.
  - `index_service`: Indexes transactions and blocks for efficient querying.
- **Performance Optimizations**:
  - Asynchronous gRPC calls with connection keep-alive.
  - Batching for transactions, blocks, and UTXOs.
  - Lean Protocol Buffer messages with 4GB buffer hints for large blocks.
  - Transaction queuing and SPV proof caching.
  - Scalable UTXO storage with Tiger Beetle DB.
  - Prometheus metrics for performance monitoring.
  - Structured logging with tracing for observability.
  - Sharding for distributed processing.
- **Monitoring**:
  - Alert notifications for critical events (e.g., consensus violations, performance issues).
  - Transaction and block indexing for efficient querying.
  - Prometheus metrics for TPS, latency, and errors across all services.

## 🛠️ Setup

### Prerequisites
- Rust 1.88+ (stable)
- Cargo
- Tiger Beetle server (for `storage_service`, see [Tiger Beetle](https://github.com/tigerbeetle/tigerbeetle))
- `grpcurl` for testing
- `sled` for overlay and index storage
- `prometheus`, `governor`, `jsonwebtoken`, and `tracing` for metrics, rate limiting, auth, and logging

### Installation
```bash
git clone https://github.com/murphsicles/Galaxy.git
cd Galaxy
cargo build
```

### Running Services
Start each service in a separate terminal:
```bash
cd network_service
cargo run
```
```bash
cd transaction_service
cargo run
```
```bash
cd block_service
cargo run
```
```bash
cd storage_service
cargo run
```
```bash
cd consensus_service
cargo run
```
```bash
cd overlay_service
cargo run
```
```bash
cd validation_service
cargo run
```
```bash
cd mining_service
cargo run
```
```bash
cd auth_service
cargo run
```
```bash
cd alert_service
cargo run
```
```bash
cd index_service
cargo run
```

### Tiger Beetle Setup
For `storage_service`, start a Tiger Beetle server:
```bash
tigerbeetle start --cluster=0 --replica=0
```

### Authentication Setup
Configure a secure JWT secret key in `tests/config.toml` under `[auth]`. Generate tokens with roles (`client`, `miner`) for testing.

## 🧪 Testing

Test services using `grpcurl` with JWT tokens in the `authorization` header. Ensure all services are running on their respective ports:
- `network_service`: `localhost:50051`
- `transaction_service`: `localhost:50052`
- `storage_service`: `localhost:50053`
- `block_service`: `localhost:50054`
- `consensus_service`: `localhost:50055`
- `overlay_service`: `localhost:50056`
- `validation_service`: `localhost:50057`
- `mining_service`: `localhost:50058`
- `auth_service`: `localhost:50060`
- `alert_service`: `localhost:50061`
- `index_service`: `localhost:50062`

### Testnet Integration
Galaxy is configured to connect to BSV testnet nodes. See `tests/config.toml` for settings:
- Testnet nodes for `network_service`.
- Tiger Beetle server address for `storage_service`.
- Sharding parameters and node mappings.
- Authentication settings.
- Alert and indexing configurations.
- Overlay consensus rules (e.g., `restrict_op_return`).
- Test cases for all services.

Run the full pipeline test:
```bash
cd tests
chmod +x run_tests.sh
./run_tests.sh
```

### Metrics
Monitor performance using the `GetMetrics` endpoint on each service (e.g., `localhost:50057` for `validation_service`):
```bash
grpcurl -plaintext -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMSIsInJvbGUiOiJjbGllbnQiLCJleHAiOjE5MjA2NzY1MDl9.8X8z7z3Y8Qz5z5z7z3Y8Qz5z5z7z3Y8Qz5z5z7z3Y8Q" -d '{}' localhost:50057 validation.Validation/GetMetrics
```
Metrics include:
- `*_requests_total`: Total requests per service (e.g., `block_requests_total`).
- `*_latency_ms`: Average request latency (ms).
- `*_alert_count`: Total alerts sent.
- `*_index_throughput`: Indexed items per second (where applicable).
- `*_cache_hits`: Cache hits (e.g., `validation_cache_hits`).
- `errors_total`: Total errors (placeholder, currently 0).

### Alerts
Subscribe to alerts using the `alert_service`:
```bash
grpcurl -plaintext -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMSIsInJvbGUiOiJjbGllbnQiLCJleHAiOjE5MjA2NzY1MDl9.8X8z7z3Y8Qz5z5z7z3Y8Qz5z5z7z3Y8Qz5z5z7z3Y8Q" -d '{"event_type": "consensus_violation"}' localhost:50061 alert.Alert/SubscribeToAlerts
```

### Logging
View logs with the `tracing` crate at the `INFO` level (configurable in `tests/config.toml` under `[metrics][log_level]`).

See individual service READMEs for detailed test commands.

## 🔄 CI/CD

Galaxy uses GitHub Actions for continuous integration and deployment:
- **Build**: Compiles the project in release mode.
- **Formatting**: Ensures code adheres to `rustfmt` standards.
- **Linting**: Runs `clippy` for code quality.
- **Dependency Checks**: Validates dependencies with `cargo outdated`.

Check the [CI workflow](.github/workflows/ci.yml) for details.

## 📈 Performance Highlights

Galaxy is optimized for ultra-high performance:
- **Asynchronous gRPC**: Non-blocking calls with connection keep-alive.
- **Batching**: Reduces network overhead for transactions, blocks, and UTXOs.
- **Tiger Beetle DB**: Scalable UTXO storage for BSV’s future dataset.
- **Transaction Queuing**: Handles high transaction volumes efficiently.
- **SPV Proof Caching**: Optimizes proof generation with LRU cache.
- **Streaming**: Supports continuous transaction, proof, and mining work processing.
- **Metrics**: Monitors TPS, latency, and errors with Prometheus.
- **Logging**: Structured logging with `tracing` for observability.
- **Sharding**: Distributed processing with shard-aware logic.
- **Security**: JWT authentication and RBAC for secure access.
- **Rate Limiting**: Prevents endpoint abuse with 1000 req/s limit.
- **Alerting**: Real-time notifications for network health.
- **Indexing**: Efficient querying for transactions and blocks.

## 📚 Project Structure

| Directory            | Description                          |
|----------------------|--------------------------------------|
| `network_service/`   | P2P networking for BSV nodes         |
| `transaction_service/`| Transaction validation, queuing, indexing   |
| `block_service/`     | Block validation, assembly, indexing        |
| `storage_service/`   | UTXO storage with Tiger Beetle       |
| `consensus_service/` | BSV consensus rule enforcement       |
| `overlay_service/`   | Private blockchain overlays          |
| `validation_service/`| SPV proof generation and verification|
| `mining_service/`    | Block mining support                 |
| `auth_service/`      | Authentication and authorization      |
| `alert_service/`     | Network health monitoring and notifications |
| `index_service/`     | Transaction and block indexing       |
| `shared/`            | Shared utilities (ShardManager)      |
| `protos/`            | gRPC proto files for services        |
| `tests/`             | Test configuration and scripts       |
| `Cargo.toml`         | Workspace configuration              |

## 🤝 Contributing

Contributions are welcome! Please open an issue or pull request on GitHub.


## 📝 License

Licensed under the Open BSV License. See [LICENSE](LICENSE) for details.

## 📬 Contact

For questions, reach out to **murphsicles** via GitHub issues.

🌌 **Galaxy: Powering the future of BSV with unmatched performance!**
