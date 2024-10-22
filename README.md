
# Domain Analyzer

Domain Analyzer is a powerful, user-friendly Vue.js application that provides comprehensive information about any given domain. It offers quick insights into DNS, IP details, geolocation, and SSL status, making it an invaluable tool for web developers, network administrators, and cybersecurity professionals.

<img alt="banner" src="https://cdn.glitch.global/146520f4-15c2-44d2-933d-590e1fdfc8aa/186348fc-a934-406d-b3e5-ae682f39faf1.image.png?v=1729614523623">

<hr>

## ✨ Features

- **Domain Validation**: Ensures input is a valid domain name before processing.
- **DNS Resolution**: Resolves domain names to their corresponding IP addresses using Google's DNS service.
- **IP Information**: Retrieves detailed information about the IP address, including:
  - Country
  - City
  - Region
  - Internet Service Provider (ISP)
- **SSL/HTTPS Check**: Verifies if the domain supports HTTPS connections.
- **Real-time Analysis**: Performs all checks in real-time, providing up-to-date information.
- **Debug Information**: Offers a detailed log of each step in the analysis process for troubleshooting.
- **Responsive Design**: Works seamlessly on desktop and mobile devices.

## Security Measures

- [x] **Input Sanitization**: Implements strict input validation to prevent injection attacks.
- [x] **Rate Limiting**: Protects against abuse by limiting the number of requests per second.
- [x] **CORS-friendly**: Uses `no-cors` mode for SSL checks to avoid cross-origin issues.
- [x] **Error Handling**: Robust error handling to prevent exposing sensitive information.
- [x] **Third-party API Usage**: Relies on reputable third-party APIs (Google DNS, ipapi.co) for data retrieval.

## Technology Stack

- **Frontend**: Vue.js 3 with Composition API
- **Styling**: Tailwind CSS for responsive and modern UI
- **HTTP Client**: Axios for making API requests
- **Icons**: Lucide Vue for sleek, customizable icons

## Usage

1. Enter a domain name in the input field.
2. Click the "Analyze" button or press Enter.
3. View the analysis results, including IP address, location, and SSL status.
4. (Optional) Toggle the debug information for a detailed view of the analysis process.

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/lordofsunshine/domain-analysis.git
   ```
2. Navigate to the project directory:
   ```
   cd domain-analyzer
   ```
3. Install dependencies:
   ```
   npm install
   ```
4. Run the development server:
   ```
   npm run dev
   ```

## License

This project is licensed under the MIT License - see the LICENSE file for details.
