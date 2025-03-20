# OMDL (Oh My dataLayer)

**OMDL** is a script for automated collection and validation of dataLayer events from websites. It simulates user journeys and recordS selected – or all – DL events to a spreadsheet with all necessary info.

If you regularly go through a website to verify dataLayer events and their content, this script can save you a lot of time. Once configured, it'll do your job in a few minutes. You'll know what, when, and where was fired to the dataLayer.

The most obvious scenario is **validation of the ecommerce funnel.** With OMDL, you can simulate almost every scenario from visiting a product page to a purchase, including sign-in or filling a shipping info to a form.

It can also be configured to randomly examine pages from a list or click random elements (e.g. links) on a website.

OMDL stands for "Oh My dataLayer", but if you need a more official name for your boss, feel free to call it _Operational Monitoring of dataLayer_.

Here's a sample of what the collected data looks like:

| Step | Event | Timestamp | URL | Event Data | Valid | Error Details |
|------|--------|-----------|----|------------|-------|---------------|
| my_step_name | purchase | 2024-12-08 10:15:25 | https://example-shop.com/thank-you | { "event": "purchase", "currency": "EUR" (...) } | ❌ | Missing required field: ecommerce.transaction_id |

## Features

- 🤖 Browser interaction automation (page visits, clicks, form filling, scrolling)
- 📊 Clean reports in an Excel file or Google Sheets
- ✨ Easy setup in TOML files
- 🛠️ Highly configurable (see [Configuration](#Configuration)) yet not demanding
- 🔄 Support for multiple sequences in one configuration
- 🔍 Error handling with human-friendly messages
- 🧩 Simple validation, following your schema
- 🖥️ Cross-platform – built with Python

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
2. Modify the config file (see [Configuration](#Configuration)) or create a new one
3. Install all dependencies
4. Run the script:
```bash
python3 omdl.py ./your-config-file.toml
```

## Configuration

OMDL aims to cover the variety of scenarios, needs, and technical challenges.

That's why OMDL leaves you – The User – [a long list of settings](CONFIGURATION.md) to let you decide what it should do on your behalf.

The good news is, most of them are optional.

OMDL uses TOML files for configuration. Each configuration file **must** include:

- General configuration (`[config]` section)
- At least one step definition (`[step]` sections)
- At least one sequence definition (`[sequence]` section)

The most minimal setup would look like this:

```toml
[config] # no config required!
[step.my_step_name]
type = "visit"
url = "https://example.com"

[sequence.my_sequence_name]
steps = ["homepage"]
```

**See the [Configuration reference](CONFIGURATION.md)** and  `example_config.toml` for a complete configuration reference with all available options.

Don't worry about messing up – the script will verify it and try to point out what should be fixed.
The majority of settings have specified default values, so even if your config is incorrect, the script can use them to run your sequences anyway.
For example, if OMDL can't find your Google credentials file, it will save results in an XLSX file.

> [!IMPORTANT]
> **I highly recommend adding `debug_mode = true` to the config section**, at least while defining steps. Some errors will not stop the script, but appear as warnings in the log.

## How to design sequence steps

The easiest method of creating steps for OMDL is to manually go through the website and record all selectors for clicks and forms and/or URLs for visits. 

To save XPath or CSS selector, you need to use developer tools in your web browser. 
1. Right click on the element you want to click or fill, choose "Inspect".
2. You'll see your element's HTML definition highlighted. Right click on its code ⇾ Copy ⇾ CSS selector / XPath. 

Usually that's enough. But sometimes crafting valid steps may be challenging, especially for clicks and forms. These tips might help:

- As mentioned before, use `debug_mode` and review its warnings and errors.
- Define longer delays to wait for elements to load/appear.
- You may need to scroll the page before clicking the element on the bottom part of the page (e.g. due to lazy loading).
- If a checkbox can't be toggled with "form" step, try clicking it instead. Or its label.
- If submitting a form doesn't work, try another value of "submit_method".
- Hide overlaying elements (like banners) with `css_elements_to_hide` setting.
- In general, "visit" steps are easier to set up than clicks. If you don't expect any events fired with a click, you skip clicking links.

### XPath or CSS selectors?

If you work with web development or web analytics, it's likely you at least have heard of CSS selectors. But there are cases when XPath is more reliable than CSS. I'm not an expert on that matter, but I'll leave you with some rules of thumb.

1. CSS selectors are easier to read, use, and comprehend. They should be your first choice.
2. But if they don't work, or you can't pinpoint the element with CSS selectors – use XPath.

## Validation

OMDL lets you run a validation for events and its parameters. To add a pattern, you can simply copy the event from dataLayer or from a documentation, and declare:

- which parameters are obligatory,
- what values (literals, regex, or value type) can be assigned.

OMDL saves the detected issues in the output file.

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

**Problem**: `WebDriverException: unknown error: Chrome failed to start`

**Solution**: Make sure Chrome is installed and up to date. On macOS, try running `xcode-select --install`.

**Problem**: `Error: Element not found or not clickable`

**Solution**: 
- Check if the selector is correct
- Increase `default_timeout` in configuration or delay time
- Consider using a different selector type (XPath can be more accurate)
- hide overlay elements with `css_elements_to_hide`
- use scroll step first to handle lazy loading

## Support the Project

If you find OMDL helpful, consider [buying me a coffee.](https://buycoffee.to/niz) Thank you! 🩷

## Questions & Contributions

- For questions and discussions, please [open an issue](https://github.com/jnizniowski/OMDL/issues).
- Contributions are welcome! Please feel free to submit a Pull Request.
- Star the project and let others know about it ⭐

## License

MIT License – see LICENSE file for details.