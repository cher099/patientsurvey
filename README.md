# Patient Satisfaction Survey

A bilingual patient satisfaction survey application built with HTML5, CSS3, and vanilla JavaScript, integrated with Tally for external survey collection.

The application provides a lightweight patient-facing interface with nurse attribution, submission confirmation, and a browser-based three-day submission lockout. It is deployed through GitHub Pages and is used in connection with Al Rass General Hospital in Al Rass, Al-Qassim, Saudi Arabia.

This repository documents the application implementation. It does not represent a formal hospital security assessment, privacy approval, or certification of regulatory compliance.

## Overview

Patient Satisfaction Survey is a static, browser-based interface between patients and an externally hosted survey form.

The application embeds a Tally form, forwards an optional nurse name from the page URL, recognizes a configured submission redirect, and manages the respondent-facing survey state.

The survey collects the following information through Tally:

- Patient File Number
- Phone Number
- Yes/No answers about the services received
- Additional comments

Tally is connected to Google Sheets through its native integration. The application source does not directly call the Google Sheets API or contain Google Sheets credentials.

## Features

- **Bilingual Interface:** English and Arabic messages, with right-to-left Arabic text rendering.
- **External Survey Integration:** Embeds a Tally form in an iframe.
- **Nurse Attribution:** Accepts a nurse name through the `nurse` URL query parameter and forwards it to Tally.
- **Submission Confirmation:** Shows a bilingual thank-you message after recognizing the configured Tally submission redirect.
- **Three-Day Browser Lockout:** Blocks the survey in the same browser profile while a submission timestamp remains within the three-day period.
- **Multi-Storage Timestamp Persistence:** Stores the lockout timestamp in localStorage, a cookie, and IndexedDB.
- **Storage Recovery:** Attempts to restore the other storage copies when a timestamp is found in one supported storage mechanism.
- **Remaining-Time Display:** Shows the approximate remaining lockout duration in English and Arabic.
- **Responsive Layout:** Uses a fluid layout for desktop and mobile browsers.
- **Static Hosting:** Runs on GitHub Pages without a custom application server.

## Technology Stack

| Technology | Purpose |
|---|---|
| HTML5 | Page structure and semantic markup |
| CSS3 | Styling, layout, and Arabic text direction |
| Vanilla JavaScript | Application logic and DOM manipulation |
| localStorage | Browser-side lockout timestamp |
| Cookies | Additional browser-side timestamp persistence |
| IndexedDB | Additional browser-side timestamp persistence |
| Tally | External survey form and response collection |
| Google Sheets | Response destination through Tally's integration |
| GitHub Pages | Static website hosting |

## Architecture and Implementation

The application is implemented as a single HTML document containing its markup, CSS, and JavaScript. Its JavaScript uses an asynchronous immediately invoked function expression (async IIFE), helper functions, Promises, async/await, and browser-native APIs. It does not use a JavaScript framework, class-based object-oriented architecture, or build system.

### 1. Initialization and interface state

The asynchronous initialization routine:

1. Reads the current URL query parameters.
2. Attempts to restore the browser-side lockout timestamp across available storage mechanisms.
3. Checks whether the page was reached through the configured Tally submission redirect.
4. Checks for an existing lockout timestamp.
5. Displays the survey, thank-you, or blocked state.

The three interface states are:

| State | Behavior |
|---|---|
| Survey | Displays the embedded Tally form |
| Thank You | Displays the bilingual submission confirmation |
| Blocked | Displays the previous-submission message and remaining lockout duration |

### 2. Browser storage

The application writes the submission timestamp to three browser storage mechanisms:

| Storage | Use |
|---|---|
| localStorage | Primary timestamp lookup |
| Cookie | Additional timestamp copy |
| IndexedDB | Additional timestamp copy |

When reading a timestamp, the application checks localStorage first, then the cookie, then IndexedDB. When it finds a timestamp, it attempts to write that value to the other storage mechanisms.

This is client-side redundancy, not a server-side record or a security boundary. Browser storage can be cleared, changed, or unavailable, and storage is separate between browser profiles and devices.

### 3. Three-day lockout

The lockout duration is configured as:

```javascript
const THREE_DAYS = 3 * 24 * 60 * 60 * 1000;
```

When the application recognizes the configured submission redirect, it records the current timestamp. On a later visit, it calculates the elapsed time and displays the remaining duration while the three-day period has not elapsed.

The restriction is enforced only by the browser-side application logic. It does not reliably prevent repeat submissions from another browser, device, or cleared browser profile, and it should not be treated as duplicate-submission control for clinical or operational records.

### 4. English and Arabic interface

The thank-you and blocked messages are shown in English and Arabic. Arabic text is given right-to-left direction using the `.arabic` CSS class.

The remaining-time helper produces separate English and Arabic strings for the displayed duration.

### 5. Nurse attribution

The page accepts a nurse name through the `nurse` query parameter. For example:

```text
https://cher099.github.io/patientsurvey/?nurse=Jane
```

The application reads the value using `URLSearchParams`, URL-encodes it, and appends it to the Tally form URL.

Because query parameters may be retained in browser history, server logs, analytics, or shared links, they should not contain patient identifiers or other confidential information. The nurse parameter should contain only information the hospital has approved for this purpose.

### 6. Tally form and submission redirect

The application embeds the configured Tally form in an iframe. It recognizes a submission redirect containing:

```text
?submitted=1&from=tally
```

When the expected redirect is detected, the application records the timestamp, removes the query string from the visible address bar, and displays the bilingual thank-you message.

The redirect must be configured in the Tally form to return to the deployed application URL with the expected parameters. The application also contains a generic `postMessage` listener for two example message strings; that listener is not the primary submission confirmation mechanism.

## Survey Data Flow and Handling

### Data flow

The response path currently documented for this project is:

```text
Patient's browser
      |
      v
GitHub Pages application
      |
      | embeds the form
      v
Tally form and submission service
      |
      | Tally's native Google Sheets integration
      v
Connected Google Sheets spreadsheet
      |
      v
Spreadsheet access by the designated account holders
```

The application source contains the Tally form URL and embeds the form. It does not contain a direct Google Sheets integration, Google API credentials, or a backend endpoint for patient responses.

### Information collected by the survey

The survey collects:

- Patient File Number
- Phone Number
- Yes/No answers about the services received
- Additional comments

Patient File Numbers and phone numbers can identify an individual. The answers and comments relate to healthcare services. This information should therefore be treated as confidential personal information and potentially sensitive health-related information.

### What the application stores in the browser

The application stores a submission timestamp in localStorage, a cookie, and IndexedDB to implement its three-day lockout. The source code does not store the Patient File Number, phone number, or survey answers in those browser storage mechanisms.

Survey response data is submitted to Tally and forwarded to the connected Google Sheets spreadsheet through Tally's integration.

## Privacy, Security, and Saudi Regulatory Considerations

Al Rass General Hospital is in Saudi Arabia. Saudi Arabia's Personal Data Protection Law (PDPL), its Implementing Regulations, and the Regulation on Personal Data Transfer Outside the Kingdom are relevant legal frameworks to be assessed for this survey's collection and handling of identifiable patient information.

This README is a technical description, not legal advice or a compliance assessment. The repository and application have not independently established PDPL compliance, hospital authorization, a security assessment, or approval of the third-party services for patient data.

Before continuing to collect identifiable patient information, the hospital's authorized privacy, legal, information security, and IT representatives should review and approve the complete arrangement. In particular, they should confirm:

- The hospital's role and responsibilities as the relevant personal data controller, and the authorised roles of individuals who can access responses.
- The lawful basis for processing, the purpose of the survey, the patient-facing privacy notice, and any consent or other notices required by the hospital.
- Whether the Patient File Number, phone number, questions, and free-text comments are necessary and proportionate for the stated purpose.
- Tally's role as a service provider, the applicable contractual and data-processing terms, and the locations where data is hosted, stored, accessed, and otherwise processed.
- The Google account and Google Sheets configuration, sharing permissions, access controls, account ownership, and applicable service terms.
- Whether any transfer or remote access of personal data outside Saudi Arabia occurs, and whether the PDPL's cross-border transfer requirements and safeguards are satisfied.
- Retention periods, deletion procedures, incident handling, access logging, and a process for responding to data-subject requests.
- Whether additional Saudi healthcare-sector rules, Ministry of Health requirements, or hospital-specific policies apply to this workflow.

The fact that Tally forwards responses to Google Sheets does not, by itself, demonstrate that data residency, international transfer, contractual, or security requirements have been met. Those details must be confirmed against the services and accounts actually used.

### Access and operational safeguards

Access to the response spreadsheet should be limited to people expressly authorised by the hospital and who need the information for their assigned duties. Personal Gmail ownership or sharing between individuals should not be assumed to constitute hospital-authorised access.

Do not place patient names, Patient File Numbers, phone numbers, response data, credentials, or private spreadsheet links in this public repository, its source code, public issue threads, or URL parameters.

If the hospital has not approved this collection and data flow, pause the collection of identifiable patient responses and refer the existing records to the hospital's authorised privacy or information security contact for direction. Do not independently delete or redistribute existing patient responses.

## Project Structure

```text
patientsurvey/
└── index.html
```

The project is intentionally self-contained. HTML, CSS, and JavaScript are maintained in `index.html`; no build step or package installation is required.

## Getting Started

### Requirements

- A modern web browser.
- An internet connection.
- The configured Tally form to be available.

### Clone the repository

```bash
git clone https://github.com/cher099/patientsurvey.git
cd patientsurvey
```

### Run locally

Start a local HTTP server:

```bash
python -m http.server 8000
```

Open:

```text
http://localhost:8000
```

An internet connection is required to load the embedded Tally form. Use test data only when developing or validating the application; do not use real patient information in an unapproved environment.

## URL Parameters

### Nurse attribution

```text
http://localhost:8000/?nurse=Jane
```

The application forwards the nurse value to the embedded Tally form. Do not use this parameter for patient-identifying or confidential information.

### Submission redirect

The application expects the configured Tally redirect to include:

```text
http://localhost:8000/?submitted=1&from=tally
```

The deployed Tally configuration should use the corresponding production URL.

## Deployment

The application is hosted using GitHub Pages.

- **Repository:** https://github.com/cher099/patientsurvey
- **Live application:** https://cher099.github.io/patientsurvey/

The site is statically hosted and does not require a custom application server. The Tally form remains an external dependency.

## AI-Assisted Development and Engineering Approach

This project was developed using an AI-assisted, human-directed workflow. AI tools supported implementation, debugging, and iterative refinement, while technical direction, review, and validation remained part of the development process.

The project demonstrates practical use of:

- Native JavaScript and browser APIs.
- Asynchronous programming with Promises and async/await.
- Function-based code organisation and an IIFE execution scope.
- Browser storage using localStorage, cookies, and IndexedDB.
- URL parameter parsing and URL encoding.
- DOM manipulation and interface state management.
- Error handling around browser storage operations.
- Awareness of client-side privacy and security limitations.

AI assistance is disclosed as part of the development process; it should not be interpreted as a claim that every implementation detail has been independently audited or that the system is security-certified.

## Scope and Limitations

This is a static survey interface. It does not include:

- A dedicated backend application.
- A central application database.
- Patient authentication or identity verification.
- Server-side duplicate-submission prevention.
- Direct Google Sheets API access.
- Integration with a hospital information system.
- Centralised audit logging or server-side access enforcement.

The survey itself is hosted and processed by Tally, with responses forwarded to Google Sheets through Tally's integration.

### Browser lockout limitations

The three browser storage mechanisms improve the chance that the local timestamp remains available, but they do not provide authoritative enforcement:

- A user can clear or modify browser storage.
- Different browsers and devices have separate storage.
- Private browsing and browser settings may affect persistence.
- A successful client-side redirect is not a server-verified proof of submission.
- The lockout does not identify a patient or guarantee one response per patient.

A stronger duplicate-submission control would require an appropriately authorised, centrally managed mechanism and a hospital-approved architecture.

## Third-Party Services

| Service | Purpose |
|---|---|
| Tally | Form hosting and survey response collection |
| Google Sheets | Response destination through Tally's native integration |
| GitHub Pages | Static application hosting |

Each service is subject to its own terms, privacy documentation, configuration, and availability. The hospital should assess the actual accounts and service arrangements used for this deployment.

## License

No open-source license is currently provided. The repository is publicly accessible, but public visibility does not itself grant permission to reuse the code, hospital-related materials, form content, branding, or third-party assets. Obtain the appropriate rights and approvals before reuse.

---

**Developer:** Cherwin Jay  
**Repository:** [github.com/cher099/patientsurvey](https://github.com/cher099/patientsurvey)  
**Live application:** [cher099.github.io/patientsurvey](https://cher099.github.io/patientsurvey/)  
**Development approach:** AI-assisted, human-directed engineering  
**Deployment context:** Al Rass General Hospital, Al Rass, Al-Qassim, Saudi Arabia  
