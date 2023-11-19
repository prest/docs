# Contributing to pRESTd

Thank you for your interest in contributing to pREST! We welcome contributions from the community to help improve and grow the project. Before you get started, please take a moment to review the following guidelines.

### Code of Conduct

Please review our [Code of Conduct](code-of-conduct.md) to understand the expected behaviour within the pREST community.

### How to Contribute

1. Fork the repository to your GitHub account.
2.  Clone the forked repository to your local machine:

    ```bash
    git clone https://github.com/your-username/prest.git
    ```
3.  Create a new branch for your changes:

    ```bash
    git checkout -b feature-branch
    ```
4.  Make your changes and commit them:

    ```bash
    git commit -m "Description of your changes"
    ```
5.  Push the changes to your forked repository:

    ```bash
    git push origin feature-branch
    ```
6. Open a pull request on the official pREST repository. Provide a clear title, a description of your changes, and any relevant information.

### Development Setup

If you're planning to make significant contributions or work on new features, follow these steps to set up your development environment:

1. Install Go (if not already installed): [Download Go](https://golang.org/dl/)
2.  Install project dependencies:

    ```bash
    go get -u ./...
    ```
3.  Run tests to make sure everything is set up correctly:

    ```bash
    go test ./...
    ```
4. Make your changes, update tests if necessary, and ensure that all tests pass before submitting a pull request.

### Coding Guidelines

* Follow the [Effective Go](https://golang.org/doc/effective\_go.html) guidelines.
* Adhere to the existing coding style and conventions used in the project.
* Write clear and concise commit messages.
* Document your code using comments where necessary.

### Reporting Issues

If you encounter any issues or have suggestions for improvement, please open an issue on the [GitHub issue tracker](https://github.com/prest/prest/issues).

### Feature Requests

If you have a feature you'd like to see added to pREST, open an issue on the [GitHub issue tracker](https://github.com/prest/prest/issues) and use the "Feature Request" label.

### Review Process

Maintainers will review pull requests, and feedback will be provided. Once approved, changes will be merged into the main branch.

Thank you for contributing to pREST! Your help is greatly appreciated.
