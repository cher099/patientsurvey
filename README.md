# Patient Satisfaction Survey

A bilingual patient satisfaction survey application built with HTML5, CSS3, and vanilla JavaScript, integrated with Tally.so for survey collection.

Designed for a hospital environment, the application provides a streamlined patient feedback experience with nurse attribution, submission confirmation, and browser-based submission restrictions.

The project demonstrates AI-assisted development guided by programming fundamentals, technical decision-making, and security-conscious implementation practices.

## Overview

Patient Satisfaction Survey is a lightweight, browser-based application that serves as an interface between patients and an externally hosted survey form.

The application integrates with Tally.so to collect survey responses while independently managing the respondent's interaction with the survey.

It was developed with practical considerations for hospital workflows, including bilingual communication, nurse attribution, submission-state management, and persistent browser storage.

The application is deployed through GitHub Pages and is used in an actual hospital setting.

## Features

- **Bilingual Interface:** Provides English and Arabic messages, including right-to-left Arabic text rendering.
- **External Survey Integration:** Embeds a Tally.so form through an iframe.
- **Nurse Attribution:** Supports dynamic nurse identification through URL query parameters.
- **Submission Confirmation:** Displays a bilingual thank-you message following the configured survey submission redirect.
- **Three-Day Submission Lockout:** Prevents subsequent survey access through the same browser while a submission lock exists.
- **Multi-Storage Persistence:** Uses localStorage, cookies, and IndexedDB to maintain submission timestamps.
- **Storage Recovery:** Restores storage redundancy when a submission timestamp is available in at least one supported storage mechanism.
- **Remaining-Time Calculation:** Displays the remaining lockout duration in English and Arabic.
- **Dynamic Interface States:** Automatically displays the survey form, submission confirmation, or lockout notification based on application state.
- **Responsive Layout:** Uses a fluid layout suitable for desktop and mobile browsers.
- **Static Hosting:** Deployed through GitHub Pages without requiring a dedicated application server.

## Technology Stack

| Technology | Purpose |
|---|---|
| HTML5 | Application structure and semantic markup |
| CSS3 | Styling, layout, and responsive presentation |
| Vanilla JavaScript | Application logic and DOM manipulation |
| localStorage | Primary browser-based timestamp persistence |
| Cookies | Secondary timestamp persistence |
| IndexedDB | Additional browser-based persistence |
| Tally.so | External survey form and response collection |
| Google Sheets | Survey response destination through Tally integration |
| GitHub Pages | Static website hosting |

## Architecture and Implementation

The application follows a lightweight, function-based JavaScript architecture using native browser APIs.

Instead of introducing an additional framework or build system, the implementation uses an asynchronous immediately invoked function expression (async IIFE), helper functions, and browser-native storage mechanisms.

### 1. Application Initialization

The application initializes through an asynchronous IIFE:

```javascript
(async function() {
    // Application initialization
})();
```

This provides an isolated execution scope for application logic and allows asynchronous storage operations to complete before determining the appropriate interface state.

The initialization process:

1. Reads URL query parameters.
2. Restores available browser storage data.
3. Checks whether the visitor arrived through the configured Tally submission redirect.
4. Determines whether an existing submission lock is present.
5. Displays the appropriate application state.

### 2. Multi-Storage Persistence

The application uses three browser storage mechanisms to maintain submission timestamps.

| Storage | Implementation |
|---|---|
| localStorage | Stores the primary submission timestamp |
| Cookies | Maintains a secondary timestamp copy |
| IndexedDB | Maintains an additional timestamp record |

The JavaScript implementation includes dedicated helper functions for:

- Saving timestamps.
- Retrieving existing timestamps.
- Restoring storage redundancy.
- Clearing stored submission data.

When retrieving an existing lock, the application follows this priority:

1. localStorage
2. Cookies
3. IndexedDB

If a timestamp is recovered from any available storage mechanism, the application attempts to restore the other storage mechanisms.

This provides additional persistence against individual storage failures or missing records.

### 3. Submission Lockout Mechanism

The submission lockout duration is configured through:

```javascript
const THREE_DAYS = 3 * 24 * 60 * 60 * 1000;
```

When a valid configured submission redirect is detected, the application records the current timestamp.

Subsequent visits check the stored timestamp and calculate the remaining lockout period.

The application displays a blocked state instead of the survey form while an existing submission lock remains.

The remaining duration is calculated dynamically and presented in both supported languages.

### 4. Bilingual Interface and Localization

The application supports English and Arabic interface messages.

Arabic text uses right-to-left text direction through CSS:

```css
.arabic {
    display: block;
    direction: rtl;
}
```

Both languages are displayed together for:

- Submission confirmation.
- Previously submitted notifications.
- Remaining lockout duration.

The remaining-time helper also generates separate English and Arabic representations of the calculated duration.

### 5. Dynamic Nurse Attribution

The application supports nurse attribution through URL query parameters.

Example:

```text
https://cher099.github.io/patientsurvey/?nurse=Jane
```

The nurse parameter is retrieved using the native URLSearchParams API.

The value is URL-encoded before being appended to the external Tally form URL.

This allows the same survey application to be accessed through different nurse-specific links without requiring separate application instances.

### 6. Tally.so Integration

The application integrates with Tally.so through an embedded iframe.

The survey URL is constructed dynamically and assigned to the iframe's `src` attribute.

The application supports a configured submission redirect using the following query parameters:

```text
?submitted=1&from=tally
```

Upon detecting the expected redirect, the application:

1. Records the submission timestamp.
2. Persists the lock across available browser storage mechanisms.
3. Removes the submission query parameters from the browser address bar.
4. Displays the bilingual thank-you message.

The Tally form must be configured to redirect to the appropriate application URL after submission.

### 7. Survey Response Data Flow

The application uses Tally.so as its external survey service, with Google Sheets connected through Tally's native integration.

The data flow is:

```text
Patient
   |
   v
GitHub Pages
   |
   v
Patient Survey Application
   |
   v
Tally.so
   |
   v
Google Sheets
```

The application itself does not directly communicate with Google Sheets or contain Google Sheets API credentials.

Tally.so handles survey submission and forwards responses to the connected spreadsheet according to its integration configuration.

### 8. Application State Management

The application manages three primary interface states.

| State | Behavior |
|---|---|
| Survey Form | Displays the embedded Tally survey |
| Thank You | Displays the bilingual submission confirmation |
| Blocked | Displays the previous-submission notification and remaining lockout duration |

State transitions are handled through dedicated JavaScript functions that control the visibility of the corresponding HTML elements.

### 9. Asynchronous Browser Operations

The application uses Promises and async/await to coordinate IndexedDB operations.

IndexedDB requests are wrapped in Promise-based helper functions to simplify asynchronous storage operations.

Error handling is also incorporated to prevent individual storage failures from unnecessarily interrupting the rest of the application.

## Project Structure

```text
patientsurvey/
│
└── index.html
```

The application is intentionally self-contained.

Its HTML markup, CSS styling, and JavaScript implementation are maintained within a single HTML document, eliminating the need for external application source files or a compilation process.

## Getting Started

### Requirements

- A modern web browser.
- An internet connection.
- An active Tally.so form configured for the application.

No Node.js installation, package manager, JavaScript framework, or build process is required.

### Installation

Clone the repository:

```bash
git clone https://github.com/cher099/patientsurvey.git
```

Navigate to the project directory:

```bash
cd patientsurvey
```

### Running Locally

Start a local HTTP server:

```bash
python -m http.server 8000
```

Open the application:

```text
http://localhost:8000
```

The application can also be opened directly in a modern browser, although using a local HTTP server provides a more consistent development environment.

An internet connection is required to load the externally hosted Tally survey.

## URL Parameters

### Nurse Attribution

Pass the nurse's name through the `nurse` query parameter:

```text
http://localhost:8000/?nurse=Jane
```

The application forwards the value to the embedded Tally form.

### Submission Redirect

The application recognizes the following configured submission redirect:

```text
http://localhost:8000/?submitted=1&from=tally
```

This triggers the submission confirmation state and records the browser-based submission timestamp.

The Tally form must be configured with the corresponding redirect URL.

## Deployment

The application is hosted using GitHub Pages.

**Repository:**

https://github.com/cher099/patientsurvey

**Live Application:**

https://cher099.github.io/patientsurvey/

The deployment uses GitHub Pages static hosting and does not require a dedicated backend server.

Application updates can be deployed by pushing changes to the configured GitHub Pages publishing source.

## AI-Assisted Development & Engineering Approach

This project was developed through an AI-assisted engineering workflow, combining AI-driven implementation with human technical direction, architectural decisions, and code review.

AI is used as a development accelerator rather than a replacement for programming fundamentals or engineering judgment.

The development process emphasizes understanding the implementation, evaluating generated code, identifying potential issues, and validating the resulting application.

### Development Workflow

- **Requirements Analysis:** Define functional requirements, application behavior, and implementation constraints.
- **Technical Planning:** Determine application structure, browser APIs, storage mechanisms, and state transitions.
- **AI-Assisted Implementation:** Leverage AI tools for code generation, implementation guidance, debugging, and iterative refinement.
- **Code Review:** Examine generated code for correctness, maintainability, unnecessary complexity, and potential security weaknesses.
- **Testing and Validation:** Evaluate application behavior, asynchronous operations, storage handling, and edge cases.
- **Technical Ownership:** Retain responsibility for implementation decisions, reviewing changes, and validating the final application.

### Engineering Fundamentals Applied

The project demonstrates practical application of:

- Asynchronous JavaScript programming using Promises and async/await.
- Function-based application organization and encapsulation through an IIFE.
- Browser-native persistence using localStorage, cookies, and IndexedDB.
- URL parameter parsing and input encoding.
- DOM manipulation and application state management.
- Graceful handling of browser storage failures.
- Awareness of client-side security and privacy limitations.

AI assistance supports implementation, but technical direction, code evaluation, and final validation remain essential parts of the development workflow.

## Data Handling, Privacy & Security Considerations

The application uses a third-party survey service for response collection rather than maintaining its own backend or database.

### Data Flow

1. GitHub Pages hosts the static application.
2. The application embeds the Tally.so survey.
3. Patients submit their responses through Tally.so.
4. Tally.so forwards submissions to the connected Google Sheets spreadsheet through its native integration.

Google Sheets is not directly accessed or configured by the application code.

### Privacy Considerations

- Survey responses are not published through the public website.
- Access to collected responses is managed through the external service accounts and their respective permissions.
- The application itself stores submission timestamps in browser storage.
- Nurse attribution is passed through URL parameters.
- Patient-identifying information should not be included in public URLs or exposed through the application's source code.

### Compliance Considerations

This project does not independently establish HIPAA compliance.

The use of external services for collecting and storing survey responses requires the hospital to verify applicable privacy, security, contractual, and regulatory requirements.

In particular, the hospital should confirm whether the selected Tally.so and Google Workspace services, account configurations, and applicable Business Associate Agreements are appropriate for the information being collected.

The application should not be used to collect protected health information unless the hospital has approved the complete data-handling arrangement.

The application is a survey interface and does not replace a secure, centralized hospital information system or independently enforce patient data security.

## Project Scope and Limitations

This project is a lightweight, browser-based survey interface.

It does not include:

- A dedicated backend application.
- A centralized application database.
- Patient authentication or identity verification.
- Server-side duplicate-submission prevention.
- Independent survey response processing.
- Direct Google Sheets API integration.
- Hospital information system integration.

Survey response collection is handled externally through Tally.so and its connected Google Sheets integration.

### Browser Storage Limitations

The three browser storage mechanisms provide redundancy but do not guarantee submission integrity.

Consequently:

- Browser storage can be cleared or modified by users.
- Submission restrictions are limited to the current browser's available storage.
- Different browsers and devices maintain separate storage environments.
- Client-side timestamps cannot independently establish patient identity.
- The three storage mechanisms do not constitute independent security controls.

For stronger submission integrity, centralized persistence and server-side validation would be necessary.

## Third-Party Services

The application integrates with the following external services:

| Service | Purpose |
|---|---|
| Tally.so | External survey hosting and response collection |
| Google Sheets | Survey response storage through Tally's native integration |
| GitHub Pages | Static website hosting and deployment |

These services remain subject to their respective terms, privacy policies, and service availability.

## License

No open-source license is currently provided.

The repository is publicly accessible as a technical demonstration and application source repository. Third-party services and materials remain subject to their respective ownership and licensing terms.

---

**Developer:** Cherwin Jay

**Repository:** [github.com/cher099/patientsurvey](https://github.com/cher099/patientsurvey)

**Live Application:** [cher099.github.io/patientsurvey](https://cher099.github.io/patientsurvey/)

**Development Approach:** AI-assisted, human-directed engineering

**Application Status:** Deployed and used in a hospital setting
