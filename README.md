# onboarding-guide

This guide is for new programs that want to be published on SAIC's public GitHub organization.

## Philosophy

Publishing something as open source is a radical change over what has been done in the past with private Source Code Management (SCM) tools like SIF GitLab. Because of the vastly increased scrutiny, our processes will have more structure and require more care than projects have in private Gitlab.

## Criteria

Projects that want to publish on SAIC's public GitHub must meet all of the following criteria.

1. The admins (the people in the team [@saic-oss/compliance](https://github.com/orgs/saic-oss/teams/compliance)) must have an email from the Chief Technical Officer (CTO) and the Chief Intellectual Property Council (CIPC) authorizing the new project, including a description of what the project entails.
    1. The people currently in those positions are Charles Onstott and Samantha Garner
1. Each repository is able to pass the required compliance pipelines (see details below)
1. The trunk branch of the project is named `main`. At the moment only [GitHub Flow](https://guides.github.com/introduction/flow/) is supported. No [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

## Process

1. The Compliance team receives authorization from CTO and CIPC that a new project can be published
1. A GitHub Team is created with the name of the project
1. The members of the project are added to the team by being invited as contributors to the organization. Each member is required to have MFA attached to their GitHub account 
1. The Compliance team receives a request for a new repository to be created. The request must include the project's name, short description, and details on what type of technology stack it will be made of.
1. Compliance team creates the new repository from a template and gives the new project team write access to the project

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

- The compliant pipelines from Codefresh are currently very simple. They will be built out with more checks as time goes on.
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
