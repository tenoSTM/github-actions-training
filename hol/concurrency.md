# How to Implement This Workflow

[Documentation](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/control-the-concurrency-of-workflows-and-jobs)

1. **Navigate to Your Repository**  
   Open your GitHub repository where you want to add the workflow.

2. **Create the Workflow Directory**  
   Ensure there is a `.github/workflows` directory in the root of your repository. If it does not exist, create it.

3. **Add a New Workflow File**  
   Inside the `.github/workflows` directory, create a new file. You can name it, for example, `concurency-lab.yml`.

4. **Set the Workflow Name**  
   At the top of the file, specify a name for the workflow, such as “Concurrency Demo”.

5. **Configure the Trigger**  
   Set the workflow to be triggered manually using the `workflow_dispatch` event.

6. **Define Concurrency Settings**  
   Add a concurrency group that uses the workflow name and reference. Enable the option to cancel any in-progress runs if a new one starts.

<details>
  <summary>Solution</summary>

```YAML
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true 
```

</details>

7. **Add the First Job**  
   Add a job that runs on the latest Ubuntu runner. Include steps to check out the repository, list the files in the repository, group the output with the `::group::` workflow command and simulate work by pausing for 30 seconds.

<details>
  <summary>Solution</summary> 

```YAML
    - name: List files in the repository
      run: |
        echo "::group::The repository ${{ github.repository }} contains the following files"
        tree
        echo "::endgroup::"
```

</details>

8. **Add the Second Job**  
   Add a second job that also runs on the latest Ubuntu runner. Include steps to simulate work by pausing for 30 seconds and then add a summary to the job.

<details>
    <summary>Solution</summary>
    
```YAML

      - name: Add job summary
        run: echo "### Job completed! :rocket:" >> $GITHUB_STEP_SUMMARY
```

</details>

9. **Save and Commit**  
   Save the workflow file and commit it to your repository.

10. **Run the Workflow**  
    Go to the Actions tab in your GitHub repository, select the workflow, and trigger it twice in quick succession. Observe that only the jobs from the last triggered workflow will run, as earlier runs will be cancelled due to the concurrency settings.

<details>
  <summary>Solution</summary>

```YAML
name: Concurrency Demo

on:
    workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  job_one:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 
        uses: actions/checkout@v4

      - name: List files in the repository
        run: |
          echo "::group::The repository ${{ github.repository }} contains the following files"
          tree
          echo "::endgroup::"

      - name: Simulate work
        run: sleep 30  

  job_two:
    runs-on: ubuntu-latest
    steps:
      - name: Simulate work
        run: sleep 30

      - name: Add job summary
        run: echo "### Job completed! :rocket:" >> $GITHUB_STEP_SUMMARY
```

</details>


