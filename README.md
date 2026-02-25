# auto-kdp: Automated KDP Publishing for AI-Generated Kids Books

## Introduction

This repository contains a customized version of the `auto-kdp` tool, designed to automate the publishing process for AI-generated kids books and coloring books on Amazon Kindle Direct Publishing (KDP). The tool streamlines the workflow by accepting book metadata from a CSV spreadsheet, handling manuscript PDF and cover image uploads, and providing robust error handling, retry logic, and logging capabilities.

## Features

*   **Automated Metadata Upload**: Uploads book title, description, keywords, categories, and pricing directly from a CSV file.
*   **Manuscript and Cover Upload**: Supports automated upload of manuscript PDFs and cover image files.
*   **AI-Generated Content Flag**: Automatically sets the "AI-generated content" flag on KDP, which is crucial for compliance with KDP's updated policies for AI-assisted works.
*   **Enhanced Robustness**: Includes retry logic for page navigation and element interactions, making the automation more resilient to KDP's often finicky interface and network glitches.
*   **Improved Logging**: Detailed verbose logging helps in monitoring the publishing process and troubleshooting any issues.
*   **OpenClaw Friendly**: Structured for easy integration as an OpenClaw skill, allowing for future expansion and workflow automation.

## Installation

To set up the `auto-kdp` tool, follow these steps:

1.  **Clone the repository**:

    ```bash
    git clone https://github.com/groupthinking/auto-kdp.git
    cd auto-kdp
    ```

2.  **Install dependencies**:

    ```bash
    npm install
    ```

3.  **Build the project**:

    ```bash
    npm run build
    ```

## Usage

The `auto-kdp` tool is executed via the command line. It requires a CSV file for book metadata, a configuration file for defaults, and a content directory for manuscript and cover files.

### Command Line Arguments

```
node build/index.js \
  --books <path/to/books.csv> \
  --config <path/to/books.conf> \
  --content-dir <path/to/content/directory> \
  --user-data <path/to/user_data_directory> \
  [--headless <yes|no>] \
  [--dry-run] \
  [--verbose]
```

*   `--books <file>`: **(Required)** Path to the CSV file that contains information for all books. Default: `books.csv`.
*   `--config <file>`: **(Required)** Path to the books configuration file. Default: `books.conf`.
*   `--content-dir <directory>`: **(Required)** Directory containing all manuscript PDFs and cover image files. Default: `.` (current directory).
*   `--user-data <directory>`: **(Required)** User data directory to store browser cookies and profile information. This is essential for maintaining login sessions. Default: `./user_data`.
*   `--headless <yes|no>`: (Optional) Whether to run the browser in headless mode (no visible browser UI). Use `no` for debugging or initial setup. Default: `no`.
*   `--dry-run`: (Optional) If set, no actual actions will be taken on KDP. Useful for testing your setup without making live changes. Default: `false`.
*   `--verbose`: (Optional) Enable verbose logging for detailed output during execution. Default: `false`.

### Example

To run the pipeline with the provided sample files:

```bash
node build/index.js \
  --books ai-kids-books-sample.csv \
  --config ai-kids-books.conf \
  --content-dir . \
  --user-data ./user_data \
  --headless no \
  --verbose
```

### CSV File Structure (`books.csv`)

The CSV file defines the metadata for each book. Each row represents a single book, and columns correspond to specific metadata fields. The column names must match the keys defined in `src/book/keys.ts`.

**Mandatory Fields for AI Kids Books Pipeline:**

*   `action`: Specifies the actions to perform for the book (e.g., `all`, `book-metadata`, `content`, `pricing`, `publish`).
*   `title`: The main title of the book.
*   `subtitle`: The subtitle of the book.
*   `authorFirstName`: Author's first name.
*   `authorLastName`: Author's last name.
*   `description`: HTML-formatted description of the book.
*   `keyword0` - `keyword6`: Search keywords for the book (up to 7).
*   `newCategory1`, `newCategory2`, `newCategory3`: JSON strings representing the KDP categories. These are critical for proper categorization of kids' books. See `ai-kids-books.conf` for examples.
*   `primaryMarketplace`: The primary Amazon marketplace for the book (e.g., `us`, `uk`, `de`).
*   `priceUsd`, `priceGbp`, etc.: Pricing for different marketplaces.
*   `paperTrim`: Trim size of the paperback (e.g., `8.5x8.5`, `6x9`).
*   `paperBleed`: Whether the book has bleed (e.g., `yes`, `no`).
*   `paperCoverFinish`: Cover finish (e.g., `glossy`, `matte`).
*   `paperColor`: Paper color (e.g., `standard-color`, `black-white-white`).
*   `manuscriptLocalFile`: Path to the local PDF file for the book's interior content (relative to `--content-dir`).
*   `coverLocalFile`: Path to the local image file for the book's cover (relative to `--content-dir`).
*   `signature`: A unique identifier for the book, used for internal tracking.

**Example `ai-kids-books-sample.csv`:**

```csv
action,title,subtitle,authorFirstName,authorLastName,description,keyword0,keyword1,keyword2,keyword3,keyword4,keyword5,keyword6,newCategory1,newCategory2,newCategory3,primaryMarketplace,priceUsd,paperTrim,paperBleed,paperCoverFinish,paperColor,manuscriptLocalFile,coverLocalFile,signature
all,The Brave Little Robot,A Journey to the Stars,AI,Author,"Join Sparky the robot on a cosmic adventure! This beautifully illustrated book is perfect for kids aged 3-7.",kids books,robot story,space adventure,picture book,bedtime story,AI generated,coloring book,"{\"level\":0,\"key\":\"Children's Books\",\"nodeId\":\"4\"}","{\"level\":1,\"key\":\"Science Fiction & Fantasy\",\"nodeId\":\"16243\"}","{\"level\":2,\"key\":\"Science Fiction\",\"nodeId\":\"17038\"}",us,9.99,8.5x8.5,yes,glossy,standard-color,manuscript.pdf,cover.jpg,sparky-robot-001
```

### Configuration File (`books.conf`)

The `books.conf` file provides default values for book metadata fields. Any value set in `books.csv` will override the corresponding value in `books.conf`. This is useful for fields that are common across many books.

**Example `ai-kids-books.conf`:**

```ini
# Default configuration for AI Kids Books Pipeline
authorFirstName=AI
authorLastName=Creative
primaryMarketplace=us
paperTrim=8.5x8.5
paperBleed=yes
paperCoverFinish=glossy
paperColor=standard-color
language=English
# The following will be used for all books unless overridden in CSV
newCategory1={"level":0,"key":"Children's Books","nodeId":"4"}
newCategory2={"level":1,"key":"Animals","nodeId":"2975"}
newCategory3={"level":2,"key":"Dragons","nodeId":"17036"}
```

## OpenClaw Compatibility

This project is designed to be OpenClaw-friendly. A `skill.json` file is included, defining the skill's metadata, entry point, and parameters. This allows the `auto-kdp` tool to be easily wrapped and utilized as an OpenClaw skill for broader automation workflows.

## Error Handling and Robustness

Several enhancements have been made to improve the tool's reliability:

*   **Retry Logic**: Page navigation and critical element interactions now include retry mechanisms with exponential backoff, reducing failures due to transient network issues or KDP's dynamic page loading.
*   **Improved Logging**: The `debug` and `error` functions provide more context-rich output, including book-specific prefixes and timestamps, making it easier to diagnose problems.
*   **Input Validation**: Basic checks for file existence (manuscript and cover) are performed before attempting uploads.

## Troubleshooting

*   **`clearTextField` errors during build**: If you encounter TypeScript errors related to `clearTextField` missing arguments, ensure you have the latest version of the code and have run `npm install` and `npm run build`.
*   **Browser issues**: If the browser automation seems stuck or behaves unexpectedly, try running with `--headless no` to observe the browser's actions. Ensure your `user_data` directory has appropriate permissions.
*   **KDP page changes**: Amazon KDP's interface can change. If the tool stops working as expected, it's likely due to changes in KDP's HTML element IDs or structure. Inspect the KDP page manually and update the relevant selectors in the `src/action/` files.
*   **Manuscript/Cover Upload Failures**: Ensure the `manuscriptLocalFile` and `coverLocalFile` paths in your CSV are correct and the files exist in the `--content-dir`.

## Development

### Adding New Metadata Fields

To extend the tool with new KDP metadata fields:

1.  **Update `src/book/keys.ts`**: Add the new key to the `Keys` enum.
2.  **Update `src/book/book.ts`**: Add the new field as a property in the `Book` class and initialize it in the constructor.
3.  **Update `src/action/update-book-metadata.ts`**: Implement the logic to interact with the KDP page element corresponding to the new field (e.g., `updateTextFieldIfChanged`, `selectValue`, `clickSomething`).
4.  **Update CSV/Config**: Add the new field to your `books.csv` or `books.conf` files as needed.

### Running Tests

```bash
npm test
```

## References

[1] [Amazon Kindle Direct Publishing](https://kdp.amazon.com/)
[2] [Puppeteer Documentation](https://pptr.dev/)
[3] [Commander.js Documentation](https://www.npmjs.com/package/commander)
