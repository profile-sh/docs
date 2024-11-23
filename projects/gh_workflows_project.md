---
layout: page
title: GitHub Workflows
description: GitHub workflows and actions
giscus_comments: true
author: {{site.author}}
date: 18-11-2024
---

Software development involves several steps (installation, development, build, and deploy) that are repeated forever. GitHub workflows are a way to automate the software development cycle. In this project, we will create and use several workflows, reusable-workflows, and composite actions to understand the GitHub workflow syntax. Visit [GitHub workflows and actions](https://docs.github.com/en/actions) for further help. 
  
## 1. Workflow setup

We need a public organization and two repos under it:

### 1.1 Exercise: Create workflow repos

- create a public organization named gh-workflows-project
- under the organization, create a public repo named call-reusable-workflows and create the following directories:
  - .github
  - .github/workflows
- create a public repo named reusable-workflows and repeat create directories as above. Also, in this repo, create an additional directory named .github/composite-actions

> Workflow files must be directly under .github/workflows, not under any subdirectories.

### 1.2 Exercise: Create a workflow

- create our first workflow w00_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: w00_hello
on:
workflow_dispatch
jobs:   
job1:
  name: test job
  runs-on: ubuntu-latest
  steps:
    - name: say hello
      run: echo 'hello'
```
{% endraw %}

### 1.3 Exercise: Run the workflow and check run log

- click *Actions* tab of the repo
- on left side menu click w1_hello
- click the dropdown *Run workflow* on the right side
- wait for the run to finish
- click on w1_hello (it is next to green cricle with a tick mark) run log
- click the job name to see the run log

If you see a red circle after the workflow run finishes, that means you have errors, clicking on the run log will show the errors.

### 1.4 Exercise: Create a reusable workflow

- create our first reusable workflow rw01_hello.yml in the repo reusable-workflows under .github/workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: rw01_hello
on:
workflow_call
jobs:   
job1:
  name: test job
  runs-on: ubuntu-latest
  steps:
    - name: say hello
      run: echo 'hello'
```
{% endraw %}

### Exercise 1.5: Call the reusable workflow

- create a workflow w01_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: w01_hello call a reusable workflow 
on:
workflow_dispatch
jobs:   
job1:
  name: j1 call a reusable workflow
  uses: gh-workflows-project/reusable-workflows/.github/workflows/rw01_hello.yml@main
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. 

### 1.6 Exercise: Create a composite action

- create our first *composite action* file a02_hello/action.yml in the repo reusable-workflows under .github/composite-actions
- paste the following content in it and commit:

{% raw %} 
```yml
name: a02_hello action
description: Just say hello
runs:
using: "composite"
steps:
  - name: say hello
    shell: bash # we have to set shell explicitly in a composite action
    run: echo hello
```
{% endraw %}

> A **composite action** must be in its own directory with the same name as the action name. For example, for an action named my_action it looks like my_action/action.yml.
> An action is called from a **step** of a job and cannot have jobs within it, it may have several steps.

### 1.7 Exercise: Create a workflow to call the composite action

- create a workflow w02_hello.yml in the repo call-reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: w02_hello call a composite action
on:
workflow_dispatch
jobs:   
job1:
  runs-on: ubuntu-latest
  name: j1 call a composite action
  steps:
    - name: call the action
      uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a02_hello@main
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. 

## 2. *env* context

> The **env** context can be defined at all levels (root, job, or step).
> The **env** context cannot be used under the **id** and **uses** keys, elsewhere it is available.

Exercise:
- create a workflow w03_env.yml in the repo call-reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w03_env demo using env context
on:
  workflow_dispatch
env:  
  globalvar: globalvar_value
jobs:
  job1:
    name: j1 use env context in a job
    runs-on: ubuntu-latest
    env: 
      jobvar: jobvar_value
    steps:
      - name: update env variables
        env: 
          stepvar1: stepvar1_value
          stepvar2: stepvar2_value
        run: |
          echo 'stepvar1, stepvar2, jobvar, and globalvar all are available in this step'
          echo 'we can access env vars with or without env context prefix:'
          echo $stepvar1 $jobvar $globalvar 
          echo ${{ env.stepvar1 }} ${{ env.jobvar }} ${{ env.globalvar }}
          echo let us update values
          echo pushing the var to the env file GITHUB_ENV changes the scope from step to job level
          echo stepvar2=stepvar2_new_value >> $GITHUB_ENV
          echo jobvar=jobvar_new_value >> $GITHUB_ENV
          echo globalvar=globalvar_new_value >> $GITHUB_ENV 
          echo The updated values will be available in the next step, not this step.
          echo see the updated values are not available in this step:
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }}            
      - name: test updated env variables 
        run: |
          echo all updated values are available and stepvar1 is not available here
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }}  
  job2:
    name: j2 show env vars behaviour across jobs
    runs-on: ubuntu-latest
    # To check if the previous job has any effect on env context, we want job1 to finish before we start this job
    needs: job1
    steps:
      - name: test env variables
        run: |
          echo stepvar1, stepvar2, and jobvar are not available acrosss jobs because each job executes with its own env context
          echo global variable is available but the update done in another job will not affect it.
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }} 
      - name: GITHUB_ENV and github.env are the same
        run: |
          echo GITHUB_ENV and github.env give path of the file that stores env variables, this path is unique for each step
          echo $GITHUB_ENV 
          echo ${{github.env}} 
          echo done with workflow 
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. The usage of env context in an *action* is not different and we do not need a separate example.

## 3. *inputs* and *outputs* contexts

While *env* context is very useful *within* a workflow or action, how do we pass information *across* workflows, action? We will learn it below.

> The **iputs** context is used to pass user defined variables **from the caller** to the callee workflows and actions.
> The **outputs** context is used to pass information **from the callee** workflow or action to the caller workflow.
> To work with outputs, we have to use **GITHUB_OUPUTS**, a github default variable for the step dependent path to the file that saves outputs.

In the following subsections we learn how to use the inputs and ouputs contexts in a reusable workflow and a composite action.

### 3.1 Reusable workflow

Exercise: Create a reusable workflow:
- create a reusasble workflow rw04_inputs_outputs.yml in the repo reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yaml
name: rw04_inputs_outputs a reusable workflow with inputs and outputs 
on:
  workflow_call:
    inputs:
      input1:
        required: true  # if 'required' is 'false', set a default value using default: somevalue
        type: string 
      input2:
        required: true  # if 'required' is 'false', set a default value using default: somevalue
        type: string
    outputs:
      an_output:
        description: "An output"
        value: ${{ jobs.set_outputs.outputs.output1 }}
      another_output:
        description: "Another output"
        value: ${{ jobs.set_outputs.outputs.output2 }}
jobs:
  set_outputs:
    name: Generate output
    runs-on: ubuntu-latest
    outputs:
      output1: ${{ steps.step1.outputs.output1 }}
      output2: ${{ steps.step2.outputs.output2 }}
    steps:
      - id: step1
        run: echo "output1=I am an output" >> $GITHUB_OUTPUT
      - id: step2
        run: echo "output2=I am another output" >> $GITHUB_OUTPUT
  print_to_console: # just verify inputs and outputs
   name: print inputs and ouputs
   runs-on: ubuntu-latest
   needs: [set_outputs]
   steps:
    - run: echo '${{ toJSON(inputs) }}'
    - run: echo '${{ toJSON(needs.set_outputs.outputs) }}'
    - run: echo done with reusable workflow
```
{% endraw %}

Now we need to call the above reusable workflow:

Exercise: Create a workflow to pass inputs to the reusable workflow and use outputs from the reusable workflow.
- create a workflow w04_inputs_outputs.yml in the repo call-reuasble-workflows.
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w04_inputs_outputs call a reusable workflow with inputs and outputs 
on:
  workflow_dispatch
jobs:   
  job1: 
    name: j1 call reusable workflow
    uses: gh-workflows-project/reusable-workflows/.github/workflows/rw04_inputs_outputs.yml@main
    with:
     input1: an_input
     input2: another_input
  job2:
    name: j2 use output from j1
    runs-on: ubuntu-latest
    needs: job1 # start only after job1 is done
    steps:
      - name: use output from a completed job
        run: |
          echo ${{ needs.job1.outputs.an_output }} 
          echo ${{ needs.job1.outputs.another_output }}  
  job3:
    name: j3 pass output between steps of a job
    runs-on: ubuntu-latest
    steps:
      - name: step1 create output
        id: step1
        run: echo "output=myoutput" >> $GITHUB_OUTPUT 
      - name: use output from previous step
        run: |
         echo ${{steps.step1.outputs.output}}
         echo workflow done
```
{% endraw %}

Run the workflow and look at the run log. 

### 3.2 Composite action

Exercise: How do we pass information between a caller workflow and an action? Can you create a demo?

Here is a demo:

- create a composite action a05_inputs_outputs/action.yml in the repo reuasble-workflows
- paste the following content in it and commit:
  
{% raw %} 
```yaml
name:  a05_inputs_outputs using inputs and outputs in a composite action
description: 'use input, give output'
inputs:
  input1:
    required: true  # if 'required' is 'false', set a default value using default: somevalue
    type: string
outputs:
  an_output:
    description: "an output"
    value: ${{ steps.step1.outputs.output1 }}
  another_output:
    description: "another output"
    value: ${{ steps.step1.outputs.output2 }}
runs:
  using: "composite"
  steps:
    - name: set outputs
      shell: bash
      id: step1
      run: |
        echo "output1=i am an out" >> $GITHUB_OUTPUT
        echo "output2=i am another output" >> $GITHUB_OUTPUT
      
    - name: use output from previous step 
      shell: bash
      id: step2
      run: |
        echo ${{ inputs.input1 }}
        echo ${{steps.step1.outputs.output1}}
        echo done with action
```
{% endraw %} 

Exercise: Create a workflow to call the above action.

- create a workflow w05_inputs_outputs.yml in the repo call-reuasble-workflows.
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w05_inputs_outputs call a composite action with inputs and outputs 
on:
  workflow_dispatch
jobs:    
  job1:
    name: j1 pass output between steps of a job using a composite action
    runs-on: ubuntu-latest
    steps:
      - name: step1 call action and create output
        id: step1
        uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a05_inputs_outputs@main
        with:
          input1: an_input
      - name: use output from previous step
        run: |
         echo ${{steps.step1.outputs.an_output}}
         echo ${{steps.step1.outputs.another_output}}
         echo workflow done
```
{% endraw %}

Run the workflow and look at the run log. 

## 4. *secrets* and *vars* contexts

Secrets and variables may be attached to a repo, an organization, or a repo environment attached to the repo. In a workflow, we use the contexts *secrets* and *vars* to access these values.
 
Sometimes we have variables that have same values for all repos under an organization or for all workflows under a repo. The repo and organization secrets and variables have to be attached to the caller workflow repo and its parent organization. The *environment* secrets and variables have to be stored in an *environment* created in the repo of the caller workflow. 

> The *env* context and the *environment* (subject of this section) have completely different purpose and functionality.

> Whether we store a variable in a repo, organization, or a *repo environment*, we can access it using *vars* context like vars.[the variable name]. Same is true for secrets. There is no mention of repo/organization/environment when accessing the values of the correspoding variables and secrets.

> We must explicitly pass secrets to reusable workflows and actions, variables are passed by default to a reusable workflows but not to actions.

> The secrets and variables stored under a *repo environment* are accessible only if we use/activate that particular *environment* in a workflow.

### 4.1 Create secrets and variables

Let us create the secrets and variables we need for this demo.

Exercise: Create a secret for the repo call-reusable-workflows and a secret for its parent organization (gh-workflows-project):

- visit the root folder of the repo
- click on repo settings
- on the left sidebar, click on the dropdown *Secrets and Variables*
- click on *actions* under the dropdown
- under *Repository secrets* click on *new repository secret* (We will talk about "Environment secrets" later)
- create a secret with the name *repo_secret* and value my_repo_secret and click *add secret*
- After adding secret you will see the secret name that we have added. On the same page, under *Organization secrets*, follow the steps and add a secret with the name *org_secret* and value my_org_secret.

Exercise: The steps for creating the variables are similar:

- create a variable for the repo and the organization with the names repo_var and org_var and values my_repo_var and my org_var.

So far we have created secrets and variables for the caller workflow repo and its parent organization. For this demo, we also need to create two repo environments with a secret and a variable for each of the environments. A detailed discussion on the use cases of the repo environments is beyond the scope of this document.

Exercise: Create two repo environments and the corresponding secrets and variables.

- visit the page: repo settings -> Environments -> new environment
- create an environment named dev
- create a secret named environment_secret with value my_dev_secret
- create a variable named environment_var with value my_dev_var
- follow the same step as above but replace *dev* with *stage* everywhere

 Our repo now has two environments, named *dev* and *stage*, with each of them having a variable and a secret.
Great, our repo is equipped with all the secrets and variables we need for our demo.

### 4.2 Workflows

Below we show a demo to understand the usage of secrets and variables created under a repo, an organization, and repo environments.

Exercise: Create a reusable workflow:

- create a reusasble workflow rw06_secrets_vars.yml in the repo reusable-workflows
- paste the following content in it and commit:

{% raw %}
```yml
name: rw06_secrets_vars using secrets and variables in a reusable workflow
on:
  workflow_call:
    secrets: # reusable workflows do not have direct access to secrets, we have to pass them from the caller
      repo_s:
        required: true
      org_s:
        required: true
jobs:
  use_secrets_and_vars:
    runs-on: ubuntu-latest
    steps:
    # the sed 's/./& /g' is used to unmask a secret, use it only for dummy secrets
      - run: |
          ${{secrets.org_s=='orgsecret'}}  | sed 's/./& /g'
          echo ${{secrets.org_s}} | sed 's/./& /g' 
          echo ${{vars.repo_var}} # a variable defined at the repository level
          echo ${{vars.org_var}} # a variable defined at the organization level
          echo reusable workflow done
```
{% endraw %}


Exercise: Create a composite-action

- create a composite action a06_secrets_vars/action.yml in the repo reuasble-workflows
- paste the following content in it and commit:

{% raw %}
```yml
name: a06_secrets_vars using secrets and variables in a composite action
description: For composite actions we have to pass secrets and vars as input from the caller, unlike a reusable workflow where secrets are passed as secrets
inputs:
  repo_s:
    required: true  # if 'required' is 'false', set a default value using default: somevalue
    type: string
  org_s:
    required: true  
    type: string
  repo_v:
    required: true  
    type: string
  org_v:
    required: true  
    type: string
runs:
  using: "composite"
  steps:
    - name: print secrets and vars that were passed as input
      shell: bash
      run: |
        echo ${{ inputs.repo_s }} | sed 's/./& /g' # sed command unmasks the secrets, use only for dummy secrets and demos.
        echo ${{ inputs.org_s }} | sed 's/./& /g'
        echo ${{ inputs.repo_v }}
        echo ${{ inputs.org_v }}
        echo done with action
```
{% endraw %}

Exercise: Create a workflow to call the above reusable workflow and action.

- create a workflow w06_secrets_vars.yml in the repo call-reuasble-workflows.
- paste the following content in it and commit:

{% raw %}
```yml
# We must explicitly pass secrets to reusable workflows and actions, variables are passed by default to workflows but not to actions.
name: w06_secrets_vars context test reusable workflow and action with secrets and variables
on:
  workflow_dispatch
jobs:
  job1:
    name: j1 Test secrets_and_vars reusable workflow 
    uses: gh-workflows-project/reusable-workflows/.github/workflows/rw06_secrets_vars.yml@main
    secrets: # We have to explicitly pass secrets from the caller, however vars do not need explicit passing
      repo_s: ${{ secrets.repo_secret }}       
      org_s: ${{ secrets.org_secret }}    
  job2:
    name: j2 Test secrets_and_vars using an action
    runs-on: ubuntu-latest
    steps:
      - name: call the action in this step
        uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a06_secrets_vars@main
        with: # we cannot really pass secrets as secrets to an action, passing as inputs works as 
              # they are masked even when passed as an input. We also HAVE to pass vars as input
          repo_s: ${{secrets.repo_secret}}       
          org_s: ${{secrets.org_secret}}
          repo_v: ${{vars.repo_var}}       
          org_v: ${{vars.org_var}}
  job3:
    name: j3 test dev environment 
    runs-on: ubuntu-latest
    environment:
      name: dev
    steps:
      - run: |
          echo ${{vars.environment_var}}
          echo ${{vars.environment_secret}}  | sed 's/./& /g'  
  job4:
    name: j4 test stage environment 
    runs-on: ubuntu-latest
    environment:
      name: stage
    steps:
      - run: |
          echo ${{vars.environment_var}}
          echo ${{vars.environment_secret}}  | sed 's/./& /g'
          echo workflow done
```
{% endraw %}

Run the workflow and look at the run logs.
Let us explore other contexts in the next section.

## 5. Other contexts and default variables
working on it.
## 6. Calling a composite action from a reusable workflow

## 7. Files and scripts





