API Quality Scorecard Tool
This project is a command-line tool designed to analyze and score the quality of an OpenAPI Specification (OAS) file. The tool evaluates the API design against a comprehensive framework, providing a detailed report with an overall score, category breakdowns, and actionable recommendations for improvement.
The goal is to help developers and technical writers identify areas where their API documentation and design can be improved to enhance clarity, usability, and consistency for consumers.
Quality Scoring Framework
The tool scores the API across five key categories, with a total possible score of 100 points.
Category
Max Score
Description
Documentation Quality
25
Assesses the presence and quality of descriptions, summaries, examples, and tags for operations and parameters.
Schema Completeness
25
Evaluates how well request and response schemas are defined, including parameter types and required fields.
Error Handling
20
Checks for proper documentation of 4xx and 5xx responses, including error schemas and examples.
Agent Usability
20
Measures clarity and consistency through operation IDs, discoverability via tags, and overall complexity.
Authentication Clarity
10
Scores the documentation of security schemes, OAuth scopes, and authentication examples.

How to Use
1. Project Setup
Ensure your project structure matches the following:
.
├── scorecard/
│   ├── analyzer.py
│   ├── parser.py
│   ├── reporter.py
└── main.py

2. Dependencies
This tool requires several Python libraries. You can install them using pip:
pip install PyYAML jsonschema openapi-spec-validator

3. Running the Tool
You can run the analysis on any OpenAPI file (YAML or JSON) by providing the file path as an argument.
First, create a main.py file to orchestrate the analysis.
# main.py

import sys
from scorecard.parser import OpenAPIParser
from scorecard.analyzer import QualityAnalyzer
from scorecard.reporter import Reporter

def main():
    """
    Main entry point for the API quality scorecard tool.
    """
    if len(sys.argv) < 2:
        print("Usage: python main.py <path_to_openapi_spec>")
        sys.exit(1)

    spec_path = sys.argv[1]
    
    # 1. Parse and validate the specification
    print(f"Analyzing specification: {spec_path}...")
    parser = OpenAPIParser(spec_path)
    if not parser.validate_spec():
        print("Invalid specification. Please fix the errors before running the analysis.")
        sys.exit(1)
        
    spec_title = parser.get_summary_and_description().get('title', 'Untitled API')

    # 2. Run the analysis
    analyzer = QualityAnalyzer(parser)
    report_data = analyzer.analyze()
    
    # 3. Generate reports
    reporter = Reporter(report_data, spec_title)

    # Generate and print a Markdown report to the console
    markdown_report = reporter.generate_markdown_summary()
    print("\n--- Markdown Report ---")
    print(markdown_report)

    # Generate and save an HTML report
    html_report = reporter.generate_html_report()
    html_filename = f"{spec_title.replace(' ', '-').lower()}_report.html"
    with open(html_filename, "w", encoding="utf-8") as f:
        f.write(html_report)
    print(f"\n--- HTML Report ---")
    print(f"Saved a detailed HTML report to ./{html_filename}")
    
    # Generate and save a JSON report
    json_report = reporter.generate_json_report()
    json_filename = f"{spec_title.replace(' ', '-').lower()}_report.json"
    with open(json_filename, "w", encoding="utf-8") as f:
        f.write(json_report)
    print(f"\n--- JSON Report ---")
    print(f"Saved a machine-readable JSON report to ./{json_filename}")


if __name__ == "__main__":
    main()

Then, from the root directory, run the script with your OpenAPI file:
python main.py path/to/your/spec.yaml

Example Output
The tool will generate a detailed summary similar to this Markdown report:
# API Quality Scorecard: Untitled API

**Overall Score: 85/100**

---

### **Category Scores**
| Category | Score | Max Score |
| :--- | :--- | :--- |
| Documentation | 22 | 25 |
| Schemas | 25 | 25 |
| Error Handling | 15 | 20 |
| Usability | 18 | 20 |
| Authentication | 5 | 10 |

---

### **Analysis Summary**
* **Total Operations:** 10

---

### **Issues Found**
* The 'GET /users' operation is missing a detailed description.
* The 'POST /items' operation is missing documentation for common client (4xx) and server (5xx) errors.
* The 'GET /items' operation has a high number of parameters, which may indicate high complexity.

---

### **Actionable Recommendations**
* Add a detailed 'description' for the 'GET /users' operation.
* Define schemas for common error responses (4xx, 5xx) in 'POST /items'.
* Add a 'tags' field to the 'GET /items' operation to group related endpoints.
* Specify 'required' fields in the request schema for 'POST /users'.

