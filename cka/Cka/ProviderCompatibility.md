# Provider Compatibility

## Content

### ❓ How does the 'parameters' block vary across different CSI plugins and what responsibility does this place on administrators?
The `parameters` block is for plugin-specific values and is different for every plugin. Most storage systems have their own features, and it's your responsibility to read the documentation for your plugin and configure it properly.

