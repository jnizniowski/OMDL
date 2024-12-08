# OMDL (Oh My dataLayer)

**OMDL** is a script for automated collection of dataLayer events from websites. It simulates user journeys and record selected – or all – DL events to a spreadsheet with all necessary info.

If you regularly go through a website to verify dataLayer events and their content, this script can save you a lot of time. Once configured, it'll do your job in a few minutes. You'll know what, when, and where was fired to the dataLayer.

The most common scenario is **validation of the ecommerce funnel.** With OMDL, you can simulate almost every scenario from visiting a product to a purchase, including sign-in or filling a shipping info to a form.

It can be also configured to randomly examine pages from a list or click random elements (e.g. links) on a website.

OMDL stands for "Oh My dataLayer", but if you need more official name for your boss, feel free to call it _Operational Monitoring of DataLayer_.

## Features

- 🤖 Browser interaction automation (page visits, clicks, form filling, scrolling)
- 📊 Clean reports in a spreadsheet (Excel file or Google Sheets)
- ✨ Easy setup in TOML files – no programming skills required
- 🛠️ Highly configurable (see [Configuration](#Configuration)) yet not demanding
- 🔄 Support for multiple sequences in one configuration
- 🔍 Error handling with human-friendly messages
- 🖥️ Cross-platform - built with Python

## Requirements

### System Requirements

- Python 3.8 or higher
- Google Chrome browser
- Python packages (see [Dependencies](#Dependencies))

If you encounter issues with Chrome/ChromeDriver on macOS, you might need to install Command Line Tools:

```bash
xcode-select --install
```

### Dependencies

To see the list, see requirements.txt file.

You can download them in bulk – download the file and run the command:

```bash
python3 -m pip install -r requirements.txt
```

## Usage

1. Download project files – you need at least:
    - omdl.py 
    - example_config.toml 
    - requirements.txt
2. Create a TOML configuration file (see [Configuration](#Configuration))
3. Install all dependencies
4. Run the script:
```bash
python3 omdl.py ./your-config-file.toml
```

## Configuration

OMDL aims to cover the variety of scenarios, needs, and technical challenges.

That's why OMDL leaves you – The User – **a long list of settings** to let you decide what it should do on your behalf.

The good news is, most of them are optional.

OMDL uses TOML files for configuration. Each configuration file **must** include:

- General configuration (`[config]` section)
- At least one step definition (`[step]` sections)
- At least one sequence definition (`[sequence]` section)

### Basic Configuration Example

The most minimal setup would look like this:

```toml
[config] # no config required!
[step.my_step_name]
type = "visit"
url = "https://example.com"

[sequence.my_sequence_name]
steps = ["homepage"]
```


### General Configuration Options

| Option | Description | Default Value |
|--------|-------------|---------------|
| `title` | Base name for output files/sheets | "datalayer" |
| `retries` | Number of times to retry failed actions | 2 |
| `default_timeout` | How long to wait for elements to appear (seconds) | 10 |
| `default_delay` | Default waiting time between steps (seconds) | 1 |
| `debug_mode` | Adds debugging info to output file in a separate sheet | false |
| `include_selenium_info` | Add "Selenium" to user agent | false |
| `bot_info` | Add "?bot=true" parameter to URLs | false |
| `output_destination` | Where to save results ("excel" or "google_sheets") | "excel" |
| `output_folder` | Directory where output files will be saved | Current directory |

### Script Blocking Options

| Option | Description | Default Value |
|--------|-------------|---------------|
| `block_ga4` | Block Google Analytics requests | false |
| `block_gtm` | Block Google Tag Manager requests | false |
| `block_piwik` | Block Piwik PRO scripts | false |
| `block_domains` | List of domains to block requests | null (no domains) |
| `css_elements_to_hide` | CSS selectors of elements to hide | null (so selectors) |

### Event Tracking Options

| Option | Description | Default Value |
|--------|-------------|---------------|
| `track_events` | List of dataLayer event names to track | null (track all events) |
| `user_agents` | List of user agents to randomly select from | [Chrome 91 on Windows 10] |

### Google Sheets Options

| Option | Description | Default Value |
|--------|-------------|---------------|
| `credentials_location` | Where to look for Google API credentials ("file" or "env") | "file" |
| `credentials_path` | Path to Google API credentials file | "credentials.json" |
| `token_location` | Where to store/look for auth token ("file" or "env") | "file" |
| `folder_id` | Google Drive folder ID for saving files | null (root directory) |

### Step Configuration Options

#### Visit Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "visit" | Yes |
| `url` | URL to visit (string or array of URLs) | Only for first visit |
| `delay_after` | Delay after step execution (seconds) | No |

#### Click Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "click" | Yes |
| `clicks` | Array of click definitions | Yes |
| `delay_after` | Delay after step execution (seconds) | No |

Click definition options:
- `xpath` or `selector`: Element locator (one required)
- `delay_after`: Delay after individual click (optional)

#### Form Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "form" | Yes |
| `fields` | Array of form field definitions | Yes |
| `submit_button` | XPath of submit button | Yes |
| `delay_after` | Delay after step execution (seconds) | No |

Field definition options:
- `xpath` or `selector`: Element locator (one required)
- `value`: Value to input

#### Scroll Steps
| Parameter | Description | Required |
|-----------|-------------|----------|
| `type` | Must be "scroll" | Yes |
| `selector` or `xpath` | Element to scroll to | No* |
| `pixels` | Number of pixels to scroll | No* |
| `percentage` | Percentage of page to scroll (0-100) | No* |
| `delay_after` | Delay after step execution (seconds) | No |

*One of `selector`, `xpath`, `pixels`, or `percentage` is required

### Sequence Configuration
| Parameter | Description | Required |
|-----------|-------------|----------|
| `steps` | Array of step names to execute in order | Yes |


See `example_config.toml` for a complete configuration reference with all available options.
Use that file as a base for your config file.

Don't worry about messing up – the script will verify it and try to point out what should be fixed.
The majority of settings have specified default values, so even if your config is incorrect, the script can use them to run your sequences anyway.
For example, if OMDL can't find your Google credentials file, it will save results in an XLSX file.

> [!IMPORTANT]
> **I highly recommend adding `debug_mode = true` to the config section**, at least while defining steps. Some errors will not stop the script, but appear as warnings in the log.

## How to design sequence steps

The easiest method of creating steps for OMDL is to manually go through the website and record all selectors for clicks and forms and/or URLs for visits. 

To save XPath or CSS selector, you need to use developer tools in your web browser. 
1. Right click on the element you want to click or fill, choose "Inspect".
2. You'll see your element's HTML definition highlighted. Right click on its code -> Copy -> CSS selector / XPath. 

Usually that's enough. But sometimes crafting valid steps may be challenging, especially for clicks and forms. These tips might help:

- As mentioned before, use `debug_mode` and review its warnings and errors.
- If you see that something (e.g. form or modal) is loading longer than a second, consider adding a delay for that step.
- If a website utilizes lazy loading, you may want to scroll the page before clicking the element on the bottom part of the page.
- If a checkbox in a form can't be marked as true with "form" step, try clickling it instead. Or its label.
- If you want to track a user journey but you don't expect any DL pushes with a click, use "visit" steps - they're much easier to set up than clicks.

### XPath or CSS selectors?

If you're from marketing or web analytics world, it's likely you at least have heard of CSS selectors. But XPath is not so common. But there are cases when XPath is more reliable than CSS. I'm not an expert, but I'll try to leave you with some rules of thumb.

1. If you know CSS selectors, use them as much as possible - they're easier to read.
2. If you can rely on `id` attributes - both options are equally good.
3. If you want to randomize the choice, use CSS selector, but don't make it too broad. 
4. If you want to click a specific element and its CSS is dynamically modified or useless to create selectors (no id, no classes) - use XPath.
5. If in doubt, use XPath - they tend to be more reliable.

## Google Sheets Integration

OMDL can save results directly to Google Sheets. To enable this:

1. Set up a Google Cloud project (or use existing one) and enable Google Sheets API and Google Drive API
2. Set up OAuth consent screen (Desktop app type)
3. Create OAuth 2.0 credentials and download as credentials.json
4. Add required parameters regarding Google Sheets to your TOML file

### Configuration Options

> [!WARNING]
> **Remember that the credentials file is a key to your Google Cloud project and should not be shared with anyone.**
If security is on top of your priorities, I recommend pointing to your credentials via environment variables.

#### Option 1: File-based Configuration

Simple and easy way, good for most local and single-user use cases.

```toml
[config]
output_destination = "google_sheets"

[config.google_sheets]
credentials_path = "credentials.json"
```

#### Option 2: Environment Variables

```toml
[config]
output_destination = "google_sheets"

[config.google_sheets]
credentials_location = "env"
token_location = "env"
```

Remember to set environment variables in your operating system:

- `GOOGLE_SHEETS_CREDENTIALS_PATH` for credentials.json
- `GOOGLE_SHEETS_TOKEN_PATH` for token.pickle

### Saving to Specific Google Drive Folder

To save files in a specific Google Drive folder:

1. Open the folder in Google Drive
2. Copy the folder ID from the URL (the part after `/folders/`)
3. Add to configuration:

```toml
[config.google_sheets]
folder_id = "your_folder_id_here"
```

## Troubleshooting

### Common Issues

#### Browser/ChromeDriver Issues

**Problem**: `WebDriverException: unknown error: Chrome failed to start`
**Solution**: Make sure Chrome is installed and up to date. On macOS, try running `xcode-select --install`.

#### Permission Issues

**Problem**: `PermissionError: [Errno 13] Permission denied: 'credentials.json'`
**Solution**: Check file permissions and ownership of credential files

#### Google Sheets Authentication

**Problem**: `Error: credentials.json file not found`
**Solution**: Ensure the credentials file is in the correct location and properly referenced in the configuration

#### Element Not Found

**Problem**: `Error: Element not found or not clickable`
**Solution**: 
- Check if the selector is correct
- Increase `default_timeout` in configuration or delay time
- Consider using a different selector type (XPath can be more accurate)

## Support the Project

If you find OMDL helpful, consider [buying me a coffee.](https://buycoffee.to/niz) Thank you! 🩷

## Questions & Contributions

- For questions and discussions, please [open an issue](https://github.com/jnizniowski/OMDL/issues).
- Contributions are welcome! Please feel free to submit a Pull Request.
- I would appreciate sharing the project with your network or fellow analysts or developers, too.

## License

MIT License – see LICENSE file for details.