What is required to be a successful contributor to an open source project?
How to take feedback, how to give reviews, how to leave your ego at the door, how to be a good team player.


1. Study the project and its codebase. Read the documentation, understand the architecture, and get familiar with the code.
2. Understand workflow and style, identify scope, and nature of work.
3. Communication: How does the project communicate? Mailing lists, chat rooms, etc.
4. Understand the governance model and decsion making. Identify maintainers.
5. Identify the activity level of the project. Is it active? Are there many contributors? Are there many open issues? How often do they release? How has been the activity in the past?

Lastly, it is important to know that most OSS orjects fail. You can start small, and is also recommended to start with "good first issue" or "beginner frinedly" issues. Most people who go on to become successful contributors start with functional changes that they were interested in and wanted to see in the project. (and rarely with something trivial).

Be patient, be persistent, humble and respectful. Help in any way you can, even if it is just triaging issues, writing documentation, or helping other contributors. Be open to feedback and be willing to learn.

Contribute in small increments, digestible pieces. Do not dump a lot of code.

When faced with nasty comments, find pearls of wisdom in the criticism, and ignore the rest. Do not take it personally. Do not respond to trolls. Do not engage in flame wars. Do not let your ego get in the way of your contribution. Be professional and respectful at all times.



## CI/CD
CI: Changes are merged into the main branch as often as possible, run automated tests after running automated builds for all S/W and H/W platformss. This allows for faster bug discovery and resolution.

Continuous Delivery: The release process is automated, and the software can be released at any time. This allows for faster delivery of new features and bug fixes to users.

Continuous Deployment: Every change that passes the automated tests is automatically deployed to production. This allows for even faster delivery of new features and bug fixes to users, but it also requires a high level of confidence in the automated tests and the deployment process.

Develop --> Build --> Test --> Deploy --> Release

CI minimizes regression bugs. CD minimizes release bugs. Can speed up development as wrong paths are discovered faster, plus tests and builds can be run automatically. But, it can be expensive to set up and maintain. 

Jenkins, Travis CI, CircleCI, GitHub Actions, GitLab CI/CD, etc. are popular CI/CD tools. They can be used to automate the build, test, and deployment process for open source projects.

What is jenkins and tekton?
Jenkins is an open source automation server that can be used to automate the build, test, and deployment process for open source projects. It is written in Java and has a large ecosystem of plugins that can be used to integrate with various tools and services.

Tekton is an open source framework for creating CI/CD systems. It is built on top of Kubernetes and provides a set of APIs and tools for defining and running CI/CD pipelines. It is designed to be flexible and extensible, allowing users to create custom pipelines that fit their specific needs.


## Licenses
Very important, and has to be set up in the beginning of the project, as it it would be very very difficult, if not impossible, to change the license later on.
1. Permissive: MIT, BSD, Apache, etc. These licenses allow for more freedom and flexibility in how the software can be used and modified. They are often preferred by companies and developers who want to use the software in commercial products or who want to avoid the restrictions of copyleft licenses.
2. Restrictive: GPL and copyleft licenses. These licenses require that any derivative works of the software must also be released under the same license. They are often preferred by developers who want to ensure that their software remains free and open source, and who want to prevent others from using their software in proprietary products.

