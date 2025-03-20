# OMDL Configuration Reference

## General Configuration

| Option | Description | Default Value |
|--------|-------------|---------------|
| `title` | Base name for output files/sheets | “datalayer” |
| `retries` | Number of times to retry failed actions | 2 |
| `default_timeout` | How long to wait for elements to appear (seconds) | 10 |
| `default_delay` | Default waiting time between steps (seconds) | 1 |
| `debug_mode` | Adds debugging info to output file in a separate sheet | false |
| `include_selenium_info` | Add “Selenium” to user agent | false |
| `bot_info` | Add “?bot=true” parameter to URLs | false |
| `output_destination` | Where to save results ("excel" or "google_sheets") | "excel" |
| `output_folder` | Directory where output files will be saved | Current directory |

## Script Blocking

| Option | Description | Default Value |
|--------|-------------|---------------|
| `block_ga4` | Block Google Analytics requests | false |
| `block_gtm` | Block Google Tag Manager requests | false |
| `block_piwik` | Block Piwik PRO scripts | false |
| `block_domains` | List of domains to block requests | null (no domains) |
| `css_elements_to_hide` | CSS selectors of elements to hide | null (so selectors) |

## Event Tracking

| Option | Description | Default Value |
|--------|-------------|---------------|
| `track_events` | List of dataLayer event names to track | null (track all events) |
| `user_agents` | List of user agents to randomly select from | Chrome 91 on Windows 10 |

## Google Sheets Options

| Option | Description | Default Value |
|--------|-------------|---------------|
| `credentials_location` | Where to look for Google API credentials ("file" or "env") | "file" |
| `credentials_path` | Path to Google API credentials file | "credentials.json" |
| `token_location` | Where to store/look for auth token ("file" or "env") | "file" |
| `folder_id` | Google Drive folder ID for saving files | null (root directory) |

## Step Configuration

### Visit Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "visit" | Yes |
| `url` | URL to visit (string or array of URLs) | Only for first visit |
| `delay_after` | Delay after step execution (seconds) | No |

### Click Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "click" | Yes |
| `clicks` | Array of click definitions | Yes |
| `delay_after` | Delay after step execution (seconds) | No |

Click definition options:
- `xpath` or `selector`: Element locator (one required)
- `delay_after`: Delay after individual click (optional)

### Form Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "form" | Yes |
| `fields` | Array of form field definitions | Yes |
| `submit_button` | XPath of submit button | Yes |
| `submit_method` | "selenium" for a default submit method, "js" and "action" as alternatives when "selenium" fails | No |
| `delay_after` | Delay after step execution (seconds) | No |

Field definition options:
- `xpath` or `selector`: Element locator (one required)
- `value`: Value to input

### Scroll Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "scroll" | Yes |
| `selector` or `xpath` | Element to scroll to | No* |
| `pixels` | Number of pixels to scroll | No* |
| `percentage` | Percentage of page to scroll (0-100) | No* |
| `delay_after` | Delay after step execution (seconds) | No |

*One of `selector`, `xpath`, `pixels`, or `percentage` is required

## Sequence Configuration
| Parameter | Description | Required |
|-----------|-------------|----------|
| `steps` | Array of step names to execute in order | Yes |

## Validation Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `code` | A block of the syntaxed, JSON-ish declaration of expected parameters and values | Yes |

### Validation syntax:
- `!` before parameter name means it's required (`!event_name: "view_item"`)
- expected literal values: as in dataLayer (`quantity: 1`, `currency: "USD"`)
- regex (full match): between two `/`; use Python regex syntax (`city: /Paris|London/`)
- define data types: <str> for strings, <int> for integers, <float> for numbers with decimals or <bool> for true/false values - booleans (`price: <float>`)
- quotes for parameter names (keys) are optional (but allowed)
- indentation is also optional
- commas after *key: value* pairs are optional as well


The syntax was designed to let you copy the existing DL value (from a console or from your documentation) and to require only minimal adjustments. You don't have to bother with a proper JSON formatting.