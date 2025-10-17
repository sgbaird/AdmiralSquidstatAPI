# Integration Guide for SquidStat API with Other Devices

This guide provides recommendations and best practices for integrating the SquidStat API into automated electrochemistry workflows and with other laboratory devices.

## Table of Contents

1. [Data Capture Patterns](#data-capture-patterns)
2. [Integration Architectures](#integration-architectures)
3. [Existing Workflow References](#existing-workflow-references)
4. [Best Practices](#best-practices)
5. [Special Considerations](#special-considerations)

## Data Capture Patterns

### 1. CSV File Output

The most straightforward approach for capturing SquidStat data is writing to CSV files using signal handlers.

**Example Pattern:**
- Use Qt signals (`activeDCDataReady`, `activeACDataReady`) to capture real-time data
- Write data to CSV files with appropriate headers
- See: `examples/Python/writeInCsv.py` and `examples/Python/dataOutput.py`

**When to use:**
- Offline analysis workflows
- Long-term data storage
- Simple integration requirements

### 2. Serial Communication

Integrate SquidStat with other devices via serial ports for parallel operations.

**Example Pattern:**
- Use `QSerialPort` for communication with other devices
- Trigger actions on other devices when experiment elements start
- Synchronize operations using Qt signals
- See: `examples/Python/writeInCsv.py` (SerialPortReader class)

**When to use:**
- Direct device-to-device communication
- Low-latency requirements
- Embedded systems integration

### 3. TCP/IP Network Communication

Enable remote control and data streaming over network connections.

**Example Pattern:**
- Server connects to SquidStat and accepts client connections
- Clients send commands (start/stop experiments) via TCP
- Server streams data back to clients in real-time
- See: `examples/Python/RemoteSquidstatExample/tcpServer.py` and `tcpClient.py`

**When to use:**
- Multi-user access
- Remote laboratory operations
- Integration with web-based dashboards
- Distributed computing workflows

### 4. Asynchronous/Non-blocking Operations

Use Qt's signal-slot mechanism for event-driven programming.

**Example Pattern:**
- Run experiments without blocking the main thread
- Handle multiple concurrent operations
- React to events (new element starting, data ready, experiment stopped)
- See: `examples/Python/nonblockingExperiment.py`

**When to use:**
- Complex multi-step workflows
- Parallel device operations
- Real-time feedback loops

## Integration Architectures

### Three-Layer Architecture for Self-Driving Labs

Modern electrochemistry automation follows a three-layer architecture:

```
┌─────────────────────────────────────┐
│      AI Planner / Optimization      │
│   (Bayesian optimization, ML)       │
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│   Controller Software / Orchestrator│
│   (ChemOS, custom Python scripts)   │
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│    Hardware Layer (SquidStat API)   │
│  (Potentiostat, pumps, robots)      │
└─────────────────────────────────────┘
```

**Components:**

1. **AI Planner**: Designs and optimizes experiments
   - Uses experimental results to inform next experiments
   - Implements Bayesian optimization or other ML algorithms
   
2. **Controller Software**: Orchestrates hardware and data flow
   - Manages device communication
   - Handles data storage and processing
   - Implements safety checks and error handling
   
3. **Hardware Layer**: Physical devices including SquidStat
   - Direct control via API
   - Real-time data acquisition
   - Error reporting

### Message-Oriented Middleware Pattern

For complex integrations, consider using message brokers:

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│SquidStat │──────│  MQTT    │──────│ Analysis │
│   API    │      │  Broker  │      │  Node    │
└──────────┘      └─────┬────┘      └──────────┘
                        │
                  ┌─────▼────┐
                  │Database  │
                  │ Logger   │
                  └──────────┘
```

**Benefits:**
- Decoupled components
- Scalable architecture
- Multiple subscribers can receive same data
- Persistent message queues

**Protocol Options:**
- MQTT: Lightweight, ideal for IoT
- RabbitMQ: Advanced routing capabilities
- Redis Pub/Sub: Fast, in-memory

## Existing Workflow References

### 1. ChemOS 2.0

Open-source orchestration platform for self-driving laboratories.

**Features:**
- Modular architecture for device integration
- Automated workflow execution
- Built-in optimization algorithms

**Reference:** https://www.cell.com/matter/fulltext/S2590-2385(24)00195-4

### 2. Acceleration Consortium SDL-Demo

Example implementations of self-driving lab workflows for materials discovery.

**Features:**
- Reference implementations in Python
- Integration patterns for electrochemistry
- Bayesian optimization examples

**Reference:** https://github.com/AccelerationConsortium/ac-training-lab

### 3. LabOperator

Commercial lab automation platform with standardized interfaces.

**Features:**
- Centralized data management
- Remote accessibility
- Workflow automation tools

## Best Practices

### 1. Standardized Interfaces

**Use standard communication protocols:**
- REST APIs for web-based integration
- MQTT/OPC for device-to-device communication
- Standard data formats (JSON, CSV, HDF5)

### 2. Modular Design

**Structure code for maintainability:**
- Separate data acquisition from processing
- Use dependency injection for device handlers
- Create reusable components (e.g., data writers, signal handlers)

**Example structure:**
```python
# device_interface.py
class DeviceInterface:
    def connect(self): pass
    def start_experiment(self): pass
    def get_data(self): pass

# squidstat_adapter.py  
class SquidStatAdapter(DeviceInterface):
    # Implements interface for SquidStat
    pass

# workflow.py
def run_workflow(device: DeviceInterface):
    device.connect()
    device.start_experiment()
    return device.get_data()
```

### 3. Centralized Data Management

**Implement structured data storage:**
- Use databases (SQLite, PostgreSQL) for queryable storage
- Implement metadata tracking (experiment parameters, timestamps, device info)
- Version control for data schemas

**Recommended schema elements:**
```
experiments/
  - experiment_id
  - timestamp
  - parameters (JSON)
  - status
  
data_points/
  - experiment_id (foreign key)
  - timestamp
  - channel
  - voltage, current, impedance, etc.
  
devices/
  - device_id
  - device_type
  - calibration_date
```

### 4. Error Handling and Recovery

**Implement robust error handling:**
- Catch and log all API errors
- Implement automatic retry logic for transient failures
- Save partial results before failures
- Use watchdog timers for hung processes

**Example:**
```python
from SquidstatPyLibrary import AisErrorCode

def safe_upload_experiment(handler, channel, experiment, max_retries=3):
    for attempt in range(max_retries):
        error = handler.uploadExperimentToChannel(channel, experiment)
        if error.value() == AisErrorCode.Success:
            return True
        print(f"Attempt {attempt + 1} failed: {error.message()}")
        time.sleep(1)
    return False
```

### 5. Logging and Monitoring

**Implement comprehensive logging:**
- Log all API calls and responses
- Record experiment parameters and outcomes
- Monitor device health metrics
- Track data quality indicators

**Use Python's logging module:**
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('squidstat_integration.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)
logger.info("Experiment started")
```

### 6. Thread Safety

**When using multi-threading:**
- Qt objects should only be accessed from the thread that created them
- Use Qt's `moveToThread()` for worker threads
- Use signals/slots for cross-thread communication
- See: `examples/Python/writeInCsv.py` for thread-safe patterns

### 7. Configuration Management

**Externalize configuration:**
- Use configuration files (YAML, JSON, TOML)
- Separate environment-specific settings
- Version control configuration schemas

**Example config.yaml:**
```yaml
squidstat:
  com_port: "COM4"
  default_channel: 0
  timeout: 30

data_output:
  format: "csv"
  directory: "./data"
  include_metadata: true

integration:
  mqtt_broker: "localhost:1883"
  database_url: "sqlite:///experiments.db"
```

## Special Considerations

### 1. Real-Time Requirements

If your workflow requires real-time responsiveness:
- Use asynchronous patterns (Qt signals/slots)
- Minimize processing in signal handlers
- Consider dedicated threads for time-critical operations
- Test latency under load conditions

### 2. Data Volume Management

For long-running experiments or high sampling rates:
- Implement data buffering
- Use streaming protocols instead of storing all data in memory
- Consider data compression (HDF5, Parquet)
- Implement data rotation/archiving strategies

### 3. Multi-Device Synchronization

When coordinating multiple devices:
- Use a central coordinator/orchestrator
- Implement synchronization primitives (locks, events)
- Define clear state machines for each device
- Handle clock drift for precise timing

**Example synchronization pattern:**
```python
import threading

class ExperimentCoordinator:
    def __init__(self):
        self.start_event = threading.Event()
        
    def start_all_devices(self):
        # Prepare all devices
        squidstat.prepare_experiment()
        pump.prepare()
        
        # Synchronize start
        self.start_event.set()
        
    def device_worker(self, device):
        self.start_event.wait()  # Wait for signal
        device.start()
```

### 4. Safety and Interlock Systems

For automated systems:
- Implement software interlocks (voltage limits, current limits)
- Add emergency stop mechanisms
- Monitor device health (temperature, connection status)
- Log all safety-critical events

### 5. Calibration and Validation

Ensure data quality:
- Regular device calibration procedures
- Validation experiments before automated runs
- Cross-check data consistency
- Document calibration history

### 6. Remote Access Security

When implementing remote control:
- Use encrypted connections (TLS/SSL)
- Implement authentication and authorization
- Limit network exposure (firewalls, VPNs)
- Log all access attempts
- Consider rate limiting

### 7. Documentation and Reproducibility

Maintain scientific reproducibility:
- Record all experiment parameters
- Version control for analysis scripts
- Document API versions and dependencies
- Save raw data alongside processed data
- Include device configuration snapshots

## Example Integration Scenarios

### Scenario 1: Automated Screening Workflow

**Goal:** Screen multiple electrolyte formulations automatically

**Architecture:**
1. Python orchestrator script
2. SquidStat for electrochemical measurements
3. Liquid handling robot for sample preparation
4. Database for results storage

**Implementation approach:**
- Use TCP server pattern for SquidStat control
- Coordinate with liquid handler via serial/HTTP API
- Store results in SQLite database with full metadata
- Generate reports after each batch

### Scenario 2: Closed-Loop Optimization

**Goal:** Optimize battery charging protocols using Bayesian optimization

**Architecture:**
1. Optimization algorithm (Python/BoTorch)
2. SquidStat for charge/discharge cycling
3. Data analysis pipeline
4. Feedback loop

**Implementation approach:**
- Non-blocking experiments for responsive control
- Real-time data streaming for immediate analysis
- Implement safety checks for battery limits
- Use message queue for optimization loop

### Scenario 3: Multi-Site Data Aggregation

**Goal:** Collect data from SquidStats at multiple locations

**Architecture:**
1. Local SquidStat instances with TCP servers
2. Central data aggregation service
3. Cloud database
4. Web dashboard for monitoring

**Implementation approach:**
- Each site runs TCP server example
- Central service polls or receives pushed data
- Standardized data format for aggregation
- Time series database for efficient queries

## Getting Started Checklist

- [ ] Review provided Python examples
- [ ] Choose appropriate data capture pattern for your use case
- [ ] Design your integration architecture
- [ ] Implement error handling and logging
- [ ] Create configuration management system
- [ ] Test with simple experiments first
- [ ] Scale up to full workflow
- [ ] Document your setup
- [ ] Implement monitoring and alerting

## Additional Resources

- **SquidStat API Documentation:** https://admiral-instruments.github.io/AdmiralSquidstatAPI
- **Acceleration Consortium Best Practices:** https://accelerated-discovery.org/t/best-practices-for-automated-electrochemistry-tool-and-workflow-design/451
- **ChemOS 2.0 Paper:** https://www.sciencedirect.com/science/article/pii/S2590238524001954
- **Self-Driving Labs Review:** https://fsk-lab.github.io/doi.org/10.26434/chemrxiv-2024-rj946

## Community and Support

For questions and discussions:
- **GitHub Issues:** https://github.com/Admiral-Instruments/AdmiralSquidstatAPI/issues
- **Acceleration Consortium Forum:** https://accelerated-discovery.org/
- **Admiral Instruments Support:** support@AdmiralInstruments.com

## Contributing

We welcome contributions to this guide! Please see `CommunityGuidelines.md` for contribution guidelines.
