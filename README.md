
# Highlights

* Developed CI pipelines for automated testing using:
  * [Jenkins](#jenkins---repo-link)
  * [GitHub Actions](#github-action---workflow1-workflow2)
  * [AWS CodePipeline](#aws-cicd-codepipeline)

* Future work to do:
  * Self hosted runner + Anka Virtual Machine
  * Have the web service app containerized, run tests, and upload docker image to Docker Hub.
  * Use Splunk when CI fails.
  * Explore Infrastructure as Code (IaC). It tries to abstract complexities of CI/CD pipelines to make it easier for developers to follow the “You build it, you run it” motto.

## Jenkins - [Repo link](https://github.com/hanjustin/Poll-Web)

```mermaid
---
title: Jenkins Workflow
---
flowchart RL;

subgraph GitHub

Repo

end

subgraph Local
  subgraph localGit [Local Git]
    code[New code]
  end
  subgraph Docker
    direction BT
    subgraph Jenkins[Jenkins Container]
      direction LR
      checkChange{Code\nchanged?} -- Yes --> CheckOut --> Build --> Test
    end
    subgraph Database[Database Container]
testdb[(Test Database)]
end
    
  end
end

Jenkins -- Poll SCM --> GitHub
localGit -- Push --> GitHub
Database <--> Jenkins
```

![JenkinsHistory](/Screenshot/JenkinsHistory.png)

* Can have locally running CI pipeline through one command: `docker compose up`
* Fully automated Jenkins configuration, plugin installations, and pipeline setup to eliminate manual steps. Used plugins:
    * [Jenkins Configuration as a Code (JCasC)](https://www.jenkins.io/projects/jcasc/)
    * [Job DSL](https://plugins.jenkins.io/job-dsl/)

### Choices considered & Rationale
* **Jenkins vs Other CI tools**
    - Jenkins seems to the most widely used CI tool.
      - **Double edged sword:**
        - Plugins provide great flexibility & customization, but also makes Jenkins to become **Plugin dependency hell**
* **Manual vs Code configuration**
  - Prioritized code configuration to automate and reduce human error when setting up Jenkins. Most online resources involved manual steps using GUI to configure Jenkins, so trying to achieve 100% configure through code greatly increased LOE.
  - Just like any other automation, tradeoffs were:
      - Very high initial implementation LOE, but very high future productivity.
* **CronJob vs PollSCM vs Webhook**
  - PollSCM had the right balance of LOE & efficiency.
    - **CronJob:**
        - Lowest setup LOE & Lowest efficiency.
        - Starts job even when there is no change.
    - **PollSCM:**
        - Medium setup LOE & High efficiency.
        - Looks for changes before starting to prevent unnecessary work.
    - **Webhook:** <br>
        - Highest setup LOE & Highest efficiency.
        - GitHub notifies Jenkins in real time whereas Cron & polling runs/checks every X min.

## GitHub Action - [Workflow1](https://github.com/hanjustin/AcronymsWiki), [Workflow2](https://github.com/hanjustin/XCTestMashup)

```mermaid
---
title: Workflow1 (Splunk)
---
flowchart LR;

subgraph localGit[Local Git]
    dev[New code]
end

subgraph GitHub
    direction LR
    Repo -- Trigger --> runner --> result{Tests\npassed?}

    subgraph runner [GitHub Actions Runner]
        direction LR
        Checkout-->Build-->Test
    end
end

localGit -- Push --> Repo
result -- Fail --> Splunk[Splunk\nTO DO: Future Work]
result -- Pass --> Done
```

```mermaid
---
title: Workflow2 (GitHub Issue)
---
flowchart LR;

subgraph localGit[Local Git]
    dev[New code]
end

subgraph GitHub
    direction LR
    Repo -- Trigger --> runner --> result{Tests\npassed?}

    subgraph runner [GitHub Actions Runner]
        direction LR
        Checkout-->Build-->Test
    end
end

localGit -- Push --> Repo
result -- Fail --> issue[Create GitHub Issue]
result -- Pass --> Done
```

### Choices considered & Rationale
* **GitHub Action vs GitLab**
  - GitHub seems to be the most popular Git repository hosting platform.

## AWS CI/CD CodePipeline

```mermaid
---
title: AWS CodePipeline Workflow
---
flowchart LR;

subgraph localGit[Local Git]
    dev[New code]
end

subgraph AWS
    direction LR
    CodeCommit -- Trigger --> BuildTest --> Deployment

    subgraph Git[Private Git Repo]
        CodeCommit[AWS\nCodeCommit]
    end

    subgraph BuildTest [Build & Test]
        CodeBuild[AWS\nCodeBuild]
    end

    subgraph Deployment
        direction LR
        cloudFormation[AWS\nCloudFormation] --> codeDeploy[AWS\nCodeDeploy] --> ec2[AWS\nEC2]
    end
end

localGit -- Push --> Git

```

![CodePipelineHistory](/Screenshot/CodePipelineHistory.png)

### Choices considered & Rationale
* **AWS vs Azure vs GCP**
  - AWS seems to be the leading cloud service provider.

## Future Work

* GitHub self hosted runner + macOS Virtual Machine using [Anka](https://veertu.com/anka-build/) or [Apple's Virtualization framework](https://developer.apple.com/documentation/virtualization)
* Explore IaC tools such:
    * Terraform
    * Pulumi


