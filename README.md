# Gyronics Wearable Protocol

This repository contains the Protocol Buffers definition (`gyronics_protobuf.proto`) for the Gyronics Wearable communication protocol. This protocol defines the message structure for communication between the wearable device and a host application (e.g., mobile app, desktop SDK).

## Overview

The protocol is designed around a single `Envelope` message that encapsulates all communication. It supports:
- **Handshake**: Version negotiation and device identification.
- **Control Plane**: Request/Response pattern for system commands, sensor configuration, and device settings.
- **Data Plane**: High-frequency streaming of sensor data (IMU, Gestures).

## File Structure

- `gyronics_protobuf.proto`: The core protocol definition.

## Protocol Details

### Transport Layer

Messages are framed with a 4-byte big-endian length prefix followed by the serialized `Envelope` protobuf message.

```
[uint32_be length][Envelope bytes]
```

### Envelope

The `Envelope` message is the top-level container.

- `sequence_number`: Monotonic counter for debugging/tracing.
- `sent_at_unix_ms`: Sender timestamp for observability.
- `payload`: One of `hello`, `hello_ack`, `request`, `response`, or `event`.

### Handshake

1.  **Client** sends `Hello` with `schema_version` and `app_id`.
2.  **Server** responds with `HelloAck` containing `schema_version`, `server_version`, and `device_info`.

*Note: No other messages are valid until the handshake is complete.*

### Control Plane

Requests and Responses are correlated via a `request_id`.

**Requests (`Request`):**
- **System**: `Ping`, `GetBattery`, `GetDeviceInfo`.
- **Sensor**: `Subscribe`, `Unsubscribe`.
- **Config**: `UseBLE`.

**Responses (`Response`):**
- Matches the `request_id` of the request.
- Contains either a specific result (System, Sensor, Config) or an `Error`.

### Data Plane

Streaming data is sent as `StreamEvent` messages.

- `stream_id`: Identifies the subscription.
- `uptime_us`: Device uptime in microseconds (for alignment).
- `seq`: Per-stream sequence number.
- `data`:
    - `ImuSample`: Accelerometer, Gyroscope, Magnetometer, and fused orientation (Roll, Pitch, Yaw).
    - `GestureUpdate`: Detected gesture name and confidence.

## Usage

To generate code for your language of choice, use `protoc`:

```bash
# Example for Python
protoc --python_out=. gyronics_protobuf.proto

# Example for Go
protoc --go_out=. gyronics_protobuf.proto
```

## License

See [LICENSE](LICENSE) file.
