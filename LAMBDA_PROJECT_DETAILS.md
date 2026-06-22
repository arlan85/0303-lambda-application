# Lambda Project Details

## Application Overview

**The Sample Application** is a Python-based AWS Lambda function that provides a REST API for querying university and college data. The application serves JSON data about various educational institutions, including their mascots, nicknames, locations, and conference affiliations.

| Endpoint                | Description                                                |
|-------------------------|------------------------------------------------------------|
| **Home Page (`/`)**     | Documentation page displaying environment, version, platform, and build information |
| **Get All Data (`/data`)** | Returns all university records in JSON format            |
| **Get Item by ID (`/{id}`)** | Returns a specific university record by its ID         |
| **Error Handling**      | Returns 404 status for invalid IDs                         |

The Lambda function is designed for deployment to AWS Lambda with support for staging and production environments. It uses GitHub Actions for continuous integration and continuous deployment (CI/CD) with OIDC authentication.

## Project Structure

The project follows a standard Python Lambda function structure with clear separation of concerns:

| File/Directory | Description |
| -------------- | ----------- |
| `index.py` | The main Lambda handler function that processes API requests and returns responses |
| `index_test.py` | Unit tests for the Lambda handler using Python's unittest framework |
| `data.json` | JSON data file containing university/college information (mascots, nicknames, locations, etc.) |
| `template.html` | HTML template for the home page documentation |
| `requirements.txt` | Python package dependencies for development and deployment |
| `Makefile` | Build automation and deployment commands |
| `1-integrate-python-app.yml` | GitHub Actions workflow for continuous integration (linting and testing) |
| `2-deploy-python-app.yml` | GitHub Actions workflow for continuous deployment (build, package, and deploy) |
| `lambda.zip` | Deployment artifact (generated during build, excluded from version control) |

The following directory tree shows the structure of this Lambda project:

```bash
LAMBDA/
├── index.py                   # Lambda handler function
├── index_test.py              # Unit tests
├── data.json                  # University/college data
├── template.html              # HTML template for home page
├── requirements.txt           # Python dependencies
├── Makefile                   # Build automation
├── 1-integrate-python-app.yml # CI workflow
├── 2-deploy-python-app.yml    # CD workflow
└── lambda.zip                 # Deployment artifact (generated)
```

## API Endpoints

The Lambda function handles three types of requests:

### GET `/`

Returns an HTML documentation page displaying:

- Environment (STAGING/PRODUCTION)
- Version (Git commit SHA)
- Platform (deployment platform name)
- Build Number (GitHub Actions run number)

**Response:**

- Status Code: `200`
- Content-Type: `text/html`
- Body: Rendered HTML template

### GET `/data`

Returns all university records in JSON format.

**Response:**

- Status Code: `200`
- Content-Type: `application/json`
- Body: JSON array of all university objects

**Example Response:**

```json
[
  {
    "id": "1",
    "school": "Iowa State University",
    "mascot": "Cy the Cardinal",
    "nickname": "Cyclones",
    "location": "Ames, IA, USA",
    "latlong": "42.026111,-93.648333",
    "ncaa": "Division I",
    "conference": "Big 12 Conference"
  },
  ...
]
```

### GET `/{id}`

Returns a specific university record by its ID.

**Response (Success):**

- Status Code: `200`
- Content-Type: `application/json`
- Body: JSON object for the matching university

**Response (Not Found):**

- Status Code: `404`
- Content-Type: `application/json`
- Body: Error message with item_id and event details

## Makefile Commands

- The project includes a Makefile for common development, testing, and deployment tasks.
- Run `make` or `make hello` to see all available commands.

### `make hello`

Display available Makefile targets and their descriptions, including usage examples.

```bash
make hello
```

### `make requirements`

Install Python dependencies from `requirements.txt`.

```bash
make requirements
```

**What it does:**

- Upgrades pip to the latest version
- Installs all packages listed in `requirements.txt` (black, pylint, flake8, awscli)

### `make check`

Verify that all required tools are installed and accessible.

```bash
make check
```

**What it does:**

- Checks environment variables
- Verifies zip command availability
- Checks Python version
- Verifies pylint and flake8 are installed
- Checks AWS CLI version

### `make lint`

Run code linters to check code quality and style.

```bash
make lint
```

**What it does:**

- Runs pylint with error-only output (exits with zero status)
- Runs flake8 with relaxed line length rules
- Both linters exit with zero status to allow CI/CD pipelines to continue

### `make black`

Format Python code using black (dry-run mode showing differences).

```bash
make black
```

**What it does:**

- Runs black in diff mode to show formatting changes without modifying files
- Use `black index.py` directly to apply formatting

### `make test`

Run the unit test suite.

```bash
make test
```

**What it does:**

- Executes all tests in `index_test.py` using Python's unittest framework
- Tests include:
  - Home page rendering
  - Get all data endpoint
  - Get item by ID endpoint
  - Invalid ID error handling

### `make build`

Create the Lambda deployment package.

```bash
make build
```

**What it does:**

- Creates `lambda.zip` containing:
  - `index.py` (Lambda handler)
  - `data.json` (university data)
  - `template.html` (HTML template)
- This zip file is ready for deployment to AWS Lambda

### `make deploy`

Deploy the Lambda function to AWS.

```bash
make deploy FUNCTION=sample-application-staging PLATFORM="GitHub Actions" VERSION=abc123 BUILD_NUMBER=42
```

**Required Variables:**

- `FUNCTION`: Name of the AWS Lambda function to update (must exist)

**Optional Variables:**

- `PLATFORM`: Deployment platform name (default: undefined)
- `VERSION`: Version identifier, typically Git commit SHA (default: undefined)
- `BUILD_NUMBER`: Build number from CI/CD platform (default: undefined)

**What it does:**

1. Verifies AWS credentials using `aws sts get-caller-identity`
2. Waits for the Lambda function to be active
3. Updates function configuration with environment variables:
   - `PLATFORM`: Deployment platform
   - `VERSION`: Version identifier
   - `BUILD_NUMBER`: Build number
   - `ENVIRONMENT`: Automatically set based on function name:
     - `STAGING` if function name contains `-staging`
     - `PRODUCTION` if function name contains `-production`
     - `undefined` otherwise
4. Waits for configuration update to complete
5. Updates function code with `lambda.zip`
6. Waits for code update to complete

**Example:**

```bash
make deploy \
  FUNCTION=sample-application-staging \
  PLATFORM="GitHub Actions" \
  VERSION="${GITHUB_SHA}" \
  BUILD_NUMBER="${GITHUB_RUN_NUMBER}"
```

### `make testdeployment`

Test the deployed Lambda function.

```bash
make testdeployment URL=https://api.example.com VERSION=abc123
```

**Required Variables:**

- `URL`: The Lambda function URL to test
- `VERSION`: Version identifier to verify in the response

**What it does:**

- Makes a curl request to the specified URL
- Greps for the version string in the response
- Verifies the deployment is working correctly

### `make clean`

Remove generated build artifacts.

```bash
make clean
```

**What it does:**

- Removes `lambda.zip` file
- Use this before rebuilding to ensure a fresh deployment package

### `make all`

Run the complete development workflow.

```bash
make all
```

**What it does:**

- Executes in order: `clean`, `lint`, `black`, `test`, `build`, `deploy`
- Note: `deploy` still requires the `FUNCTION` variable to be set

## Development Workflow

| Step | Command | Details |
|------|---------|---------|
| **Getting Started** | | |
| 1. Install dependencies | `make requirements` | Installs all Python packages from requirements.txt |
| 2. Verify setup | `make check` | Confirms all tools are installed and accessible |
| **Making Changes** | | |
| 1. Edit code | — | Modify `index.py` to change Lambda handler logic |
| 2. Edit data | — | Modify `data.json` to update university records |
| 3. Edit template | — | Modify `template.html` to change home page appearance |
| **Testing Changes** | | |
| 1. Run linters | `make lint` | Checks code quality and style |
| 2. Format code | `make black` | Shows formatting differences (or run `black` directly) |
| 3. Run tests | `make test` | Executes unit tests to verify functionality |
| **Building** | | |
| 1. Build package | `make build` | Creates `lambda.zip` for deployment |
| **Deploying** | | |
| 1. Deploy to staging | `make deploy FUNCTION=my-app-staging PLATFORM="Local" VERSION=dev BUILD_NUMBER=1` | Deploys to staging environment |
| 2. Test deployment | `make testdeployment URL=https://staging-api.example.com VERSION=dev` | Verifies deployment works |
| 3. Deploy to production | `make deploy FUNCTION=my-app-production PLATFORM="Local" VERSION=prod BUILD_NUMBER=1` | Deploys to production environment |

## CI/CD Workflows

The project includes two GitHub Actions workflows:

### 1-Integration Workflow (`1-integrate-python-app.yml`)

**Triggers:**

- Push to any branch except `main`
- Manual workflow dispatch
- Called by other workflows (`workflow_call`)

**What it does:**

- Sets up Python 3.11 environment
- Installs dependencies (flake8, pytest, requirements.txt)
- Runs flake8 linter
- Runs pytest with JUnit XML output
- Publishes test results as GitHub Actions checks

**Permissions:**

- `contents: read` - Read repository contents
- `checks: write` - Write check results

### 2-Deployment Workflow (`2-deploy-python-app.yml`)

**Triggers:**

- Push to `main` branch
- Manual workflow dispatch

**What it does:**

1. **Integration Job**: Reuses the integration workflow to run tests
2. **Package Job**:
   - Sets up Python environment
   - Installs requirements
   - Builds `lambda.zip` artifact
   - Uploads artifact for use in deployment jobs
3. **Staging Deployment**:
   - Configures AWS credentials using OIDC
   - Downloads `lambda.zip` artifact
   - Deploys to staging environment
   - Tests the deployment
4. **Production Deployment**:
   - Requires manual approval (environment protection rules)
   - Configures AWS credentials using OIDC
   - Downloads `lambda.zip` artifact
   - Deploys to production environment
   - Tests the deployment

**Required GitHub Variables:**

- `AWS_ROLE_ARN`: AWS IAM role ARN for OIDC authentication
- `AWS_REGION`: AWS region for deployment
- `FUNCTION_NAME`: Name of the Lambda function
- `URL`: API Gateway URL or Lambda function URL for testing

**Permissions:**

- `id-token: write` - Required for OIDC authentication
- `contents: read` - Read repository contents
- `checks: write` - Write check results

## Configuration

### Environment Variables

The Lambda function uses the following environment variables (set during deployment):

| Variable | Description | Example |
|----------|-------------|---------|
| `ENVIRONMENT` | Deployment environment | `Staging`, `Production` |
| `PLATFORM` | Deployment platform name | `GitHub Actions`, `Local` |
| `VERSION` | Version identifier (typically Git SHA) | `abc123def456` |
| `BUILD_NUMBER` | Build number from CI/CD platform | `42` |

The `ENVIRONMENT` variable is automatically determined from the function name:

- Functions containing `-staging` → `ENVIRONMENT=STAGING`
- Functions containing `-production` → `ENVIRONMENT=PRODUCTION`
- Otherwise → `ENVIRONMENT=undefined`

### Data File

The `data.json` file contains an array of university/college objects. Each object includes:

- `id`: Unique identifier
- `school`: University name
- `mascot`: Mascot name
- `nickname`: Team nickname
- `location`: City and state
- `latlong`: Latitude and longitude coordinates
- `ncaa`: NCAA division
- `conference`: Athletic conference

## Features

- **RESTful API**: Clean API design with JSON responses
- **Error Handling**: Proper HTTP status codes (200, 404)
- **Environment Awareness**: Displays deployment environment information
- **CI/CD Integration**: Automated testing and deployment via GitHub Actions
- **OIDC Authentication**: Secure AWS access without long-lived credentials
- **Multi-Environment Support**: Separate staging and production deployments
- **Automated Testing**: Unit tests with pytest
- **Code Quality**: Linting with flake8 and pylint

## Troubleshooting

### Build Fails

- Ensure all required files exist: `index.py`, `data.json`, `template.html`
- Check that `zip` command is available: `zip --version`
- Verify Python is installed: `python --version`

### Tests Fail

- Ensure `data.json` exists in the same directory as `index_test.py`
- Check that `template.html` exists (required for home page test)
- Verify all dependencies are installed: `make requirements`

### Deployment Fails

- Verify AWS credentials are configured: `aws sts get-caller-identity`
- Check that the Lambda function exists: `aws lambda get-function --function-name YOUR_FUNCTION_NAME`
- Ensure you have permissions to update Lambda functions
- Verify the function name matches the expected pattern for environment detection

### OIDC Authentication Issues

- Verify GitHub repository secrets are configured correctly
- Check that the IAM role trust policy allows GitHub Actions OIDC provider
- Ensure the workflow has `id-token: write` permission
- Verify `AWS_ROLE_ARN` variable is set correctly in GitHub repository settings

### Function Returns 404 for Valid IDs

- Check that `data.json` is included in `lambda.zip` (verify with `unzip -l lambda.zip`)
- Ensure the file is readable in the Lambda execution environment
- Verify the ID format matches exactly (case-sensitive)
