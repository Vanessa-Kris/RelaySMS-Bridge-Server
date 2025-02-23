# Bridge Specification

## Table of Contents

- [Content Format](#content-format)
- [Payload Format](#payload-format)
  - [Auth Request Payload](#auth-request-payload)
  - [Auth Code Payload](#auth-code-payload)
  - [Auth Code and Payload](#auth-code-and-payload)
  - [Payload Only](#payload-only)

## Content Format

The Bridge supports the following content formats:

1. **Email**: `to:cc:bcc:subject:body`
   - Example: Email Bridge

## Payload Format

| **Payload Type**                                | **Switch** | **Description**                                       |
| ----------------------------------------------- | ---------- | ----------------------------------------------------- |
| [Auth Request Payload](#auth-request-payload)   | `0`        | Contains a client public key                          |
| [Auth Code Payload](#auth-code-payload)         | `1`        | Contains an authentication code                       |
| [Auth Code and Payload](#auth-code-and-payload) | `2`        | Contains an authentication code and encrypted content |
| [Payload Only](#payload-only)                   | `3`        | Contains encrypted content only                       |

### Auth Request Payload

- **Switch**: `0`
- **Format**:
  - 1 byte: Content Switch
  - 4 bytes: Public Key Length (integer)
  - Variable: Client Public Key

#### Visual Representation:

```
+------------------+----------------------------+-------------------+
| Content Switch   | Length of Client Public Key| Client Public Key |
| (1 byte)         | (4 bytes, integer)         | (variable size)   |
+------------------+----------------------------+-------------------+
```

```python
content_switch = bytes([0])  # b'\x00'
client_public_key = b"client_pub_key"

payload = (
    content_switch +
    struct.pack("<i", len(client_public_key)) +
    client_public_key
)
encoded = base64.b64encode(payload).decode("utf-8")
print(encoded)
```

### Auth Code Payload

- **Switch**: `1`
- **Format**:
  - 1 byte: Content Switch
  - 1 byte: Auth Code Length (integer)
  - Variable: Auth Code

#### Visual Representation:

```
+------------------+---------------------------+-----------------+
| Content Switch   | Length of Auth Code       | Auth Code       |
| (1 byte)         | (1 byte, integer)         | (variable size) |
+------------------+---------------------------+-----------------+
```

```python
content_switch = bytes([1])  # b'\x01'
auth_code = b"123456"

payload = (
    content_switch +
    bytes([len(auth_code)]) +
    auth_code
)
encoded = base64.b64encode(payload).decode("utf-8")
print(encoded)
```

### Auth Code and Payload

- **Switch**: `2`
- **Format**:
  - 1 byte: Content Switch
  - 1 byte: Auth Code Length (integer)
  - 4 bytes: Ciphertext Length (integer)
  - 1 byte: Bridge Letter
  - Variable: Auth Code
  - Variable: Ciphertext

> [!NOTE]
>
> For detailed instructions on using the Double Ratchet algorithm to create ciphertext, refer to the [smswithoutborders_lib_sig documentation](https://github.com/smswithoutborders/lib_signal_double_ratchet_python?tab=readme-ov-file#double-ratchet-implementations).

#### Visual Representation:

```
+------------------+---------------------------+----------------------------+-------------------+------------------+----------------------------+
| Content Switch   | Length of Auth Code       | Length of Ciphertext       | Bridge Letter     | Auth Code        | Ciphertext                 |
| (1 byte)         | (1 byte, integer)         | (4 bytes, integer)         | (1 byte)          | (variable size)  | (variable size)            |
+------------------+---------------------------+----------------------------+-------------------+------------------+----------------------------+
```

```python
content_switch = bytes([2])  # b'\x02'
auth_code = b"123456"
bridge_letter = b"e"
ciphertext = b"Hello world!"

payload = (
    content_switch +
    bytes([len(auth_code)]) +
    struct.pack("<i", len(ciphertext)) +
    bridge_letter +
    auth_code +
    ciphertext
)
encoded = base64.b64encode(payload).decode("utf-8")
print(encoded)
```

### Payload Only

- **Switch**: `3`
- **Format**:
  - 1 byte: Content Switch
  - 4 bytes: Ciphertext Length (integer)
  - 1 byte: Bridge Letter
  - Variable: Ciphertext

> [!NOTE]
>
> For detailed instructions on using the Double Ratchet algorithm to create ciphertext, refer to the [smswithoutborders_lib_sig documentation](https://github.com/smswithoutborders/lib_signal_double_ratchet_python?tab=readme-ov-file#double-ratchet-implementations).

#### Visual Representation:

```
+------------------+-----------------------------+------------------+----------------------------+
| Content Switch   | Length of Ciphertext        | Bridge Letter    | Ciphertext                 |
| (1 byte)         | (4 bytes, integer)          | (1 byte)         | (variable size)            |
+------------------+-----------------------------+------------------+----------------------------+
```

```python
content_switch = bytes([3])  # b'\x03'
bridge_letter = b"e"
ciphertext = b"Hello world!"

payload = (
    content_switch +
    struct.pack("<i", len(ciphertext)) +
    bridge_letter +
    ciphertext
)
encoded = base64.b64encode(payload).decode("utf-8")
print(encoded)
```
