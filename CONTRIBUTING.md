# Contributing to AWS IAM Identity Center - Inactive User Removal

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to this project.

## Code of Conduct

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Be open to different perspectives

## How to Contribute

### Reporting Bugs

1. Check if the bug has already been reported in [Issues](../../issues)
2. If not, create a new issue with:
   - Clear title and description
   - Steps to reproduce
   - Expected vs actual behavior
   - Environment details (AWS region, CloudFormation version, etc.)
   - Relevant logs or error messages

### Suggesting Enhancements

1. Check existing issues and pull requests
2. Create a new issue with:
   - Clear description of the enhancement
   - Use case and benefits
   - Possible implementation approach (if you have one)

### Submitting Pull Requests

1. **Fork the repository**
   ```bash
   git clone https://github.com/your-username/aws-identity-center-inactive-user-removal.git
   cd aws-identity-center-inactive-user-removal
   ```

2. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

3. **Make your changes**
   - Follow existing code style
   - Update documentation if needed
   - Add comments for complex logic
   - Test your changes thoroughly

4. **Commit your changes**
   ```bash
   git commit -m "Add: Description of your changes"
   ```
   Use conventional commit messages:
   - `Add:` for new features
   - `Fix:` for bug fixes
   - `Update:` for improvements
   - `Remove:` for removals
   - `Docs:` for documentation changes

5. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```
   Then create a Pull Request on GitHub.

## Development Guidelines

### Code Style

- Follow AWS CloudFormation best practices
- Use meaningful parameter and resource names
- Add comments for complex logic
- Keep functions focused and single-purpose
- Use consistent indentation (2 spaces for YAML)

### Testing

Before submitting:
- Test the CloudFormation template validates correctly
- Test with different parameter combinations
- Verify Lambda function executes successfully
- Test Slack notifications (if applicable)
- Check CloudWatch Logs for errors

### Documentation

- Update README.md if adding new features
- Add parameter descriptions in the template
- Include examples for new functionality
- Update CHANGELOG.md with your changes

## Pull Request Process

1. Ensure your code follows the style guidelines
2. Update documentation as needed
3. Test thoroughly
4. Create a clear PR description:
   - What changes were made
   - Why they were made
   - How to test
   - Any breaking changes

5. Wait for review and address feedback
6. Once approved, maintainers will merge

## Questions?

Feel free to open an issue for questions or discussions about contributions.

Thank you for contributing! 🎉

