# Proposal: Add `array` and `object` Types to Feature Options Schema

## Problem Statement

The current Dev Container Feature Options Schema lacks support for `array` and `object` types. This limitation hinders the ability to configure features that require lists of values (e.g., dependencies, ports) or key-value mappings (e.g., environment variables, custom settings). Addressing this limitation will enhance the flexibility and usability of the schema.

This proposal aims to address [Issue #567](https://github.com/devcontainers/spec/issues/567).

## Proposed Solution

Extend the Dev Container Feature Options Schema by introducing `array` and `object` as valid types for feature options. These new types will enable developers to create more complex and customizable features.

### Schema Changes

1. **Array Type**:
   - Represents a list of values (e.g., strings, numbers, booleans, or objects).
   - Supports optional constraints such as allowed item types and default values.

2. **Object Type**:
   - Represents a mapping of key-value pairs.
   - Allows nested structures and supports default values for specific keys.

### Example Feature Using `array` and `object`

Below are two examples of how `array` and `object` types can be used in feature configurations.

#### Example 1: Environment Variables

**Feature Definition (`example-feature/devcontainer-feature.json`):**

```json
{
  "name": "example-feature",
  "id": "example",
  "description": "A feature that demonstrates the use of array and object types.",
  "options": {
    "dependencies": {
      "type": "array",
      "description": "List of software packages to install",
      "default": ["git", "curl", "nodejs"]
    },
    "envVariables": {
      "type": "object",
      "description": "Environment variables for the container",
      "default": {
        "NODE_ENV": "development",
        "APP_PORT": "3000"
      }
    }
  },
  "install": {
    "script": "./install.sh",
    "args": []
  }
}
```

**Installation Script (`example-feature/install.sh`):**

```bash
#!/bin/bash

# Install dependencies
echo "Installing dependencies..."
for package in "${DEPENDENCIES[@]}"; do
    sudo apt-get install -y "$package"
done

# Configure environment variables
echo "Setting up environment variables..."
for key in "${!ENVVARIABLES[@]}"; do
    export "$key=${ENVVARIABLES[$key]}"
done

echo "Feature setup complete."
```

#### Example 2: HTTP Headers

**Feature Definition (`example-feature/devcontainer-feature.json`):**

```json
{
  "name": "example-feature",
  "id": "example-http",
  "description": "A feature that demonstrates the use of HTTP headers in a configuration.",
  "options": {
    "httpHeaders": {
      "type": "object",
      "description": "Custom HTTP headers for the development environment",
      "default": {
        "Authorization": "Bearer example-token",
        "Content-Type": "application/json",
        "X-Custom-Header": "DevContainerFeature"
      }
    }
  },
  "install": {
    "script": "./install-http.sh",
    "args": []
  }
}
```

**Installation Script (`example-feature/install-http.sh`):**

```bash
#!/bin/bash

# Apply HTTP headers
echo "Setting up HTTP headers..."
for key in "${!HTTPHEADERS[@]}"; do
    echo "Header: $key -> Value: ${HTTPHEADERS[$key]}"
    # Example usage: Write headers to a configuration file
    echo "$key: ${HTTPHEADERS[$key]}" >> ./http-headers-config.txt
done

echo "Feature setup complete."
```

### User Configuration Example

Below is an example of how a user can configure these features in their `devcontainer.json` file:

```json
{
  "features": {
    "ghcr.io/devcontainers/features/example:1": {
      "dependencies": ["python3", "pip", "make"],
      "envVariables": {
        "NODE_ENV": "production",
        "APP_PORT": "8080",
        "DEBUG": "true"
      }
    },
    "ghcr.io/devcontainers/features/example-http:1": {
      "httpHeaders": {
        "Authorization": "Bearer user-provided-token",
        "X-Custom-Header": "CustomValue",
        "Accept-Language": "en-US"
      }
    }
  }
}
```

### Implementation Steps

1. Modify the JSON schema to support the `array` and `object` types for feature options.
2. Add validation logic to ensure proper usage of the new types.
3. Update documentation to include guidance on defining and using `array` and `object` types.
4. Test and validate the functionality with various feature configurations.

## Impact and Risks

### Benefits

1. Enhances the flexibility of feature configurations, enabling developers to define richer and more powerful features.
2. Simplifies user workflows by reducing the need for workarounds involving concatenated strings, triple escaping quotes or other hacks.

### Risks

1. Increased complexity in validation logic for feature options.
2. Potential learning curve for users unfamiliar with JSON schema concepts.

## Acceptance Criteria

1. `array` and `object` types are supported in the Feature Options Schema.
2. Example features using these types are implemented and published.
3. Comprehensive tests and documentation are provided for the new functionality.
