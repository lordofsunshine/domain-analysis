# Domain Analyzer

Domain Analyzer is a powerful, user-friendly Vue.js application that provides comprehensive information about any given domain. It offers quick insights into DNS, IP details, geolocation, SSL status, and much more, making it an invaluable tool for web developers, network administrators, and cybersecurity professionals.

<img alt="banner" src="https://cdn.glitch.global/146520f4-15c2-44d2-933d-590e1fdfc8aa/43dfc0ea-d6e6-4653-96a8-f2ff7c8f7dc7.image.png?v=1729697694156">

## 👀 v.0.0.2
1. **Expanded Analysis Results**: The tool now provides more comprehensive information about the analyzed domain, including:
   - Mobile Friendliness
   - Load Time
   - Security Headers
   - Domain Creation Date
2. **Technology Detection**: The tool can now detect various technologies used by the website, such as content management systems, JavaScript frameworks, and server technologies.
3. **Website Screenshot**: A screenshot of the analyzed website is now included in the results.
4. **Improved Debug Information**: The debug information now includes more detailed results for each step of the analysis process.
5. **Retry Mechanism**: A retry mechanism has been implemented for all API calls to improve reliability.
6. **Enhanced UI**: The user interface has been improved with a more modern and responsive design.
   
## ✨ Features

- **Domain Validation**: Ensures input is a valid domain name before processing.
- **DNS Resolution**: Resolves domain names to their corresponding IP addresses using Google's DNS service.
- **IP Information**: Retrieves detailed information about the IP address, including country, city, region, and Internet Service Provider (ISP).
- **SSL/HTTPS Check**: Verifies if the domain supports HTTPS connections.
- **Site Availability**: Checks if the website is accessible and measures load time.
- **Technology Detection**: Identifies technologies and frameworks used by the website.
- **Mobile Friendliness**: Assesses whether the site is optimized for mobile devices.
- **Security Headers**: Checks for the presence of important security headers.
- **Domain Creation Date**: Retrieves the registration date of the domain.
- **Website Screenshot**: Captures and displays a screenshot of the website.
- **Debug Information**: Offers a detailed log of each step in the analysis process for troubleshooting.
- **Responsive Design**: Works seamlessly on desktop and mobile devices.

## Security Measures

- [x] **Input Sanitization**: Implements strict input validation to prevent injection attacks.
- [x] **Rate Limiting**: Protects against abuse by limiting the number of requests per second.
- [x] **Error Handling**: Robust error handling to prevent exposing sensitive information.
- [x] **Third-party API Usage**: Relies on reputable third-party APIs for data retrieval.
- [x] **CORS-friendly**: Uses appropriate headers and request methods to avoid cross-origin issues.

## Technology Stack

- **Frontend**: Vue.js 3 with Composition API
- **Styling**: Tailwind CSS for responsive and modern UI
- **HTTP Client**: Axios for making API requests
- **Icons**: Lucide Vue for sleek, customizable icons

## Usage

1. Enter a domain name in the input field.
2. Click the "Analyze" button or press Enter.
3. View the comprehensive analysis results, including:
   - IP address and location information
   - SSL status
   - Site availability and load time
   - Detected technologies
   - Mobile friendliness assessment
   - Security headers
   - Domain creation date
   - Website screenshot
4. (Optional) Toggle the debug information for a detailed view of the analysis process.

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/domain-analyzer.git
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

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ❕ Acknowledgements

- [Vue.js](https://vuejs.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Axios](https://axios-http.com/)
- [Lucide Vue](https://lucide.dev/)
- [Google DNS](https://developers.google.com/speed/public-dns)
- [ipapi.co](https://ipapi.co/)
- [RDAP](https://rdap.org/)
- [Thum.io](https://www.thum.io/)
