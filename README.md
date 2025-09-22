# ZAP-Zed-Attack-Proxy-
A comprehensive guide/cheatsheet for learning ZAP(Zed Attack Proxy)
# ZAP (Zed Attack Proxy) Comprehensive Cheat Sheet

## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [GUI Interface](#gui-interface)
- [Command Line Interface](#command-line-interface)
- [API Usage](#api-usage)
- [Scanning Types](#scanning-types)
- [Authentication](#authentication)
- [Scripting](#scripting)
- [Add-ons](#add-ons)
- [Reporting](#reporting)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Troubleshooting](#troubleshooting)

## Overview

ZAP (Zed Attack Proxy) is an open-source web application security scanner maintained by OWASP. It's designed to find security vulnerabilities in web applications during development and testing.

### Key Features
- **Free and Open Source**: No licensing costs
- **Cross-platform**: Windows, macOS, Linux
- **Automated & Manual Testing**: Supports both approaches
- **API Support**: RESTful API for integration
- **Extensible**: Plugin architecture with add-ons
- **OWASP Top 10 Coverage**: Detects common vulnerabilities

## Installation

### Desktop Installation
```bash
# Download from official website
wget https://github.com/zaproxy/zaproxy/releases/latest/download/ZAP_2.14.0_Linux.tar.gz

# Extract and run
tar -xzf ZAP_2.14.0_Linux.tar.gz
cd ZAP_2.14.0
./zap.sh
```

### Docker Installation
```bash
# Pull official Docker image
docker pull zaproxy/zap-stable

# Run ZAP in Docker (headless)
docker run -t zaproxy/zap-stable zap-baseline.py -t https://example.com

# Run ZAP with GUI (requires X11 forwarding)
docker run -u zap -p 8080:8080 -i zaproxy/zap-stable zap-x.sh -daemon -host 0.0.0.0 -port 8080
```

### Package Managers
```bash
# Homebrew (macOS)
brew install --cask zap

# Snap (Linux)
sudo snap install zaproxy --classic

# APT (Debian/Ubuntu)
sudo apt update
sudo apt install zaproxy
```

## Getting Started

### Basic Workflow
1. **Configure Proxy**: Set browser to use ZAP as proxy (default: localhost:8080)
2. **Explore Application**: Browse target application to build site tree
3. **Scan**: Run automated scans (Spider + Active Scan)
4. **Review Results**: Analyze findings and generate reports
5. **Manual Testing**: Use ZAP tools for manual security testing

### Initial Setup
```bash
# Start ZAP in GUI mode
./zap.sh

# Start ZAP in daemon mode (headless)
./zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.disablekey=true
```

## GUI Interface

### Main Components

#### Sites Tree
- Shows discovered URLs and site structure
- Right-click for context menus
- Green = In scope, Red = Out of scope

#### History Tab
- All HTTP requests/responses
- Filter by URL, response codes, etc.
- Double-click to view request/response details

#### Search Tab
- Search across all requests/responses
- Regex support
- Export search results

#### Alerts Tab
- Security vulnerabilities found
- Organized by risk level (High, Medium, Low, Informational)
- Detailed descriptions and remediation advice

### Key Panels

#### Request/Response Tabs
```
Request Tab: Shows HTTP request details
Response Tab: Shows HTTP response details
Break Tab: For intercepting and modifying requests
```

#### Bottom Panel Tabs
- **History**: All HTTP traffic
- **Search**: Search functionality  
- **Alerts**: Security findings
- **Output**: ZAP console output
- **Spider**: Web crawling results

## Command Line Interface

### Basic Commands
```bash
# Quick baseline scan
zap-baseline.py -t https://example.com

# Full scan
zap-full-scan.py -t https://example.com

# API scan
zap-api-scan.py -t https://example.com/api -f openapi

# Start daemon with API key
zap.sh -daemon -port 8080 -config api.key=your-api-key
```

### Common CLI Options
```bash
-daemon                    # Run in background mode
-host <host>              # Listen on specific host (default: localhost)
-port <port>              # Listen on specific port (default: 8080)
-config <key=value>       # Set configuration options
-dir <directory>          # Use specific session directory
-installdir <directory>   # ZAP installation directory
-newsession <path>        # Create new session
-session <path>           # Load existing session
```

### Docker-based Scans
```bash
# Baseline scan with report
docker run -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-baseline.py \
  -t https://example.com -g gen.conf -r testreport.html

# Full scan with custom config
docker run -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py \
  -t https://example.com -g gen.conf -r fullreport.html

# API scan with OpenAPI spec
docker run -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-api-scan.py \
  -t https://api.example.com -f openapi -r apireport.html
```

## API Usage

### Authentication
```bash
# Generate API key
curl "http://localhost:8080/JSON/core/action/generateApiKey/"

# Use API key in requests
curl "http://localhost:8080/JSON/core/view/version/?apikey=your-api-key"
```

### Common API Endpoints
```bash
# Core API
GET  /JSON/core/view/version/                    # Get ZAP version
GET  /JSON/core/view/sites/                      # Get sites in scope
POST /JSON/core/action/newSession/               # Create new session
POST /JSON/core/action/loadSession/              # Load session

# Spider API
POST /JSON/spider/action/scan/                   # Start spider scan
GET  /JSON/spider/view/status/                   # Get spider status
GET  /JSON/spider/view/results/                  # Get spider results

# Active Scan API
POST /JSON/ascan/action/scan/                    # Start active scan
GET  /JSON/ascan/view/status/                    # Get active scan status
GET  /JSON/ascan/view/scanProgress/              # Get scan progress

# Alerts API
GET  /JSON/core/view/alerts/                     # Get all alerts
GET  /JSON/core/view/alertsSummary/              # Get alerts summary
GET  /JSON/core/view/alertCounts/                # Get alert counts
```

### Python API Example
```python
from zapv2 import ZAPv2

# Connect to ZAP
zap = ZAPv2(proxies={'http': 'http://127.0.0.1:8080', 
                     'https': 'http://127.0.0.1:8080'})

# Start spider scan
target = 'https://example.com'
scanid = zap.spider.scan(target)

# Wait for spider to complete
while int(zap.spider.status(scanid)) < 100:
    print(f'Spider progress: {zap.spider.status(scanid)}%')
    time.sleep(1)

# Start active scan
scanid = zap.ascan.scan(target)

# Wait for active scan to complete
while int(zap.ascan.status(scanid)) < 100:
    print(f'Active scan progress: {zap.ascan.status(scanid)}%')
    time.sleep(5)

# Get alerts
alerts = zap.core.alerts()
for alert in alerts:
    print(f"Alert: {alert['alert']} - Risk: {alert['risk']}")
```

## Scanning Types

### 1. Spider Scan (Crawling)
```bash
# GUI: Tools > Spider > Start Scan
# API: POST /JSON/spider/action/scan/

# Parameters:
- url: Target URL
- maxChildren: Maximum child nodes
- recurse: Follow redirects
- contextName: Context to use
- subtreeOnly: Stay within subtree
```

### 2. Active Scan
```bash
# GUI: Tools > Active Scan > Start Scan  
# API: POST /JSON/ascan/action/scan/

# Parameters:
- url: Target URL
- recurse: Scan child nodes
- inScopeOnly: Scan in-scope URLs only
- scanPolicyName: Custom scan policy
- method: HTTP method
- postData: POST data for forms
```

### 3. Passive Scan
- Automatically runs on all traffic
- No additional requests sent
- Analyzes existing requests/responses
- Lower risk of disruption

### 4. AJAX Spider
```bash
# For JavaScript-heavy applications
# GUI: Tools > AJAX Spider > Start Scan
# API: POST /JSON/ajaxSpider/action/scan/

# Better for SPAs and modern web apps
# Uses actual browser engine
# Slower but more thorough
```

## Authentication

### Form-Based Authentication
```bash
# 1. Configure Context
Tools > Options > Context > Add Context

# 2. Set Authentication Method
Context > Authentication > Form-Based Authentication
- Login URL: /login
- Username parameter: username
- Password parameter: password
- Username: testuser
- Password: testpass

# 3. Configure Session Management
Context > Session Management > Cookie-Based Session Management

# 4. Add User
Context > Users > Add User
```

### Script-Based Authentication
```javascript
// Custom authentication script
function authenticate(helper, paramsValues, credentials) {
    var loginUrl = paramsValues.get("loginUrl");
    var username = credentials.getParam("username");
    var password = credentials.getParam("password");
    
    // Perform login request
    var loginRequest = new org.parosproxy.paros.network.HttpMessage();
    loginRequest.getRequestHeader().setMethod("POST");
    loginRequest.getRequestHeader().setURI(new org.apache.commons.httpclient.URI(loginUrl));
    
    var requestBody = "username=" + username + "&password=" + password;
    loginRequest.setRequestBody(requestBody);
    
    helper.sendAndReceive(loginRequest);
    
    return loginRequest;
}
```

### HTTP Authentication
```bash
# Basic/Digest/NTLM authentication
Tools > Options > Context > Authentication > HTTP Authentication
- Hostname: example.com
- Realm: Protected Area
- Username: admin
- Password: secret
```

## Scripting

### Script Types
1. **Authentication**: Custom login procedures
2. **Session Management**: Handle session tokens
3. **Input Vectors**: Define injection points
4. **Active Rules**: Custom vulnerability checks
5. **Passive Rules**: Analyze responses
6. **Proxy Scripts**: Modify requests/responses
7. **Standalone**: Independent scripts
8. **Targeted**: Run against specific URLs

### JavaScript Example
```javascript
// Passive vulnerability detection script
function scanNode(ps, msg, src) {
    var url = msg.getRequestHeader().getURI().toString();
    var response = msg.getResponseBody().toString();
    
    // Check for sensitive information disclosure
    if (response.indexOf("password") != -1 || 
        response.indexOf("secret") != -1) {
        
        ps.raiseAlert(
            2,                           // Risk: Medium
            1,                           // Confidence: Low
            "Sensitive Information",     // Alert name
            "Sensitive data found",      // Description
            url,                        // URL
            "",                         // Parameter
            "",                         // Attack
            "Remove sensitive data",     // Other info
            "Sanitize responses",        // Solution
            "",                         // Evidence
            0,                          // CWE ID
            0                           // WASC ID
        );
    }
}
```

### Python Example
```python
# Active scan rule in Python
def scanNode(as, msg):
    # Get original response
    original_msg = msg.cloneRequest()
    
    # Test for XSS
    xss_payload = "<script>alert('XSS')</script>"
    
    # Modify parameter
    params = original_msg.getUrlParams()
    for param in params:
        test_msg = original_msg.cloneRequest()
        test_msg.getUrlParams().set(param.getName(), xss_payload)
        
        # Send request
        as.sendAndReceive(test_msg)
        
        # Check response
        response = test_msg.getResponseBody().toString()
        if xss_payload in response:
            as.raiseAlert(
                3,                      # High risk
                3,                      # High confidence
                "Cross Site Scripting", # Name
                "XSS vulnerability found",
                test_msg.getRequestHeader().getURI().toString(),
                param.getName(),
                xss_payload,
                "Implement input validation",
                response
            )
```

## Add-ons

### Essential Add-ons
```bash
# Install via GUI: File > Manage Add-ons
# Or via command line during startup

# Popular add-ons:
- Advanced SQLInjection Scanner
- DOM XSS Active Scanner  
- Path Traversal Scanner
- Server Side Include Scanner
- XSLT Injection Scanner
- XXE Scanner
- JWT Support
- GraphQL Support
- WebSockets Support
```

### Custom Add-on Development
```xml
<!-- addon.xml -->
<addon>
    <name>Custom Scanner</name>
    <version>1.0.0</version>
    <description>Custom vulnerability scanner</description>
    <author>Your Name</author>
    <url>https://example.com</url>
    
    <dependencies>
        <addon id="commonlib"/>
    </dependencies>
    
    <files>
        <file>scripts/active/CustomScanner.js</file>
    </files>
</addon>
```

## Reporting

### Report Formats
```bash
# Available formats:
- HTML: Detailed web-based report
- XML: Machine-readable format
- JSON: API-friendly format  
- MD: Markdown format
- PDF: Print-ready format
```

### Generate Reports via GUI
```bash
Tools > Generate Report
- Select format (HTML, XML, JSON, etc.)
- Choose template
- Set output location
- Configure alert filters
```

### Generate Reports via API
```bash
# HTML Report
curl "http://localhost:8080/OTHER/core/other/htmlreport/?apikey=your-key" > report.html

# XML Report  
curl "http://localhost:8080/OTHER/core/other/xmlreport/?apikey=your-key" > report.xml

# JSON Report
curl "http://localhost:8080/JSON/core/view/alerts/?apikey=your-key" > alerts.json
```

### Custom Report Templates
```xml
<!-- Custom XML template -->
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <html>
            <body>
                <h1>Security Report</h1>
                <xsl:for-each select="//alertitem">
                    <div class="alert">
                        <h3><xsl:value-of select="name"/></h3>
                        <p>Risk: <xsl:value-of select="riskdesc"/></p>
                        <p>URL: <xsl:value-of select="url"/></p>
                    </div>
                </xsl:for-each>
            </body>
        </html>
    </xsl:template>
</xsl:stylesheet>
```

## Best Practices

### Pre-Scan Preparation
1. **Define Scope**: Set clear boundaries for testing
2. **Configure Context**: Set up authentication and session handling
3. **Review Policies**: Customize scan policies for your application
4. **Set Exclusions**: Exclude logout, delete, and destructive operations
5. **Timing**: Consider rate limiting and scan timing

### During Scanning
```bash
# Monitor scan progress
- Watch CPU/memory usage
- Monitor target application performance
- Check for false positives
- Pause scans if needed

# Scan optimization
- Use multiple contexts for different app sections
- Implement custom input vectors
- Configure appropriate scan policies
- Use authentication for authenticated sections
```

### Post-Scan Analysis
1. **Triage Alerts**: Review and validate findings
2. **False Positive Analysis**: Mark and document false positives
3. **Risk Assessment**: Evaluate actual risk in your environment
4. **Remediation Planning**: Prioritize fixes based on risk
5. **Retest**: Verify fixes with follow-up scans

### Performance Tuning
```bash
# Memory optimization
export JAVA_OPTS="-Xmx4g -XX:+UseG1GC"

# Scan policy tuning
- Reduce scan intensity for production-like environments
- Adjust timeout values
- Configure concurrent threads
- Set appropriate delays between requests

# Database optimization
- Regular session cleanup
- Optimize database settings
- Consider using PostgreSQL for large scans
```

## Common Use Cases

### CI/CD Integration
```yaml
# Jenkins Pipeline Example
pipeline {
    agent any
    stages {
        stage('Security Scan') {
            steps {
                script {
                    // Start ZAP
                    sh 'docker run -d --name zap -p 8080:8080 zaproxy/zap-stable zap-x.sh -daemon -host 0.0.0.0 -port 8080'
                    
                    // Run baseline scan
                    sh 'docker exec zap zap-baseline.py -t ${TARGET_URL} -r baseline-report.html'
                    
                    // Archive results
                    archiveArtifacts artifacts: 'baseline-report.html'
                    
                    // Cleanup
                    sh 'docker stop zap && docker rm zap'
                }
            }
        }
    }
}
```

### GitHub Actions
```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'https://example.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
```

### API Security Testing
```bash
# OpenAPI/Swagger testing
zap-api-scan.py -t https://api.example.com -f openapi -r api-report.html

# GraphQL testing
zap-api-scan.py -t https://api.example.com/graphql -f graphql -r graphql-report.html

# Custom API testing with Postman collections
zap-api-scan.py -t https://api.example.com -f postman -r postman-report.html
```

### Mobile Application Testing
```bash
# Configure ZAP as proxy for mobile testing
# Android: Settings > WiFi > Advanced > Proxy: Manual
# Host: ZAP_HOST, Port: 8080

# Install ZAP certificate on mobile device
# Download from: http://zap/ROOT.cer

# Test mobile API endpoints
zap-baseline.py -t https://mobile-api.example.com
```

## Troubleshooting

### Common Issues

#### Connection Problems
```bash
# ZAP not starting
- Check Java version (Java 11+ required)
- Verify port availability (8080)
- Check firewall settings
- Review ZAP logs in ~/.ZAP/zap.log

# Browser proxy issues
- Verify proxy settings (localhost:8080)
- Install ZAP certificate for HTTPS
- Check for proxy bypass settings
```

#### Scanning Issues
```bash
# Spider not finding content
- Check authentication configuration
- Verify context and scope settings
- Review robots.txt exclusions
- Enable AJAX spider for JavaScript apps

# Active scan false positives
- Review and tune scan policies
- Update to latest ZAP version
- Check add-on versions
- Customize input vectors
```

#### Performance Issues
```bash
# Slow scanning
- Increase Java heap size: -Xmx4g
- Reduce scan policy intensity
- Implement request delays
- Use fewer concurrent threads

# Memory issues
- Regular session cleanup
- Optimize scan scope
- Use appropriate scan policies
- Monitor system resources
```

### Debug Mode
```bash
# Enable debug logging
zap.sh -daemon -config log.level=DEBUG

# View detailed logs
tail -f ~/.ZAP/zap.log

# API debug
curl -v "http://localhost:8080/JSON/core/view/version/"
```

### Support Resources
- **Official Documentation**: https://www.zaproxy.org/docs/
- **User Group**: https://groups.google.com/group/zaproxy-users
- **GitHub Issues**: https://github.com/zaproxy/zaproxy/issues
- **YouTube Channel**: ZAP official tutorials
- **OWASP ZAP Wiki**: Comprehensive documentation

---

## Quick Reference Commands

```bash
# Start ZAP GUI
./zap.sh

# Start ZAP daemon
./zap.sh -daemon -host 0.0.0.0 -port 8080

# Quick baseline scan
zap-baseline.py -t https://example.com

# Full automated scan
zap-full-scan.py -t https://example.com

# API scan with OpenAPI
zap-api-scan.py -t https://api.example.com -f openapi

# Generate HTML report
curl "http://localhost:8080/OTHER/core/other/htmlreport/" > report.html

# Get all alerts as JSON
curl "http://localhost:8080/JSON/core/view/alerts/" | jq .
```

---

**Remember**: Always ensure you have proper authorization before scanning any web application. ZAP is a powerful tool that should be used responsibly and only on applications you own or have explicit permission to test.
