To access and inject runtime environment variables into an NGINX server hosting a SPA (Single Page Application), you can use NGINX's built-in support for environment variables and a simple script to inject these variables at runtime. Here's a step-by-step guide:

**1. Define Environment Variables:**

First, define your runtime environment variables. You can store these variables in a file, and we'll later use a script to read and inject them into NGINX configuration.

```shell
# Example env-vars.conf
set $api_base_url "https://api.example.com";
set $debug_mode "false";
```

**2. Create a Script to Inject Environment Variables:**

Create a script (e.g., `env-injector.sh`) that reads your environment variable file (`env-vars.conf`) and generates NGINX configuration.

```bash
#!/bin/bash

# Read environment variables from a file
source /path/to/env-vars.conf

# Create a temporary NGINX configuration file
cat > /etc/nginx/conf.d/env-vars.conf <<EOF
location / {
    # Inject environment variables into JavaScript code
    add_header Content-Type application/javascript;
    echo "window._env_ = {";
    echo "  API_BASE_URL: '$api_base_url',";
    echo "  DEBUG_MODE: $debug_mode";
    echo "};";
    echo "window.API_BASE_URL = '$api_base_url';";
    echo "window.DEBUG_MODE = $debug_mode;";
    echo "}";
}
EOF

# Test NGINX configuration and reload NGINX
nginx -t && nginx -s reload
```

Make sure to give execute permissions to the script:

```bash
chmod +x env-injector.sh
```

**3. Set Up NGINX Configuration:**

In your NGINX configuration (usually found in `/etc/nginx/nginx.conf` or in separate config files in `/etc/nginx/conf.d/`), include the `env-vars.conf` file:

```nginx
# Include environment variable configuration
include /etc/nginx/conf.d/env-vars.conf;

server {
    # Your server configuration here
}
```

**4. Run the Script at Runtime:**

When you want to inject environment variables into your NGINX server, run the `env-injector.sh` script:

```bash
./env-injector.sh
```

This script will generate a temporary NGINX configuration file that injects the environment variables into your SPA code when a request is made. After running the script, NGINX will reload its configuration to apply the changes.

**5. Access Environment Variables in Your SPA:**

In your SPA code (e.g., a JavaScript file), you can now access the environment variables like this:

```javascript
const apiBaseUrl = window._env_.API_BASE_URL;
const debugMode = window._env_.DEBUG_MODE;
```

These variables will be available to your SPA as global variables, and you can use them as needed.

Remember to secure access to the script that injects environment variables, as it can potentially expose sensitive information. Additionally, automate the script's execution as part of your deployment process or server startup to ensure that the environment variables are always injected correctly.
