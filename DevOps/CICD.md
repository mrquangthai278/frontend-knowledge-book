# CI/CD

## Definition / Concept
**CI/CD** stands for **Continuous Integration/Continuous Deployment**. It's a practice and set of tools that automate the testing, building, and deployment of software applications. **Continuous Integration** (CI) automatically builds and tests code changes, while **Continuous Deployment** (CD) automatically releases validated changes to production. CI/CD pipelines reduce manual errors, speed up release cycles, and enable rapid feedback loops.

- Automates build, test, and deployment processes
- Reduces manual errors and deployment failures
- Enables frequent releases with confidence
- Provides rapid feedback to developers

## Visual Representation
```
Developer Code Commit
    ↓
Webhook Trigger (GitHub, GitLab)
    ↓
CI Server (Jenkins, GitHub Actions, GitLab CI)
    ↓
Build Stage (Compile, Dependencies)
    ↓
Test Stage (Unit, Integration, E2E tests)
    ↓
Artifact Creation (Docker image, binaries)
    ↓
Deploy to Staging (Test environment)
    ↓
Run Smoke Tests
    ↓
Deploy to Production
    ↓
Monitor & Alert
```

## Example
```yaml
# GitHub Actions CI/CD Pipeline (.github/workflows/deploy.yml)
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run tests
        run: npm run test

      - name: Build application
        run: npm run build

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Push to Registry
        if: github.ref == 'refs/heads/main'
        run: |
          docker tag myapp:${{ github.sha }} myapp:latest
          docker push myapp:latest

      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        run: |
          curl -X POST https://api.example.com/deploy \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}" \
            -d '{"version": "${{ github.sha }}"}'
```

## Usage
- **When to use**: Every software project should implement CI/CD to maintain code quality and accelerate deployments
- **Real-world example**: A frontend team uses GitHub Actions to automatically test, build, and deploy their React application to AWS S3 whenever code is pushed to main
- **Best practices**:
  - **Automated testing**: Run unit, integration, and E2E tests automatically
  - **Fast feedback**: Keep CI pipelines fast (target under 10 minutes)
  - **Fail fast**: Stop pipeline on first failure to save resources
  - **Environment consistency**: Use Docker or containers to ensure test environment matches production
  - **Secrets management**: Never hardcode credentials; use environment variables or secret managers
  - **Monitoring**: Track deployment success rates and performance metrics
  - **Rollback capability**: Enable quick rollbacks if deployment fails

## FAQ / Interview Questions

**Q: What is the difference between Continuous Integration and Continuous Deployment?**
A:
- **Continuous Integration (CI)**: Automatically builds and tests every code change pushed to the repository. Developers get immediate feedback on whether their changes break the application.
- **Continuous Deployment (CD)**: Automatically deploys validated changes to production without manual approval. Some teams use **Continuous Delivery** instead, which requires manual approval before production deployment.
- In practice: CI is essential; CD varies based on team risk tolerance.

**Q: What are the main benefits of CI/CD?**
A: Key benefits include:
- **Speed**: Automated processes reduce deployment time from hours/days to minutes
- **Reliability**: Consistent automated testing reduces bugs reaching production
- **Safety**: Frequent small releases are lower risk than large deployments
- **Feedback**: Developers get immediate feedback on code quality
- **Scalability**: Enables teams to deploy multiple times daily
- **Cost**: Reduces manual labor and production incidents

**Q: What tools are commonly used for CI/CD?**
A: Popular CI/CD platforms include:
- **Jenkins**: Open-source, highly customizable
- **GitHub Actions**: Integrated with GitHub, free for public repos
- **GitLab CI**: Built into GitLab with strong features
- **CircleCI**: Cloud-based, developer-friendly
- **Azure DevOps**: Microsoft's comprehensive solution
- **Travis CI**: Simple, good for open-source projects

**Q: How do you handle secrets and sensitive data in CI/CD pipelines?**
A: Best practices for secrets management:
- **Never commit secrets** to version control (use `.gitignore`)
- **Use secret managers**: GitHub Secrets, GitLab CI/CD Variables, HashiCorp Vault
- **Rotate credentials** regularly
- **Limit access**: Only grant necessary permissions to CI/CD runners
- **Audit access**: Log who accessed secrets and when
- **Use short-lived tokens**: Temporary credentials reduce exposure window
- Example: `${{ secrets.DATABASE_PASSWORD }}` in GitHub Actions

**Q: What should a good CI/CD pipeline include?**
A: A comprehensive pipeline should have:
- **Source control**: Code committed to Git (trigger point)
- **Build**: Compile code, resolve dependencies
- **Unit Tests**: Fast, focused tests on individual components
- **Linting**: Code quality and style checks
- **Integration Tests**: Test component interactions
- **Build artifacts**: Create Docker images or binaries
- **Deploy to staging**: Test in production-like environment
- **Smoke tests**: Verify critical functionality post-deployment
- **Deploy to production**: Release to users
- **Monitoring**: Track performance and errors
- **Rollback**: Ability to quickly revert if needed

## References
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [AWS DevOps - CI/CD Practices](https://aws.amazon.com/devops/continuous-integration/)

---
*See also: [Dockerfile](./Dockerfile.md), [Dockerfile Optimization](./DockerfileOptimization.md)*
