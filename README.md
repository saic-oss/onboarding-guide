# onboarding-guide

This guide is to be used by new programs that want to be published on SAIC's public GitHub organization.

## Philosophy

Publishing something as open source is a radical change over what has been done in the past with private Source Code Management (SCM) tools like SAIC Innovation Factory's GitLab. Because of the vastly increased scrutiny, our processes will have more structure and require more care than projects have in private GitLab. There will be no exceptions to the process, regardless of how small or insignificant a project might be. Amendments can be made to the process after careful consideration from the public GitHub admin team.

## Supported project types

The types of projects that get published must be supported. If you have a project that you want to publish of a type that is currently unsupported, please let us know so we can build out support for that project. It will not be published until it supported.

The types of projects we currently support are:

- A Terraform module
- A containerized Java/SpringBoot app that uses Gradle

Types of projects that we don't yet support but will soon are:

- A Helm chart
- A containerized Java/SpringBoot app that uses Maven
- A Docusaurus documentation site that deploys to GitHub Pages

## Permission structure

Org Owners can:

- Do anything. Only Org Owners have permission to create new repositories.

Compliance Team can:

- Administer all existing repositories, including making changes to settings, webhooks, branch protections, etc.
- Approve changes to controlled files and folders, like `/.github`, `/.pre-commit-config.yaml`, `/Taskfile.yml`, etc.
- Force-push to the trunk branch (but please never do it!)
- Merge Pull Requests without the required approvals/status checks (Should only happen very rarely in exceptional circumstances)

Project team members can:

- Have write access to the repositories in their project. Write access lets you read, clone, and push to the repository, as well as manage issues and pull requests.

## Criteria

Projects that want to publish on SAIC's public GitHub must meet all of the following criteria.

1. The admins (the people in the team [@saic-oss/compliance](https://github.com/orgs/saic-oss/teams/compliance)) must receive an email from the Chief Technical Officer (CTO) and the Chief Intellectual Property Council (CIPC) authorizing the new project, including a description of what the project entails.
1. Each repository is able to pass the required compliance pipelines (see details below)
1. The trunk branch of the project is named `main`. At the moment only [GitHub Flow](https://guides.github.com/introduction/flow/) is supported. No [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).
1. The project must be published as Open Source Software (OSS) and use the Apache 2.0 license. If you need to use a different license, approval is needed from the CIPC and the compliance team with appropriate justification. Any kind of custom license or any license that can't be found on [TLDRLegal](https://tldrlegal.com/) will be met with strong opposition.

## Process

1. The Compliance team receives authorization from CTO and CIPC that a new project can be published
1. A GitHub Team is created with the name of the project
1. The members of the project are added to the team by being invited as contributors to the organization. Each member is required to have MFA attached to their GitHub account. 
1. The Compliance team receives a request for a new repository to be created. The request must include the project's name, short description, and details on what type of technology stack it will be made of.
1. Compliance team creates the new repository from a template and gives the new project team write access to the project. The repo's visibility is set to private.
1. Project team creates a Pull Request (PR) to add the initial migration of code (or new code if the project is greenfield)
1. Compliance team does a security audit and review of the initial PR
1. Compliance team changes the repo's visibility to public AFTER the Codefresh CI pipeline goes green in the PR but BEFORE the initial PR is merged.
1. Compliance team adds the missing Branch Protection Rule (since they aren't available in private projects) and sets the WIP, Codefresh, and Codacy status checks as required. See [Branch Protection Rule](#branch-protection-rule) for full details on the required Branch Protection Rule.

## Software Development Lifecycle (SDLC)

In order to ensure the quality of the work we publish to the public GitHub, developers will be required to adhere to the following SDLC at all times.

1. Developer clones the project
1. Developer creates a feature branch off of `main` and adds commits to the branch
1. The feature branch is pushed to GitHub
1. Developer creates a Pull Request. The Pull Request requires at least one review from each [CODEOWNER](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/about-code-owners) that is identified. The compliance team is the CODEOWNER of the `/.github` folder, `/.pre-commit-config.yaml`, `/Taskfile.yml`, and any configuration files that Codacy uses. The compliant pipelines use these files to complete their process. Developers may change them, but changes require an approval from the compliance team.
1. The Pull Request can be merged once it has all necessary approvals, and all required status checks have passed. The required status checks include:
    1. A [Codacy](https://app.codacy.com/) automated code review
    1. A Continuous Integration (CI) pipeline powered by Codefresh that runs things like pre-commit hook validation, automated test suites, SonarQube scan, license compliance scan, etc.
    
## Notes

- The compliant pipelines from Codefresh are currently very simple. They will be built out with more checks as time goes on. You will be given 
- Due to the danger in potentially leaking secrets when running the CI pipeline (especially in forks), and to reduce the load on the limited concurrent pipelines we have in Codefresh, all runs of the CI pipeline are triggered by creating a comment in the Pull Request with the value `/test`. You must have your [org visibility](https://github.com/orgs/saic-oss/people) set to "public" for Codefresh to accept you as someone who is authorized to trigger a pipeline. Please review what has been changed before triggering a new CI pipeline.
- All projects are required to use certain tools. For full details see the [Tools][#tools] section.

## Tools

### Pre-commit

[Pre-commit](https://pre-commit.com/) is a tool that runs checks on the changes you make before you commit them. It is required on all of SAIC's open source projects. The CI pipeline will verify that pre-commit hooks were run and fail if they weren't.

To install pre-commit locally, follow these steps:

1. Ensure Python is installed
1. Run `pip install pre-commit`

To use pre-commit:

```sh
# Install the hooks
pre-commit install

# Manually run the hooks (if necessary). They will automatically run on every git commit.
pre-commit run --all-files
```

### Go-Task

[Go-Task](https://taskfile.dev/#/) is a task runner and is a simpler alternative to `make`. All projects require a Taskfile because the CI pipeline uses it to run its stages. Doing it this way makes it so that individual projects can be flexible with what each stage does while providing a simple configuration in the CI engine.

The following tasks are required to be present in all projects.

1. `task validate` - Runs all pre-commit hooks to validate that they were run before pushing the commit up. The pipeline will fail if the pre-commit hooks were not run.
1. PROPOSED - `task build` - If applicable, builds the production artifact
1. PROPOSED - `task test` - If applicable, Runs all automated tests, and any tools that rely on those tests, like SonarQube
1. PROPOSED - `task secure` - If applicable, Runs all security tools, like container security scans, OpenSCAP, Fossa/WhiteSource, etc.
1. PROPOSED - `task deliver` - If applicable, Delivers the production artifact to an artifact repository
1. PROPOSED - `task deploy` - If applicable, Deploys to specified environment (Note: This is not likely to be used as we use Harness for compliant deployments.)

All tasks must be able to be independently run. For example, if the `test` task depends on `build`, it must run `build` as part of `task test`.

### ASDF

The Docker image that runs all CI pipeline stages uses [ASDF](https://asdf-vm.com/#/) whenever possible for the installation of tools. If you include a `.tool-versions` file the pipeline will ensure the correct version of the tools you use are installed before running the stage. If you do not include a `.tool-versions` file the default installed version of all tools will be used. We strongly encourage the use of a `.tool-versions` file because the default version installed of the tools will change with time and are not guaranteed to work for you, or even with each-other.

## Branch Protection Rule

The following is required to be set as a Branch Protection Rule on every new and existing project:

- Branch name pattern: `main`
- Require pull request reviews before merging: Yes
- Required approving reviews: At least 1, can be more if project team desires
- Dismiss stale pull request approvals when new commits are pushed: Yes
- Require review from Code Owners: Yes
- Restrict who can dismiss pull request reviews: Yes (leave the list empty)
- Require status checks to pass before merging: Yes
- Require branches to be up to date before merging: Yes
- Required status checks: `Codacy Static Code Analysis`, `Public GitHub/<InsertNameOfPipelineHere>`, `WIP`
- Require signed commits: No
- Require linear history: No
- Include administrators: No
- Restrict who can push to matching branches: Yes (leave the list blank)
- Allow force pushes: No
- Allow deletions: No
